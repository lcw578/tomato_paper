# 论文库索引(docs/papers)

> 收录调研体系中精选的论文 PDF,与《调研原始报告/05_核心论文逐篇精读.md》配套。
> 精读编号(T1-xx / T2-xx / T3)对应精读文档中的条目;"原始报告未收录"标记的论文是自行收集、调研报告尚未编目的,建议下次更新时补充精读。
> 命名规则:新下载文件 = `短名_arXiv编号.pdf`;原有文件保留原始文件名。

## 目录结构

```
papers/
├── 00_基准与直接先例/    — 基准论文 + 温室/农业采摘 VLA 先例
├── 01_VLA模型与训练/     — VLA 基座模型与微调配方
├── 02_遥操与数据采集/    — 遥操硬件路线 + 示教数据生成/公开数据集
├── 03_Real2Sim_3DGS与世界模型/ — 3DGS 仿真、物理+高斯、重建、世界模型
├── 04_仿真平台与农业仿真/      — 果园/温室仿真基准与平台
└── 05_果梗力学与物性/    — 断裂/损伤阈值标定文献(T3)
```

## 00 基准与直接先例(4 篇)

| 文件 | 论文 | 来源 | 精读 | 一句话 |
|---|---|---|---|---|
| Journal of Field Robotics - 2026 - Xu - Design Development and Field Testing of a Tomato Bunch Harvesting Robot.pdf | 番茄串采摘机器人设计/开发/田间测试(**基准论文**) | DOI 10.1002/rob.70250 | — | 我们的系统与失败分布来源(70.77%,12.23 s/串) |
| HarvestFlex: Strawberry Harvesting via Vision-Language-Action Policy Adaptation in the wild.pdf | HarvestFlex | arXiv [2603.05982](https://arxiv.org/abs/2603.05982) | T1-01 | 温室 VLA 唯一实测先例(74% vs 模块化 89%) |
| Field Validation of Vision-Guided Action Policy Generation for RoboticTomato HarvestingField.pdf | Field Validation of Vision-Guided Action Policy Generation for Robotic Tomato Harvesting(中国农业大学等,Information Processing in Agriculture) | **原始报告未收录**,已核读原文 | 已核读 | 混合视觉伺服 + 改进 **ACT 模仿学习(非 VLA,无语言条件)**;**无仿真环境**,纯真机 440 条示教;果梗对齐成功率 91.0%(无偏差)/66.7–70.7%(中等)/56.7–61.8%(严重),开环对照在轻微偏差下 0%。不改变"番茄串 VLA 空白"结论;独立印证"开环是缺陷、闭环有价值",建议补入调研相关工作 |
| From Simulation to the Real-World: An In-Field 6D Pose Dataset and Baseline for Robotic Strawberry Harvesting.pdf | 草莓采摘田间 6D 位姿数据集与基线 | **原始报告未收录** | 待精读 | 草莓 sim2real 6D 位姿数据集,采集协议可参考 |

## 01 VLA 模型与训练(8 篇)

| 文件 | 论文 | arXiv | 精读 | 一句话 |
|---|---|---|---|---|
| pi0_2410.24164.pdf | π0(Physical Intelligence) | [2410.24164](https://arxiv.org/abs/2410.24164) | T1-02 | PaliGemma+flow-matching 动作专家,首选基座 |
| pi0.5_2504.16054.pdf | π0.5 | [2504.16054](https://arxiv.org/abs/2504.16054) | T1-02 | Knowledge Insulation,开放世界泛化;HarvestFlex 最佳配置 |
| OpenVLA_2406.09246.pdf | OpenVLA | [2406.09246](https://arxiv.org/abs/2406.09246) | T2-11 | 7B 离散动作 token 基线,LoRA 1.4% 参数即匹配全量 |
| OpenVLA-OFT_2502.19645.pdf | OpenVLA-OFT | [2502.19645](https://arxiv.org/abs/2502.19645) | T2-11 | 最成熟微调配方(连续动作+并行解码),LIBERO 97.1% |
| GR00T-N1_2503.14734.pdf | GR00T N1/N1.5(NVIDIA) | [2503.14734](https://arxiv.org/abs/2503.14734) | T2-12 | 双系统 2B,官方"少量示教+仿真增广"教程 |
| SmolVLA_2506.01844.pdf | SmolVLA | [2506.01844](https://arxiv.org/abs/2506.01844) | T2-13 | 450M 快速迭代/消融专用 |
| RDT-1B_2410.07864.pdf | RDT-1B | [2410.07864](https://arxiv.org/abs/2410.07864) | T2-14 | 1.2B diffusion,双臂强;4090 偏紧 |
| Visual-Language-Guided Task Planning for Horticul robots.pdf | 园艺机器人 VLM 任务规划 | [2601.11906](https://arxiv.org/abs/2601.11906) | 报告 04 引用 | 农业 VLA 迹象;待补精读条目 |

## 02 遥操与数据采集(6 篇)

| 文件 | 论文 | arXiv | 精读 | 一句话 |
|---|---|---|---|---|
| ALOHA-ACT_2304.13705.pdf | ALOHA/ACT | [2304.13705](https://arxiv.org/abs/2304.13705) | 报告 01 引用 | "50 条/任务"数据预算学说的来源 |
| MimicGen_2310.17596.pdf | MimicGen | [2310.17596](https://arxiv.org/abs/2310.17596) | T2-15 | <200 条示教 → 50k+ 仿真演示 |
| RoboCasa_2406.02523.pdf | RoboCasa | [2406.02523](https://arxiv.org/abs/2406.02523) | T2-16 | 程序化场景库架构参考 |
| UMI_2402.10329.pdf | UMI | [2402.10329](https://arxiv.org/abs/2402.10329) | T2-27 | 反面教材:腕部鱼眼与固定 L515 分布不匹配 |
| GELLO_2309.13037.pdf | GELLO | [2309.13037](https://arxiv.org/abs/2309.13037) | T1-09 | <$300 同构主臂,臂未到货即可开工 |
| RoboHarvest_2411.09929.pdf | RoboHarvest(CMU) | [2411.09929](https://arxiv.org/abs/2411.09929) | T2-26 | 300 条甜椒采摘示教公开(含剪切动作) |

## 03 Real2Sim / 3DGS 与世界模型(13 篇)

| 文件 | 论文 | arXiv | 精读 | 一句话 |
|---|---|---|---|---|
| VGGT_2503.11651.pdf | VGGT(Meta,CVPR 2025 Best Paper) | [2503.11651](https://arxiv.org/abs/2503.11651) | T1-07 | 前馈重建,专治温室重复纹理;重建第一站 |
| SplatSim_2409.10161.pdf | SplatSim | [2409.10161](https://arxiv.org/abs/2409.10161) | T1-05 | 3DGS 渲染纯仿真训练,零样本 86.25% |
| GSWorld_2510.20813.pdf | GSWorld(ICRA 2026) | [2510.20813](https://arxiv.org/abs/2510.20813) | T1-04 | GSDF:高斯×URDF×物理引擎标准格式,闭环评测 |
| PhysTwin_2503.17973.pdf | PhysTwin(ICCV 2025) | [2503.17973](https://arxiv.org/abs/2503.17973) | T1-06 | 交互视频标定弹簧质点物理,果梗柔性框架 |
| GreenhouseSplat_2510.01848.pdf | GreenhouseSplat | [2510.01848](https://arxiv.org/abs/2510.01848) | T1-08 | 廉价 RGB → 82 株黄瓜温室照片级仿真 |
| SimplerEnv_2405.05941.pdf | SimplerEnv(CoRL 2024) | [2405.05941](https://arxiv.org/abs/2405.05941) | T1-10 | real2sim 评测协议方法论(视觉匹配+系统辨识) |
| RoboSplat_2504.13175.pdf | RoboSplat(RSS 2025) | [2504.13175](https://arxiv.org/abs/2504.13175) | T2-17 | 高斯编辑演示增广,one-shot 87.8% |
| GaussianGrouping_2312.00732.pdf | Gaussian Grouping(ECCV 2024) | [2312.00732](https://arxiv.org/abs/2312.00732) | T2-18 | 3DGS 实例化,果串/茎叶分离路径 |
| PhysGaussian_2311.12198.pdf | PhysGaussian(CVPR 2024) | [2311.12198](https://arxiv.org/abs/2311.12198) | T2-19 | MPM 驱动高斯,谱系起点 |
| PhysDreamer_2404.13026.pdf | PhysDreamer | [2404.13026](https://arxiv.org/abs/2404.13026) | T2-19 | 视频扩散先验标定刚度 |
| Spring-Gaus_2403.09434.pdf | Spring-Gaus(ECCV 2024) | [2403.09434](https://arxiv.org/abs/2403.09434) | T2-19 | 弹簧质点嵌入 3DGS,实时 |
| DreamGen_2505.12705.pdf | DreamGen(ICML 2025) | [2505.12705](https://arxiv.org/abs/2505.12705) | T2-20 | 世界模型 neural trajectories 训 VLA |
| Cosmos-Transfer1_2503.14492.pdf | Cosmos-Transfer1(NVIDIA) | [2503.14492](https://arxiv.org/abs/2503.14492) | T2-21 | 仿真视频→照片级视频转写增广 |

## 04 仿真平台与农业仿真(5 篇)

| 文件 | 论文 | 来源 | 精读 | 一句话 |
|---|---|---|---|---|
| OrchardBench_2607.06337.pdf | OrchardBench | arXiv [2607.06337](https://arxiv.org/abs/2607.06337) | T1-03 | 唯一开源"操作级"果实脱离物理(Newton 引擎) |
| Agri-Sim: Agricultural Simulation Platform for Embodied Intelligence Evaluation in Greenhouse Robotics.pdf | Agri-Sim | arXiv [2608.29100](https://arxiv.org/abs/2608.29100) | T2-24 | 温室双臂采摘完整管线先例(未开源) |
| Digital twin-driven system for efficient tomato harvesting in greenhouses.pdf | 温室番茄采摘数字孪生系统 | **原始报告未收录** | **待精读** | 番茄温室数字孪生,与 WUR 线相关 |
| GazeboPlants-Cosserat_2402.02570.pdf | Gazebo Plants(ICRA 2024 WS) | arXiv [2402.02570](https://arxiv.org/abs/2402.02570) | T2-25 | Cosserat 杆植物形变交互插件 |
| Helios-程序化植株框架_2512.17966.pdf | Helios 通用程序化植株框架(UC Davis) | arXiv [2512.17966](https://arxiv.org/abs/2512.17966) | T2-22 | 程序化植株生成(含番茄),感知合成数据主力 |

## 05 果梗力学与物性(2 篇,标定引用)

| 文件 | 论文 | 关键参数 | 精读 |
|---|---|---|---|
| 番茄茎剪切力_InternationalAgrophysics.pdf | International Agrophysics(果梗剪切) | 果梗-果梗 19.91 N / 果梗-果荚 17.41 N / 果梗-幼苗藤蔓 8.62 N | T3 |
| 草莓果梗切断力_2207.12552.pdf | 草莓果梗切断(arXiv) | 约 15 N(跨作物对照) | T3 |

## 缺失项(反爬/付费墙,需浏览器手动下载)

均为开放获取或机构可获取,链接见精读文档第三节:

- [ ] Weng et al., Agronomy 2024(果梗拉伸 ~7 N):https://www.mdpi.com/2073-4395/14/10/2274 — MDPI 反爬,脚本无法直下
- [ ] 樱桃番茄拉摘力 0.58–2.46 N:Agronomy 2021,https://www.mdpi.com/2077-0472/11/9/815 — 同上
- [ ] aoc_tomato_farm(TAROS 2024):DOI 10.1007/978-3-031-72059-8_2 — Springer 付费墙
- [ ] Helios 应用论文:DOI 10.34133/plantphenomics.0189 — Plant Phenomics 开放获取
- [ ] 番茄茎弹性模量 0.82–2.41 GPa:ResearchGate(反爬,手动下)
- [ ] 末端夹持/切断力论文:Academia.edu(需登录)
- [ ] 果实压伤阈值 ~10 N:CABI(付费墙)

## 使用约定

1. **仅限组内研究使用**:arXiv PDF 按其默认许可分发使用;付费出版方论文(Xu JFR 2026 等)不得对外分发。
2. 正式引用任何数字前,对照《05_核心论文逐篇精读.md》开头的"复核修正"一节。
3. 新论文入库:先查精读文档是否有条目 → 下载归入对应类别 → 在上表补一行 → 缺精读条目的标"待精读"。
