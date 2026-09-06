# 番茄串采摘机器人 VLA 技术路线调研(Tomato Robot Paper Finding)

面向温室番茄串采摘机器人的 VLA(Vision-Language-Action)模型训练体系调研:从一篇田间验证论文出发,梳理 2024–2026 年 VLA 模型、机器人仿真平台、Real2Sim(3DGS)与世界模型四条前沿线的最新进展,给出可落地的仿真环境搭建技术路线。

## 基准论文

Xu, C., Xu, Z., Li, H., & Zhou, Y. (2026). **Design, Development, and Field Testing of a Tomato Bunch Harvesting Robot.** *Journal of Field Robotics*. https://doi.org/10.1002/rob.70250

- 系统:Aubo i5 机械臂 + 自研剪切-夹持一体化末端 + 双 RealSense L515(eye-to-hand)+ 移动底盘
- 田间结果:采摘成功率 70.77%,12.23 s/串;失败原因中感知系统占 57.9%、机械臂被主茎阻挡占 21.05%

## 仓库内容

| 文件 | 说明 |
|---|---|
| [VLA技术路线调研与落地方案_2026-09.md](./VLA技术路线调研与落地方案_2026-09.md) | **主报告**:论文解读 + VLA/仿真/Real2Sim 全景 + 三条技术路线对比 + 12–18 个月落地方案 |
| [调研原始报告/01_VLA模型与数据采集路线调研.md](./调研原始报告/01_VLA模型与数据采集路线调研.md) | π0/π0.5、OpenVLA-OFT、GR00T 等模型选型;微调数据量;GELLO/Quest3 遥操路线;HarvestFlex 案例解读 |
| [调研原始报告/02_机器人仿真平台对比调研.md](./调研原始报告/02_机器人仿真平台对比调研.md) | Isaac Sim / Genesis / MuJoCo / ManiSkill3 对比;可变形体与断裂物理支持现状;Aubo i5 资产来源 |
| [调研原始报告/03_Real2Sim_3DGS_世界模型调研.md](./调研原始报告/03_Real2Sim_3DGS_世界模型调研.md) | SplatSim / GSWorld / PhysTwin / GreenhouseSplat;扫描→重建→实例化→物理绑定管线;Cosmos/DreamGen 世界模型 |
| [调研原始报告/04_农业温室仿真与植物建模调研.md](./调研原始报告/04_农业温室仿真与植物建模调研.md) | 温室操作仿真空白验证;OrchardBench/aoc_tomato_farm/Helios;番茄果梗力学参数;公开数据集清单 |

## 核心结论(2026-09)

1. **开源的"操作级"温室采摘仿真环境目前不存在**,但全部必要组件已就绪(OrchardBench 果实脱离物理、SplatSim/GSWorld 3DGS 仿真框架、PhysTwin 视频标定柔性体、GreenhouseSplat 温室重建);"番茄串 VLA"是尚无发表的空白方向。
2. **视觉渲染难点的解法是 Real2Sim**:手机扫描真实温室 → VGGT → 3DGS → SAM2 实例化,照片级观测与真机同分布。
3. **物理仿真难点的解法是事件触发断裂**:不做连续介质剪切过程仿真,果梗 = 可断裂弹簧约束(文献标定剪切力 8.6–19.9 N),刀片到位即触发分离。
4. **仿真与遥操双轨并行**:仿真解决"当前无数据";GELLO(<$300)/Quest3 遥操台不依赖整机改造,应立即启动,机器人就绪后 50–200 条示教 + 失败恢复数据微调 π0(LoRA,单卡 4090 可行)。

> 注:依据论文 PDF 及各出版方版权要求,原始论文文件不入本仓库,请通过 DOI 自行获取。
