# 番茄采摘机器人 3DGS 仿真环境构建 —— 进度报告

## 〇-2、2025-09-03 更新：L515_day 纯 RGB 3DGS 路线（已验证可行）

**结论：可行，且新视角质量显著优于夜景深度路线**（验证帧 PSNR 8.8 → 13.0，+4.2dB）。

管线（全程不读深度图）：
1. `scripts/prep_day_rgb.py`：122 张白天图去畸变（工厂 Brown-Conrady 系数）+ 裁边 5%
   → 1152×648 无畸变 PINHOLE 图（`data/l515_day_rgb/images/`）
2. 固定内参重跑 SfM（PINHOLE 定值 904.136/904.541/582.646/316.313 + `ba_refine_* 0` +
   sequential matching + 放宽初始化）→ 8 个模型，最大组件 **34 帧连续**（16:12:31~16:15:56）
3. 体检（对比旧自由自标定模型的橡皮世界）：
   - 轨迹折线 12.9m（旧 118.7m）——尺度恢复正常
   - 观测深度中位 1.87m（p10~p90: 1.68~3.73m）——冠层真实距离，米制量级自动正确
   - 重投影误差中位 0.33px
4. `scripts/export_day_rgb_data.py`：导出 transforms.json（30 训 + 4 验证帧，无深度字段）
   + sparse_init.npz（COLMAP 稀疏点过滤后 1092 点，抖动 x25 = 27300 点，sigma 8mm）
5. `train_3dgs_v3.py` 已支持纯 RGB 分支：无 depth_scale 字段时自动跳过深度加载/深度损失，
   有 sparse_init.npz 时用它替代深度反投影初始化

12k 迭代结果（MCMC，200k 高斯上限）：
| 指标 | 夜景深度路线（8帧） | **白天纯 RGB（34帧）** |
|---|---|---|
| 验证帧 PSNR | 8.81（1帧） | **13.0 均值（4帧: 14.0/12.6/13.2/12.1）** |
| 训练 PSNR | 22.1（7帧易过拟合） | 17.7（30 帧，更难拟合） |
| 验证视图视觉 | 絮状伪影，结构不可辨 | 植株行/花盆/管道/果实结构清晰，有模糊 |

局限与下一步：
- 覆盖只含模型 1 的 34 帧（整条垄约 1/4）；其余 7 个组件（28/30/17 帧）可各自独立重建
- 训练 30k 迭代 + 合并相邻组件（各自坐标系独立，需逐段处理）预计再提升
- 任意尺度（轨迹 12.9m 与真实距离同量级，尺度接近米制但未标定）；放入 ManiSkill 前需尺度锚定
- 白天自然光外观（绿色植株+橙色果实）比夜景补光外观更接近真实温室，作为 sim 背景更合理



## 〇、2025-09-03 更新：v3 官方配方训练器

`scripts/train_3dgs_v3.py` 把训练流程对齐 gsplat 官方 `simple_trainer.py` 主流配方（无需克隆任何库，
全部用已装 gsplat 1.5.3 自带组件）：

| 主流配方项 | v2 | v3 |
|---|---|---|
| 自适应密度控制 | ❌ 固定 6 万 | ✅ MCMCStrategy（默认，cap 20万）或 DefaultStrategy（Clone/Split/Prune） |
| 球谐 SH degree 3 | ❌ 纯 RGB | ✅ 16 系数，每 1000 迭代升一阶（夜间移动补光=视角相关外观，SH 正好吸收） |
| 图像损失 | MSE | ✅ 0.8·L1 + 0.2·D-SSIM（官方配比） |
| 参数化 | 真值空间 | ✅ scales 存 log / opacities 存 logit（策略要求），每参数独立 Adam |
| 学习率 | 恒定 | ✅ 指数衰减到 1%，means lr 随场景尺度缩放 |
| 初始化 | scale 0.008/opacity 0.85 | ✅ kNN-3 均值定尺度，opacity 0.1（官方值） |
| 深度正则 | L1 权重 0.5 | ✅ 权重 0.1（可调，L515 夜间深度噪声大） |

用法：
```bash
export CUDA_HOME=~/tomato_robot/cuda_toolkit PATH=~/tomato_robot/.venv/bin:$PATH
.venv/bin/python scripts/train_3dgs_v3.py --data data/night_3dgs_v2 --iters 30000
# --strategy mcmc|default  --cap-max 200000  --depth-weight 0.1
```

v3 实现中踩过的 gsplat 坑（复用官方组件时都会遇到）：
1. 策略约定参数存 log(scales)/logit(opacities)，但调 `rasterization` 必须换回真值空间
   （`exp`/`sigmoid`/四元数归一化）——否则负 opacity 导致全黑渲染、loss 不降；
2. SH 模式下 `backgrounds` 通道数须等于 SH 总维数（RGB+D 还会 +1 深度通道）——传 `None` 最稳；
3. `rasterization` 默认 `packed=True`（info 为 [nnz,...] 稀疏格式），配 `DefaultStrategy`
   必须传 `packed=True`，否则 `_update_state` 按 [C,N,...] 解析崩溃；
4. conda-forge 新版 CUDA 包头文件在 `targets/x86_64-linux/include/`，需符号链接到 `include/`；
5. CUDA toolkit 已固定到项目内 `cuda_toolkit/`（不再依赖易失的 /tmp）。

### v2 vs v3 同数据同迭代数对比（8 帧数据，12000 迭代，2025-09-03）

| 指标 | v2（MSE+固定28k高斯） | v3（官方配方 MCMC+SH+DSSIM） |
|---|---|---|
| 训练视角 PSNR | ~16.6（MSE 0.0217） | **~20+ 峰值 22.1**（L1 0.037） |
| 高斯数 | 28000 固定 | 200000（MCMC 自动增减） |
| 训练视图渲染 | 模糊、结构糊成一片 | 果实/主茎/管道清晰可辨 |
| 验证帧 PSNR | 8.82 | 8.81 |
| 验证帧渲染 | 絮状伪影 | 絮状伪影（程度相近） |

**结论**：v3 在训练视图质量上全面胜出（这正是主流配方的价值）；但验证视图两版同样失败，
PSNR 都在 8.8 附近——瓶颈在数据（7 训练帧撑不起新视角合成），与训练器无关。
要验证帧 PSNR 上 15+，必须扩充训练帧（night_map 最优段 16 帧起）。

数据侧结论同步更正：v1 的 `data/night_3dgs/transforms.json` 内参是 COLMAP 3 帧坏自标定
（fl_x=614/fy=937，且相机位置漂到 ±5m），不可用；v2/v3 统一用官方内参 + ICP 米制位姿。


## 一、项目目标

基于 L515 夜间 RGB-D 数据，使用 3D Gaussian Splatting（3DGS）重建温室场景，为 VLA 模型训练提供合成视图和深度数据。

---

## 二、已完成工作

### 2.1 数据分析与筛选

| 数据源 | 帧数 | 有效深度率 | 平均深度 | 结论 |
|--------|------|-----------|---------|------|
| L515 夜间 | 122 | 55–71% | 1.69m | **可用** |
| L515 白天 | 122 | 4–15% | — | 深度噪声大 |
| D435 白天 | ~120 | 2–36% | 0.44m | 距离太近 |
| D435 夜间 | 119 | 低 | — | 质量差 |

- 从 122 帧夜景中筛选出 20 帧高质量帧（有效深度率 >55%），复制到 `data/night_3dgs/`
- 相机内参已验证：L515 原始 1920×1080，实际输出 1280×720，缩放后 fx=904.135, fy=904.542, cx=646.646, cy=352.313

### 2.2 gsplat 安装与编译

**问题**：pip 预编译的 gsplat 1.5.3 wheel 不包含 CUDA 扩展（`_C` 模块为 None），需要本地 JIT 编译。

**解决方案**：
1. 通过 conda 创建独立环境 `/tmp/cuda_env`，安装 `cuda-nvcc=12.4` 和 `cuda-cudart-dev=12.4`
2. 设置环境变量 `CUDA_HOME=/tmp/cuda_env`，将 `.venv/bin` 加入 PATH
3. gsplat 自动检测 CUDA toolkit 并完成 JIT 编译（约 55 秒）

**编译命令**（后续每次使用需设置）：
```bash
export CUDA_HOME=/tmp/cuda_env
export PATH=/home/lcw/tomato_robot/.venv/bin:$PATH
```

### 2.3 COLMAP 位姿估计

编写了 `scripts/colmap_night.py`，针对夜景窄基线序列放宽参数：
- `init_min_tri_angle = 0.1°`（默认 16°）
- `init_min_num_inliers = 5`（默认 100）
- `abs_pose_min_num_inliers = 5`（默认 30）

**结果**：20 帧中仅 3 帧被成功注册（图像名：`192308`、`192318`、`192326`），生成 9 个稀疏 3D 点。

**失败原因**：温室沿垄道行走采集，帧间基线极短（1–5 秒间隔），几何约束不足。

导出脚本 `scripts/export_colmap_poses.py` 将 COLMAP 位姿转换为 `transforms.json` 格式。

### 2.4 3DGS 训练

编写了 `scripts/train_3dgs.py`，流程：
1. 加载 COLMAP 稀疏点云 + 深度图反投影点云作为高斯初始化
2. 使用 gsplat 光栅化，RGB 损失 + 深度损失联合优化
3. 输出高斯参数（`gaussians.pt`）和点云（`point_cloud.ply`）

**训练配置**：
- 1500 个高斯，3000 迭代
- 3 帧参与训练（受 COLMAP 限制）
- 损失函数：L2 图像损失 + L1 深度损失（权重 0.5）

**训练结果**：
| 迭代 | 图像损失 | 深度损失 | PSNR |
|------|---------|---------|------|
| 500 | 0.0228 | 0.3621 | 16.4 |
| 1500 | 0.0169 | 0.2830 | 17.7 |
| 3000 | 0.0162 | 0.2566 | 17.9 |

输出文件：`output/3dgs_night/gaussians.pt`（85K）、`output/3dgs_night/point_cloud.ply`（47K）

---

## 三、当前文件结构

```
tomato_robot/
├── data/night_3dgs/
│   ├── images/          # 20 帧夜景 RGB
│   ├── depth/           # 20 帧夜景深度图
│   ├── sparse/0/        # COLMAP 输出（cameras.bin, images.bin, points3D.bin）
│   └── transforms.json  # 导出的相机位姿
├── scripts/
│   ├── colmap_night.py          # COLMAP 位姿估计（放宽参数）
│   ├── export_colmap_poses.py   # COLMAP 位姿导出
│   └── train_3dgs.py            # 3DGS 训练脚本
├── output/3dgs_night/           # 3DGS 输出
│   ├── gaussians.pt
│   └── point_cloud.ply
└── recon/night_recon/           # TSDF 重建结果（之前的工作）
    ├── night_scene.ply
    └── night_simple.ply
```

---

## 四、已知问题

1. **COLMAP 注册率极低**：3/20 帧，3DGS 训练帧数不足
2. **无自适应密度控制**：当前 1500 个高斯是固定数量，缺少 Split/Clone/Densify 操作
3. **深度监督信号弱**：深度损失收敛慢，可能因为深度图噪声

---

## 五、下一步计划

### 5.1 解决 COLMAP 注册率问题

**方案 A — 跳过 COLMAP，直接用深度图对齐位姿**：
- 用 ICP 或 PnP 从深度图估计帧间相对位姿
- 不依赖外观特征，对夜景更鲁棒

**方案 B — 扩大 COLMAP 基线**：
- 间隔选取帧（如每隔 5 帧取 1 帧），增大基线/深度比
- 手动指定初始图像对

**方案 C — 使用已知运动学模型**：
- 机器人沿固定轨道移动，可用匀速运动模型初始化位姿
- 再用光束法平差（BA）精化

### 5.2 增强 3DGS 训练

- 实现自适应密度控制（gsplat 的 `DefaultStrategy` 或 `MCMCStrategy`）
- 增加训练帧数（目标：10–20 帧）
- 调整深度损失权重，尝试平滑 L1 损失
- 添加 SSIM 损失提升感知质量

### 5.3 评估与可视化

- 从训练好的高斯中渲染新视角，与原始图像对比
- 提取网格（Marching Tetrahedra 或 TSDF 融合高斯点云）
- 导出为 ManiSkill 可用格式

---

## 六、关键环境变量

每次使用 gsplat 前需设置：
```bash
export CUDA_HOME=/tmp/cuda_env
export PATH=/home/lcw/tomato_robot/.venv/bin:$PATH
```

conda 环境位于 `/tmp/cuda_env`，包含 CUDA 12.4 toolkit（nvcc + cudart dev headers）。

---

## 七、2026-09-06 更新：gaussians.ply 渲染体检（M0 验收检查）

用新脚本 `scripts/render_gaussian_ply.py` 对 `output/3dgs_day_rgb/gaussians.ply`（20 万高斯，Inria 格式）做了渲染复查，渲染结果在 `output/3dgs_day_rgb/renders/`（3 张验证帧 + 1 张相邻帧插值新视角）。

| 检查项 | 结果 | 判定 |
|---|---|---|
| 验证帧 PSNR 复测 | 14.08 / 12.62 / 13.28 dB（均值 13.3，与训练时报告的 13.0 一致，渲染管线可信） | 与预期相符 |
| 新视角渲染 | 结构可辨（栽培槽/盆/冠层/橙色果串），但整体模糊、拖影、半透明鬼影；无絮状爆炸伪影 | 仅够开发靶场，不够照片级 |
| 空间跨度（op>0.5 的 1~99%） | 15.7 × 2.9 × 7.0 m——株高方向 7.0m 超出番茄实际株高（1.5~2.5m）2~4 倍 | **米制不可信，尺度未锚定** |
| 离群高斯 | 极端离群（>20m 外）仅 108 个（0.1%），可裁 | 小问题 |
| 环境变更 | gsplat JIT 缓存失效后重编译：`ninja` 已装入 venv；CUDA_HOME 改用仓库持久化工具链 | 见下方新环境变量 |

**M0 结论（对照 sim 里程碑验收标准）**：场景单元 ✅（34 帧垄段）；PLY 导出 ✅；**尺度锚定 ❌（株高方向膨胀 2~4 倍 + 报告已证实的非均匀漂移）；质量 ❌（13dB，需补采后预期 25dB+）**。当前 PLY 定位 = M1/M2（MuJoCo 物理、高斯↔刚体绑定、数据管线）的开发靶场，不是最终视觉素材。

**环境变量更正**：`/tmp/cuda_env` 会被系统清理导致 JIT 重编译，今后改用仓库内工具链：

```bash
export CUDA_HOME=/home/lcw/tomato_robot/cuda_toolkit
export PATH=/home/lcw/tomato_robot/.venv/bin:$PATH
```

---

## 八、2026-09-06 更新：M0 收尾执行 + M1 开工

### M0 收尾（本轮完成）

| 项 | 结果 |
|---|---|
| 离群高斯裁剪 | `scripts/crop_gaussian_ply.py` → `output/3dgs_day_rgb/gaussians_clean.ply`（裁掉 674/200000，0.3%） |
| 尺度锚定工具 | `scripts/anchor_scale.py`；网格标尺图 `output/3dgs_day_rgb/gaussians_clean_grid.png`（XY/XZ/YZ 三视图，1m 网格） |
| **待用户完成** | 温室里量两个可辨认点的真实距离 → 在网格图上读两点坐标 → `--from-measure x1 y1 z1 x2 y2 z2 --real D` 一步完成锚定 |

**认知修正（对第七节）**：网格三视图显示 COLMAP 世界系相对重力**倾斜**（YZ 视图中垄沿对角线延伸），第七节"株高方向膨胀 2~4 倍"的表述不成立——Z 向 7m 跨度混入了坐标系倾斜与沿程漂移，不能解读为纯尺度膨胀。结论不变且更强：**必须物理测量锚定，且锚定后仅保证物理开发够用**。

### M1 开工（物理侧）

| 项 | 结果 |
|---|---|
| MuJoCo | venv 安装 3.12.0 |
| 机器人资产 | `third_party/aubo_ros2_driver`（含子模块 aubo_description 332MB，官方 URDF+mesh） |
| URDF→MJCF | `scripts/aubo_urdf_to_mjcf.py`：package:// 路径改写、视觉网格统一指到碰撞 STL（3ds/DAE 不被 MuJoCo 支持）、注入 compiler 选项；**注意 `<mujoco>` 必须是 `<robot>` 子元素，放外面会被解析成空 MJCF** |
| 加载体检 | nq=6/nv=6（6 自由度 ✓）、7 连杆、总质量 21.99kg（官方 ~27kg，量级正确）、零位最大连杆距基座 1.008m（米制 ✓，臂展 0.8865m） |
| 产物 | `third_party/aubo_i5_mujoco/scene_aubo_i5.xml`（MJCF，无执行器，待后续加 ground/果串占位/可断约束/执行器） |

**M0 状态：除"实测量距"外全部关账。下一步 = 用户温室量距（10 分钟）→ M1 场景搭建（地面 + 果串占位体 + 可断 weld + 执行器 + 脚本化接近动作）。**
