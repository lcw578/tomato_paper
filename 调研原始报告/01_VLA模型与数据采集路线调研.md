# 温室番茄串采摘机器人 VLA 训练体系调研报告(2026-09)
> 调研子任务:VLA 模型全景 / 预训练数据集 / 微调数据需求 / 无仿真数据采集路线 / Sim-to-Real 真实案例 / 农业与采摘类 VLA
> 所有关键事实均经 WebSearch/WebFetch 核实(来源链接见文末),未经核实处已标注。

## 一、VLA 模型全景

| 模型(年份/arXiv) | 架构要点 | 开源 | 规模/微调算力 |
|---|---|---|---|
| **π0**(2024.10, [2410.24164](https://arxiv.org/abs/2410.24164)) | PaliGemma 骨干(~3.3B)+ **flow-matching 动作专家**,连续动作 chunk | 开源(Apache-2.0,openpi + LeRobot) | LoRA 微调 **>22.5GB(RTX 4090 可行,JAX 路径)**;全量微调 >70GB(A100/H100);注意新 PyTorch 实现暂列 LoRA 为不支持 |
| **π0.5**(2025.04, [2504.16054](https://arxiv.org/abs/2504.16054)) | "Knowledge Insulation",web 数据/多机器人协同训练,开放世界泛化 | 开源 | 规模未核实(π0 同量级);openpi 支持 LoRA |
| **π0-FAST**(2025.01) | FAST 动作 token 化 + **自回归**动作头,训练快约 5 倍 | 开源 | arXiv 号未核实 |
| **OpenVLA**(2024.06, [2406.09246](https://arxiv.org/abs/2406.09246)) | Llama2-7B + DINOv2/SigLIP,**离散动作 token**,预训练 970k OXE episodes | 开源 | 7B;LoRA 仅调 ~1.4% 参数即匹配全量微调;官方 A100 级,4090 QLoRA 社区实践(未核实) |
| **OpenVLA-OFT**(2025.02, [2502.19645](https://arxiv.org/abs/2502.19645)) | 微调配方:连续动作 + 并行解码(L1 回归或 diffusion 头),LIBERO 76.5%→**97.1%** | 开源 | 官方 A100/H100;4090 未核实 |
| **RT-2**(2023.07, [2307.15818](https://arxiv.org/abs/2307.15818)) | PaLI-X 55B / PaLM-E 12B,web VLM co-fine-tune,语义涌现 | **闭源** | 不可微调 |
| **Octo**(2024.05, [2405.12213](https://arxiv.org/abs/2405.12213)) | ~93M transformer + **diffusion 头**,OXE 25 数据集 800k 轨迹 | 开源 | 单卡可微调(显存需求未核实) |
| **RDT-1B**(2024.10, [2410.07864](https://arxiv.org/abs/2410.07864)) | 1.2B **diffusion transformer** + T5-XXL 语言编码器,统一动作空间,预测 64 步 chunk | 开源 | 官方 FAQ 明确警告 **RTX 4090 显存可能不足**(建议预计算 T5 embedding、梯度检查点);后继 RDT2(基于 Qwen2.5-VL 7B)LoRA 需 ≥32GB |
| **GR00T N1/N1.5**(2025.03 [2503.14734](https://arxiv.org/abs/2503.14734) / 2025.05) | **双系统**:Eagle-2 VLM(System 2)+ DiT **flow-matching** 动作头(System 1) | 开源(HF `nvidia/GR00T-N1-2B`,Isaac-GR00T) | 2B;N1.5 用 RoboCasa 仿真数据增广,官方提供真机微调教程 |
| **CogACT**(2024.11, [2411.19650](https://arxiv.org/abs/2411.19650)) | Prismatic VLM + **diffusion action transformer**(系统验证扩散动作模块优于回归头) | 开源(GitHub microsoft/CogACT) | 规模分档未核实 |
| **SpatialVLA**(2025.01, [2501.15830](https://arxiv.org/abs/2501.15830)) | 自适应动作网格 + 空间语义动作表示,强化 3D 空间理解;7 个任务套件评测 | 开源 | 规模未核实 |
| **UniVLA**(两篇) | BAAI [2506.19850](https://arxiv.org/abs/2506.19850):自回归统一多模态;OpenDriveLab [2505.06111](https://arxiv.org/abs/2505.06111):从无动作标签视频学 latent action | 均开源 | — |
| **AgiBot GO-1**(2025.03,见 [2503.06669](https://arxiv.org/abs/2503.06669)) | **ViLLA**:VLM + Latent Planner + Action Expert(Mixture-of-Transformers) | 开源(随 AgiBot World) | — |
| **GR-3**(2025.07, [2507.15493](https://arxiv.org/abs/2507.15493)) | ~4B(AlphaXiv 口径),Qwen2.5-VL 初始化 + **flow-matching** 动作专家,双臂移动机器人;多源数据(真机+人类视频+合成) | **未开源** | — |
| **Gemini Robotics / 1.5**(2025.03 / 2025.10, [2510.03342](https://arxiv.org/abs/2510.03342)) | ER(具身推理 VLM)+ VLA 代理式架构,跨本体迁移 | **闭源**(ER 1.5 有付费 API) | 不可自持微调 |
| **Figure Helix**(2025.02, [figure.ai/news/helix](https://www.figure.ai/news/helix)) | 双系统:System 2 = 7B VLM(~7-9Hz)+ System 1 = 80M 快速策略(200Hz),机载运行 | **闭源** | — |

## 二、VLA 预训练数据集

- **Open X-Embodiment**([2310.08864](https://arxiv.org/abs/2310.08864)):1M+ 轨迹、22 种本体、21 机构(RT-X 训练用)。
- **DROID**([2403.12945](https://arxiv.org/abs/2403.12945)):76k-86k 轨迹/350h、564 场景 86 任务、Franka、13 机构 50 名操作员、多视角立体相机。
- **AgiBot World**([2503.06669](https://arxiv.org/abs/2503.06669),IROS 2025 最佳论文):1,003,672 条轨迹、~43.8TB、217 任务、5 大场景、100 台机器人共 2976h,动捕+主臂远程遥操采集;2026 年发布 AgiBot World 2026。
- **RoboMIND**([2412.13877](https://arxiv.org/abs/2412.13877)):V1 107k 轨迹/~305h、479 任务、96 类物体、4 种本体;2.0 已达 310k+ 轨迹/6 本体/739 任务。
- 共同形态:多视角 RGB + 本体状态 + 关节/末端动作 + 语言指令(RLDS/LeRobotDataset 格式)。注意:**均为桌面/结构化场景,无温室农业数据**。

## 三、微调数据需求(引用)

- **π0 论文 §5.3**:新任务分别用 1/5/50/100 条演示微调,**π0 用 50 条即超过 OpenVLA/ACT 用 100 条的效果**;性能随数据量提升,最难任务(叠衣)仍不达标。
- **openpi 官方 README**:未给硬性条数,明示"may or may not work";社区共识约 **50 条/任务起步**(复杂任务 100-200 条)。
- **ALOHA/ACT**([2304.13705](https://arxiv.org/abs/2304.13705)):50 条/任务达 80-90%+ 成功率;**Diffusion Policy 通常需 100-200 条**;Mobile ALOHA 50 条移动演示 + 静态数据 co-training,成功率最高提升 90%。
- OpenVLA 微调研究(OpenReview "Robust Fine-tuning of VLAs"):DROID 上每任务 50-100 条;"Self-Improving VLA"(OpenReview):~50 条为能力阈值,且要求基础策略成功率 >80% 才适合自提升。
- **失败/恢复数据的必要性**:HarvestFlex(2026)刻意保留"失败-恢复"片段;FLARE(CVPR 2026)、RACER、[Diagnose, Correct, and Learn from Manipulation Failures](https://arxiv.org/abs/2512.02787)、Rewind-IL(2604.16683)等 2025-2026 工作一致表明:只学成功演示的策略对异常状态脆弱,恢复演示显著提升鲁棒性。

## 四、无仿真数据采集路线

| 方案 | 成本 | 要点 |
|---|---|---|
| ALOHA / Mobile ALOHA(2023/2024,[2304.13705](https://arxiv.org/abs/2304.13705)、[2401.02117](https://arxiv.org/abs/2401.02117)) | $20k / $32k(BOM) | 主从臂(骑乘式全身遥操);50 条/任务即可训 ACT |
| **UMI**(2024.02,[2402.10329](https://arxiv.org/abs/2402.10329)) | 手持夹爪+GoPro,约 $371(第三方口径) | 免机器人 in-the-wild 采集,训 Diffusion Policy 直接迁移;RSS 2024 最佳系统论文入围;**腕部鱼眼视角与我们固定 L515 eye-to-hand 分布不匹配,剪切动作需定制** |
| **GELLO**([2309.13037](https://arxiv.org/abs/2309.13037)) | **BOM <$300/设备** | 3D 打印运动学同构主臂,直出关节空间,低延迟;Aubo i5 需自建运动学适配(社区支持情况未核实) |
| AirExo / AirExo-2([2309.14975](https://arxiv.org/abs/2309.14975),ICLR 2025) | ~$300/臂 | 外骨骼遥操,支持 in-the-wild 大规模采集 |
| Bunny-VisionPro([2407.03162](https://arxiv.org/abs/2407.03162))/ AnyTeleop | Vision Pro/VR | VR 视觉遥操开源;**HarvestFlex 用 Quest3(端到端延迟 <0.1s)** |
| AgiBot 遥操 | 基础设施重 | 动捕+主臂,支撑百台机器人百万轨迹 |
| **LeRobot 生态** | SO-101 套件 ~$110-280/臂(DIY ~$120),主从遥操台 $280-560;LeKiwi 底盘 $199-2000 | `lerobot record/calibrate/replay/train` CLI;LeRobotDataset 已成 HF 托管标准;内置 ACT、Diffusion Policy、π0/π0.5、SmolVLA、GR00T 微调 |

## 五、Sim-to-Real 真实案例(非设想)

- **MimicGen**([2310.17596](https://arxiv.org/abs/2310.17596)):<200 条人类演示 → 50k+ 仿真演示(18 任务,~50 倍放大);需要场景与机器人的仿真资产。
- **RoboCasa**([2406.02523](https://arxiv.org/abs/2406.02523)):120→2500+ 厨房场景、3200+ 物体,MimicGen 生成 ~1615h 数据。
- **GR00T N1.5**:NVIDIA 官方博客明确"人类真机少量演示 → 仿真/世界模型放大 → 微调真机部署,成功率显著提升";GR00T-Dreams 11 小时生成 780k 条合成轨迹(~6.5k 小时)。
- **DreamGen**([2505.12705](https://arxiv.org/abs/2505.12705),ICML 2025):视频世界模型生成 "neural trajectories" 用于微调 π0,在未见行为/环境上泛化提升(DreamGenBench)。
- **瓶颈**:需要高质量场景/资产重建;接触密集任务(剪切、滑动)物理保真度低;视觉域差。温室强遮挡 + 漫射光 + 细果梗剪切恰好最难仿真。

## 六、采摘类任务与农业的 VLA 工作

- **HarvestFlex**(2026.03,[2603.05982](https://arxiv.org/abs/2603.05982))——与我们最相关:**首个 VLA → 真实温室(桌面草莓)采摘的系统研究**。6DoF 臂 + 硅胶吸盘末端、2 台固定 D455 + 腕部 D405、LeRobot 框架、Quest3 遥操;3.71h/227 条演示微调 π0/π0.5/WALL-OSS(2×A800)。结果:π0.5 全量微调 **74% 成功率 vs 模块化管线 89%**,但周期 32.6s vs 8.3s;相机消融(3/2/1 视角)= 74%/42%/10%;异步推理优于同步;失败集中在严重遮挡、镜面反射、接触动力学失配。结论:**约 4 小时真机数据足以获得非平凡的采摘策略,但 VLA 当前不敌调优的模块化管线**。
- "VLA Models for Unmanned Systems in Agriculture"(Inkyu Sa 等,[2607.06706](https://arxiv.org/pdf/2607.06706),2026)——农业 VLA 综述(全名未逐字核实);另有 "Low-Cost VLA-RL 果园采摘框架"(ResearchGate 条目,细节未核实)。
- **番茄串采摘**:检索到的樱桃番茄串/采摘点检测(MDPI Horticulturae 2025)、双臂甜椒收获([2603.13987](https://arxiv.org/abs/2603.13987))、双臂苹果收获([2506.05714](https://arxiv.org/abs/2506.05714))仍是"感知+规划"路线;**目前未检索到已发表的番茄串端到端 VLA 工作——这是空白**。

## 七、对我们(无遥操设备、机器人改造中)的启示

**现阶段即可做:**
1. **数据基建先行**:采用 LeRobotDataset 格式 + `lerobot record`;先冻结动作空间设计(臂关节 + 底盘 + 剪切/夹持两路离散或连续量,考虑把剪切状态作为额外观测输入)与多机位协议。HarvestFlex 的相机消融(74%/42%/10%)说明固定双 L515 + 可选第三视角价值极大。
2. **遥操设备选型(无需整机就绪)**:首选 **GELLO 式 3D 打印同构主臂(<$300)**——只需 Aubo i5 运动学即可校准,机械臂到货前就能调试;或 **Quest3 视觉遥操**(HarvestFlex 同款,LeRobot 原生支持,<$1000)。两者都以真末端执行器 + 真相机采数,零 embodiment 差距。**不推荐 UMI**:腕部相机视角与固定 L515 分布不符,且剪切动作无法自然映射。
3. **模型与算力**:第一梯队 **π0/π0.5(LoRA 22.5GB 起,RTX 4090 可行)** 与 **OpenVLA-OFT**;快速迭代用 **SmolVLA(450M,[2506.01844](https://arxiv.org/abs/2506.01844))**;RDT-1B 在 4090 上偏紧。注意所有预训练权重均无农业数据,域差大,按 50-200 条/任务预算,并分层覆盖易/中/难遮挡。
4. **失败-恢复数据从第一天就录**(剪断失败、夹持脱落、果串回弹、手柄纠偏段),证据链充分。

**必须等机器人就绪:**
真机微调与闭环评测、RL/自提升(VLA-RL 类)、恢复策略部署、以及有意义的 sim-to-real——温室场景重建 + RoboCasa 式增广技术上可行,但剪切接触动力学仿真保真度低,建议先只对"接近段"做仿真增广,剪切段靠真机数据;仿真投入等真机基线建立后再评估。

**现实预期**:HarvestFlex 表明 VLA 目前成功率与速度低于调优的模块化管线(74% vs 89%,32.6s vs 8.3s),但开发成本低、重试行为更好、泛化到新品种/光照更省事。建议采用"模块化感知(成熟度/剪点检测)+ VLA 接近-剪切-放置"的混合架构,并双轨并行评估,把番茄串 VLA 空白转化为我们的发表机会。

## Sources
- [π0 (arXiv 2410.24164)](https://arxiv.org/abs/2410.24164) / [openpi](https://github.com/Physical-Intelligence/openpi) / [π0.5 (2504.16054)](https://arxiv.org/abs/2504.16054)
- [OpenVLA](https://openvla.github.io/) / [OpenVLA-OFT (2502.19645)](https://arxiv.org/abs/2502.19645) / [RT-2 (2307.15818)](https://arxiv.org/abs/2307.15818) / [Octo](https://octo-models.github.io/)
- [RDT-1B (2410.07864)](https://arxiv.org/abs/2410.07864) / [GitHub thu-ml](https://github.com/thu-ml/RoboticsDiffusionTransformer) / [GR00T N1 (2503.14734)](https://arxiv.org/abs/2503.14734) / [NVIDIA 合成数据博客](https://developer.nvidia.com/blog/enhance-robot-learning-with-synthetic-trajectory-data-generated-by-world-foundation-models/)
- [CogACT (2411.19650)](https://arxiv.org/abs/2411.19650) / [SpatialVLA (2501.15830)](https://arxiv.org/abs/2501.15830) / [UniVLA (2505.06111)](https://arxiv.org/abs/2505.06111) / [GR-3 (2507.15493)](https://arxiv.org/abs/2507.15493) / [Gemini Robotics 1.5 (2510.03342)](https://arxiv.org/abs/2510.03342) / [Figure Helix](https://www.figure.ai/news/helix)
- [AgiBot World (2503.06669)](https://arxiv.org/abs/2503.06669) / [RoboMIND (2412.13877)](https://arxiv.org/abs/2412.13877) / [DROID](https://droid-dataset.github.io/) / [OXE](https://robotics-transformer-x.github.io/)
- [HarvestFlex (2603.05982)](https://arxiv.org/abs/2603.05982) / [DreamGen (2505.12705)](https://arxiv.org/abs/2505.12705) / [MimicGen](https://mimicgen.github.io/) / [RoboCasa](https://robocasa.ai/)
- [ALOHA (2304.13705)](https://arxiv.org/abs/2304.13705) / [Mobile ALOHA](https://mobile-aloha.github.io/) / [UMI (2402.10329)](https://arxiv.org/abs/2402.10329) / [GELLO (2309.13037)](https://arxiv.org/abs/2309.13037) / [AirExo](https://airexo.github.io/) / [Bunny-VisionPro (2407.03162)](https://arxiv.org/abs/2407.03162)
- [LeRobot](https://github.com/huggingface/lerobot) / [SO-101 (Seeed)](https://www.seeedstudio.com/SO-ARM101-Low-Cost-AI-Arm-Kit-p-6426.html) / [LeKiwi](https://github.com/SIGRobotics-UIUC/LeKiwi) / [SmolVLA (2506.01844)](https://arxiv.org/abs/2506.01844)
