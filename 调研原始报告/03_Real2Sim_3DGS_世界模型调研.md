# Real2Sim / 3DGS / 世界模型 用于机器人操作训练调研报告
> 调研子任务:3DGS 机器人仿真器 / 物理+神经渲染混合 / 场景重建工具链 / 世界模型 / 三路线对比
> (面向温室番茄串采摘 VLA 仿真,调研截至 2026-09,主要条目均已通过 arXiv/项目页/GitHub 核实)

## 一、3DGS 作为机器人操作仿真器

**机制范式**:目前所有工作都遵循"物理引擎管动力学、3DGS 管渲染"的解耦思路——物理引擎(Isaac Sim/MuJoCo/SAPIEN)输出物体与机械臂的位姿序列,3DGS 场景中对应物体的 splats 跟随刚体变换(或按关节层级运动)后重新光栅化,得到照片级观测。区别主要在重建对象(是否含机械臂)、闭环与否、数据用途(训练 vs 评测)。

| 工作 | 机制与支持操作 | 开源 | 关键限制 |
|---|---|---|---|
| **SplatSim** (arXiv 2409.10161) | 重建场景+机械臂为 splats,与仿真器点云对齐;机械臂 splats 随关节变换;纯 RGB 策略零样本 sim2real。任务:Push-T、抓苹果等 4 类刚体 | 是,[GitHub](https://github.com/qureshinomaan/SplatSim) | 刚体为主;抓取接触瞬间/splat 穿越无物理修正;86.25% vs 真实数据 97.5% 的成功率差距 |
| **RoboGSim** (arXiv 2411.11839) | Isaac Sim 数字孪生 + 3DGS:Gaussian Reconstructor(机械臂按 MDH 关节图驱动)/Digital Twin Builder/Scene Composer(物体替换、新视角)/Interactive Engine(闭环评测+数据合成) | 项目页 [robogsim.github.io](https://robogsim.github.io/);GitHub repo 存在但许可未核实 | 作者自述:仅刚体;合成物体光照不统一;mesh 反求质量差 |
| **GSWorld** (arXiv 2510.20813, ICRA 2026) | 提出 GSDF(Gaussian Scene Description Files)中间格式,把"机器人+物体+相机"绑定,与 MuJoCo/Isaac 闭环耦合;支持零样本 sim2real 训练、闭环策略评测、真机轨迹复现 | 是,[GitHub](https://github.com/luccachiang/GSWorld) | 刚体操作基准;可变形/遮挡区域重建仍需人工 |
| **GaussGym** (arXiv 2510.15352) | 3DGS 作为 vectorized 物理引擎的 drop-in 渲染器,10 万+ 步/秒训练(locomotion,非操作) | 是,[gauss-gym.com](https://gauss-gym.com/) | 不面向操作 |
| **RoboSplat** (arXiv 2504.13175, RSS 2025) | 不是仿真器而是**演示增广**:重建 1 条演示的 3DGS 场景,直接操纵高斯(移物体、换视角、改夹爪/光照)生成新演示,实现 one-shot 模仿学习 | 是,[GitHub](https://github.com/InternRobotics/RoboSplat) | 需 1 条真机演示;无形变处理 |
| **Splat-MOVER** (arXiv 2405.04378) | ASK-Splat(CLIP 特征蒸馏)→SEE-Splat(场景编辑模拟操作后状态)→Grasp-Splat;开词汇推动类操作 | 是,[GitHub](https://github.com/StanfordMSL/Splat-MOVER) | 编辑是几何平移,非物理;仅推/移 |
| **GaussianGrasper** (arXiv 2403.09637, RA-L 2024) | E-EVSM 语义蒸馏 + 开词汇抓取位姿生成,实时闭环抓取(真机感知,非训练仿真) | 是,[GitHub](https://github.com/MrSecant/GaussianGrasper) | 感知管线而非训练环境 |
| **VR-Robo** (arXiv 2502.01536) | Real2Sim2Real:GS 数字孪生为导航/locomotion 提供 egocentric 真实感观测 | 项目页公开;代码未核实 | 非操作 |

**名称澄清**:未检索到名为 **"GS-Sim"** 的论文(未核实,疑与 GaussGym/GSWorld 混淆);**Sim-on-Wheels**(arXiv 2306.08807, RA-L 2023)是自动驾驶 vehicle-in-loop 工作,非操作领域——操作领域对应物是 **SimplerEnv**("Evaluating Real-World Robot Manipulation Policies in Simulation", arXiv 2405.05941, CoRL 2024)。

**代表性结论(sim2real 相关性证据)**:SimplerEnv 证明经"Visual Matching"(真实图像贴入仿真背景+控制频率/系统辨识校正)后,SAPIEN/Isaac 内评测的成功率与真机排名呈强 Spearman/Kendall 相关——即**视觉一致性是 sim2real 相关性的第一决定因素**。SplatSim 把渲染换成 3DGS 后零样本迁移成功率达 86.25%(真机数据训练为 97.5%);RoboGSim 用合成数据训练的策略在常规场景达真机数据 90% 水平、在新场景反超(90% vs 60%)。2025 年的新趋势是 GSWorld/real2sim-eval([real2sim-eval.github.io](https://real2sim-eval.github.io),arXiv 2511.04665)把"3DGS 渲染 + 软体数字孪生"用于**闭环策略评测**,这是目前证据最强的路线。

**共性硬限制**(对温室场景至关重要):① 绝大多数只支持刚体——抓取时物体被当作刚体位姿驱动,**不存在果梗剪断、果串脱落的拓扑变化事件**;② 多物体间遮挡与动态遮挡边界的重建伪影明显(温室枝叶重叠正是重灾区);③ 机器人 splat 与物体 splat 的光照/阴影不完全一致;④ 无接触力学,夹爪-果梗交互需由外部物理引擎假设决定。

## 二、物理 + 神经渲染混合(可变形物体)

| 工作 | 机制 | 从真实视频标定? | 交互/实时性 | 对茎叶适用性 |
|---|---|---|---|---|
| **PhysGaussian** (arXiv 2311.12198, CVPR 2024, [GitHub](https://github.com/XPandora/PhysGaussian)) | MPM 连续介质力学直接驱动高斯核 | 否(参数手设,物理性质可由 DreamPhysics 2406.01476 用视频扩散先验估计) | 离线为主 | MPM 可表达叶片各向异性,但参数难定 |
| **PhysDreamer** (arXiv 2404.13026) | MPM + 视频扩散先验作为"交互监督"标定刚度,输出可交互 3D 物体 | 部分(用生成的视频回标) | 准实时 | 适合单株软体,细枝干断裂不支持 |
| **Spring-Gaus** (arXiv 2403.09434, ECCV 2024) | 弹簧质点网络嵌入 3DGS,免解 PDE | 是(多视角交互视频) | **实时** | 弹簧拓扑可自定义——最接近枝条-果串铰接结构 |
| **GauSim/GausSim** (arXiv 2412.17804, ICCV 2025) | 学习型高斯模拟器,层级 coarse-to-fine | 是(自建 READY 多视角数据集) | 快于 MPM | 中等 |
| **PhysTwin** (arXiv 2503.17973, ICCV 2025, [GitHub](https://github.com/jianghanxiao/phystwin), [项目页](https://jianghanxiao.github.io/phystwin-web/)) | 弹簧质点 + 3DGS 渲染,多阶段逆优化从稀疏交互视频同时恢复几何/密度物理参数/外观 | **是,这是它最大卖点**(支持遮挡、有限视角) | **全实时、键盘交互、支持机器人操作规划(优化式 planning)** | 官方支持 rope/stuffed/cloth/package——绳类模型与"果梗+果串"结构同构性最好 |

**对温室场景的判断**:这条线已从"离线 4D 生成"演进为"可从视频标定的实时交互仿真器"(PhysTwin 一步);但**所有工作都不支持断裂/分离事件**(cutting 需要改变弹簧/粒子拓扑,PhysGaussian/PhysTwin 均未实现;MPM 断裂模拟存在但未进入 3DGS 管线,未核实有现成开源实现)。因此"剪果梗"动作可建模为:弹簧拓扑中人工删除果梗-枝条连接边(事件触发式),这在 PhysTwin 代码框架内可改造,属定制开发而非开箱即用(判断,未核实有公开实现)。

## 三、场景重建管线与工具链(2025-2026)

- **位姿/几何初始化**:COLMAP 仍是基线,但温室重复纹理(成排同款叶片)+ 玻璃反光是 SfM 噩梦;**VGGT**(arXiv 2503.11651,CVPR 2025 Best Paper,[GitHub](https://github.com/facebookresearch/vggt))前馈秒级输出相机参数/深度/点图,配合 DUSt3R(2312.14132)/MASt3R(2406.09756)系,对弱纹理、重复结构的鲁棒性显著更好,已是 2026 年推荐首选。MASt3R-SLAM(2412.12392)可做视频流式方案。
- **3DGS 训练**:[nerfstudio](https://docs.nerf.studio)(splatfacto)/[gsplat](https://github.com/nerfstudio-project/gsplat)(CUDA 库)开源主力;[Postshot](https://www.jawset.com/)(Jawset,Windows,免费 beta,RTX 20 系+)适合快速试错与整机产出。
- **分割/物体级提取**:SAM2(视频一致掩码)+ 特征蒸馏:LangSplat(2312.16084, CVPR 2024)、Feature Splatting(2404.01223,蒸馏 CLIP 且**接物理引擎做语言驱动场景编辑**——机制上与需求最贴);Gaussian Grouping(2312.00732, ECCV 2024)支持物体删除/重定位;OmniSeg3D(arXiv 2403.08484,未核实编号)。实践上 SAM2 逐帧 + 3D 一致性投票(如 GSWorld 管线)比纯特征蒸馏更可控。
- **物体资产生成**:TRELLIS(2412.01506, [GitHub](https://github.com/microsoft/TRELLIS),输出 Gaussian/Radiance/Mesh 三格式)、Hunyuan3D 2.0(2501.12202, [GitHub](https://github.com/Tencent-Hunyuan/Hunyuan3D-2),带纹理 mesh)、InstantMesh(2404.07191)。**可以**用单张番茄串照片生成"番茄串+果梗"资产:用 TRELLIS 生成 mesh → MuJoCo/Isaac 中做碰撞体 → 再转回高斯渲染(需 gaussian↔mesh 绑定,SplatSim/GSWorld 已提供该绑定代码)。风险:生成资产的几何细节(细果梗直径、萼片)不可靠,建议"果串从扫描提取、果梗参数化重建"混合策略。
- **大场景扫描**:GreenhouseSplat(arXiv 2510.01848)已证明用**廉价 RGB 视频重建照片级温室资产**用于移动机器人仿真;PlantGaussian(Plant Phenomics 2025)、Ushiroji et al.(2026, 3DGS 番茄栽培场景 + UE5 合成采摘数据)验证了整排作物可行。算力估计(经验值,未核实文献):单排温室 300-800 张图,RTX 4090 训练 1-4 小时,显存 24GB 够用;整栋温室建议分株/分排重建再拼接,而不是一次重建全场。**手机视频扫描优于 RealSense RGBD**:分辨率与视差质量更高;RealSense 深度在日光/IR 强干扰的温室内噪声大(经验判断,未核实),但 RGBD 可提供尺度先验用于 VGGT 结果校准。

## 四、世界模型 / 视频生成作为仿真器(2025-2026)

- **NVIDIA Cosmos**(arXiv 2501.03575):平台级 WFM;**Cosmos-Transfer1**(arXiv 2503.14492,[GitHub](https://github.com/nvidia-cosmos/cosmos-transfer1),开源)用分割/深度/边缘多条件把仿真视频"转写"成真实感视频(sim2real 增广);**Cosmos-Predict2**([GitHub](https://github.com/nvidia-cosmos/cosmos-predict2))支持 **action-conditioned post-training**——输入末端执行器动作序列生成未来视频,2B 权重已在 HF 开源([样例](https://huggingface.co/nvidia/Cosmos-Predict2-2B-Sample-Action-Conditioned))。视频长度仅数秒、几何一致性无保证,定位是"数据增广"而非"可评测仿真器"。
- **DreamGen**(arXiv 2505.12705,NVIDIA GEAR,MLLR v305):4 阶段管线,用视频世界模型生成 **neural trajectories**(合成机器人轨迹)训练 VLA/策略,在跨行为、跨环境泛化上显著提升——这是"世界模型当仿真器训练策略"目前最强的正面证据,但它依赖 Action-conditioned 后训练良好的领域世界模型(需大量领域演示视频)。
- **Genie 3**(DeepMind,2025-08,[官方博客](https://deepmind.google/blog/genie-3-a-new-frontier-for-world-models/)):文本→实时可交互世界,未开源、无公开 API,对学术/产业不可用(截至调研日未核实到开放计划)。
- **Gaia-2**(arXiv 2503.20523,Wayve):多相机可控驾驶世界模型,其"结构化条件控制(ego 动作、天气、布局)"思路值得借鉴,但领域锁定驾驶;GAIA-3 已发布 15B 版(未核实细节)。
- **1X World Model**(1XWM,[博客](https://www.1x.tech/discover/world-model-self-learning)):视频预训练世界模型集成进 NEO 人形做策略,数百小时第一人称数据,未开源。
- **iVideoGPT**(arXiv 2405.15223, [GitHub](https://github.com/thuml/iVideoGPT)):轻量级交互世界模型(动作+观测 token 自回归),百万级机器人轨迹预训练,适合低算力试水。
- **对罕见农业场景的泛化**:预训练视频分布中温室第一人称操作数据稀少(推断,未核实系统研究),需用温室扫描视频 + Cosmos-Transfer 风格条件生成做领域后训练;数据量经验需求为小时级领域视频(未核实精确阈值)。

## 五、综合判断:2026 年最现实的 Real2Sim 管线(针对"有真实温室、无遥操、机器人未就绪")

建议主线走 **B 路线(3DGS real2sim + 物理)**,用 A 做补充、C 做增广。具体步骤与工具:

1. **扫描**:手机(iPhone/带 LiDAR 更好)沿单排番茄缓慢绕行拍视频,每串果 15-30° 覆盖;果梗剪切点拍特写。工具:手机 + 稳定云台(无特殊要求)。
2. **位姿/重建**:VGGT(2503.11651)出相机位姿与点图 → COLMAP 校验 → gsplat/splatfacto 或 Postshot 训 3DGS;单排温室为独立场景单元。
3. **分割与物体化**:SAM2 出视频一致掩码 → Gaussian Grouping(2312.00732)提升到 3D 实例 → 把"番茄串(果+梗)"与"茎叶/支架"分离;预留 Feature Splatting(2404.01223)做语言可查询实例。
4. **资产化与物理接入**:果串 gaussian 绑到 MuJoCo/Isaac 刚体(复用 SplatSim/GSWorld 的绑定代码);果梗用弹簧质点(借 PhysTwin 参数标定思想,用剪切/拉拽视频标定刚度);"剪断"实现为弹簧边删除事件(定制开发点)。
5. **渲染与数据生成**:GSWorld 式闭环:物理引擎状态 → 3DGS 重渲染 → 生成 (观测, 动作, 下一观测) 轨迹;配合 RoboSplat 式高斯编辑做物体位姿/光照多样性增广。
6. **策略训练与评测**:先用 SimplerEnv 协议在自建"温室评测集"上做闭环评测;真机就绪后用同一 GS 场景做 zero-shot sim2real。
7. **并行轻量项**:用 Cosmos-Transfer1 把物理引擎渲染转成照片级视频、或 Cosmos-Predict2 action-conditioned 后训练,作为 VLA 数据增广(风险低、见效快)。

## 三条可行技术路线对比

| 维度 | A: 传统仿真引擎手建(Isaac/MuJoCo + UE/Blender 资产) | B: 3DGS real2sim + 物理(SplatSim/GSWorld/PhysTwin 混合) | C: 世界模型/视频生成(Cosmos/DreamGen 路线) |
|---|---|---|---|
| 成熟度(2026) | 高(工具链全);温室植物资产需自制 | 中:刚体部分成熟(GSWorld/SplatSim 已验证),柔性/剪断部分属前沿 | 低-中:DreamGen 已验证有效但依赖大算力与领域数据 |
| 视觉真实感 | 低-中(温室植物尤其假) | **高**(直接来自真实扫描,天然解决"视觉不真实"痛点) | 高但不可控(有幻觉,几何不保证) |
| 物理可信度 | **高**(成熟求解器;但植物参数仍需标定) | 中:刚体好;柔性可从视频标定(PhysTwin);剪断事件需自研 | 低-无(视频先验替代物理,无法做精确碰撞/力反馈) |
| 成本(人力+算力) | 中:建模耗时(植物难建),算力低 | 中:扫描+重建+绑定工程量大;RTX 4090 级即可;无需遥操 | 高:世界模型后训练需多卡 A100/H100 级;数据需小时级领域视频 |
| 主要风险 | 视觉 sim2real gap 大,VLA 对外观敏感 → 迁移差 | 遮挡/动态伪影;剪断形变无人做过;多排场景拼接工程风险 | 农业场景分布外泛化差;不可评测;开源生态变动快 |
| 综合评分(1-5) | 3.0 | **3.8(推荐主线)** | 2.5(作增广副线) |

**核心结论**:约束条件(无遥操、可随意扫描、机器人未就绪)恰好是 B 路线的最优适用面——3DGS real2sim 用扫描替代遥操演示、用渲染替代真实机器人,而"剪断果梗"这一前沿空白(分离事件)是必须自研、也是可发表的贡献点;C 路线建议仅以 Cosmos-Transfer 照片级增广形式引入以摊薄风险。

## Sources
[SplatSim](https://arxiv.org/abs/2409.10161) · [RoboGSim](https://arxiv.org/abs/2411.11839) · [GSWorld](https://arxiv.org/abs/2510.20813) · [GaussGym](https://arxiv.org/abs/2510.15352) · [RoboSplat](https://arxiv.org/abs/2504.13175) · [Splat-MOVER](https://arxiv.org/abs/2405.04378) · [GaussianGrasper](https://arxiv.org/abs/2403.09637) · [VR-Robo](https://arxiv.org/abs/2502.01536) · [SimplerEnv](https://arxiv.org/abs/2405.05941) · [Sim-on-Wheels](https://arxiv.org/abs/2306.08807) · [PhysGaussian](https://arxiv.org/abs/2311.12198) · [PhysDreamer](https://arxiv.org/abs/2404.13026) · [Spring-Gaus](https://arxiv.org/abs/2403.09434) · [GausSim](https://arxiv.org/abs/2412.17804) · [PhysTwin](https://arxiv.org/abs/2503.17973) · [DreamPhysics](https://arxiv.org/abs/2406.01476) · [VGGT](https://arxiv.org/abs/2503.11651) · [DUSt3R](https://arxiv.org/abs/2312.14132) · [MASt3R](https://arxiv.org/abs/2406.09756) · [Feature Splatting](https://arxiv.org/abs/2404.01223) · [LangSplat](https://arxiv.org/abs/2312.16084) · [Gaussian Grouping](https://arxiv.org/abs/2312.00732) · [TRELLIS](https://arxiv.org/abs/2412.01506) · [Hunyuan3D 2.0](https://arxiv.org/abs/2501.12202) · [InstantMesh](https://arxiv.org/abs/2404.07191) · [GreenhouseSplat](https://arxiv.org/abs/2510.01848) · [Cosmos](https://arxiv.org/abs/2501.03575) · [Cosmos-Transfer1](https://arxiv.org/abs/2503.14492) · [cosmos-predict2](https://github.com/nvidia-cosmos/cosmos-predict2) · [DreamGen](https://arxiv.org/abs/2505.12705) · [Genie 3](https://deepmind.google/blog/genie-3-a-new-frontier-for-world-models/) · [Gaia-2](https://arxiv.org/abs/2503.20523) · [iVideoGPT](https://arxiv.org/abs/2405.15223) · [1X World Model](https://www.1x.tech/discover/world-model-self-learning) · [Postshot](https://www.jawset.com/)
