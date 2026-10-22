# ReadingNotes

个人论文精读笔记库。每篇论文一个 Markdown 文件,固定结构:

- **Input / Output** 一行:模型/系统的输入是什么、输出是什么,写在 metadata 块里
- 中文精读:解决了什么问题 / 新意在哪里 / idea 具体落在哪里 / 最大难点在哪里 / 局限与相关性 / 是否能连 GitHub 复现
  - 关键概念第一次出现时,中文后面用英文括号注释(方便之后用英文讲解/组会汇报)
  - 关键原文摘录(中英对照):5 句左右最关键的原文逐字引用 + 中文翻译 + 批注,并尽量把论点和论文里的具体表格/公式数字对应起来
- 英文一目了然总结:一张大尺寸结构化 SVG 海报(问题/新意/架构图/难点/证据/代码可用性)+ 图解读(逐框讲解)+ 中英对照表格
- **2026-08-22 起新增**(仅用于此后新写的论文,未回填旧笔记):开头一段 TL;DR(问题→Gap→Novelty→核心洞察→方法/论证→结果→意义)、按重要性排序并逐条配证据的贡献列表、明确的 Novelty 分类判断(新问题/新insight/新方法/新理论/新数据集/新场景,且要说清是首创还是改进组合)、"可略过背景"清单、更严格的假设/局限/质疑点、"一个月后记住5件事"

## Index

| # | Paper | Venue / Year | Notes | Code/GitHub |
|---|-------|--------------|-------|--------------|
| 1 | DriveVLM | CoRL 2024 | [notes/drivevlm.md](notes/drivevlm.md) | Not released (project page only) |
| 2 | DriveMLM | Tech report 2023 (v3 2025) | [notes/drivemlm.md](notes/drivemlm.md) | Repo exists, placeholder only |
| 3 | LoRA | ICLR 2022 | [notes/lora.md](notes/lora.md) | Released & usable (microsoft/LoRA) |
| 4 | Adaptive Mixtures of Local Experts | Neural Computation 1991 | [notes/mixture-of-local-experts.md](notes/mixture-of-local-experts.md) | N/A (pre-GitHub) |
| 5 | Alpamayo-R1 | arXiv 2025 (v2 2026), NVIDIA | [notes/alpamayo-r1.md](notes/alpamayo-r1.md) | Weights + inference code released |
| 6 | An Open Approach to Autonomous Vehicles (Autoware) | IEEE Micro 2015 | [notes/autoware.md](notes/autoware.md) | Open & maintained 10+ years |
| 7 | Constructing a Digital Twin of a Physical Testbed (UB CAVAS) | IEEE IV Workshops 2026 | [notes/ub-digital-twin.md](notes/ub-digital-twin.md) | Released, verified usable (ub-cavas/UB-DigitalTwin) |
| 8 | RoboLab | RSS 2026, NVIDIA | [notes/robolab.md](notes/robolab.md) | Released, mature (NVlabs/RoboLab, 452★) |
| 9 | End-to-end Autonomous Driving: Challenges and Frontiers (survey) | TPAMI 2024, OpenDriveLab | [notes/end-to-end-ad-survey.md](notes/end-to-end-ad-survey.md) | Curated lit. repo, not code (3.7k★) |
| 10 | WA-JEPA | arXiv 2026 | [notes/wa-jepa.md](notes/wa-jepa.md) | Released, weights included (39★) |
| 11 | Qwen-Drive-1.0 | arXiv 2026, Qwen Team + HUST | [notes/qwen-drive.md](notes/qwen-drive.md) | Released & runnable (QwenLM/Qwen-Drive-1.0) |
| 12 | DriveZero | arXiv 2026, Xiaomi EV | [notes/drivezero.md](notes/drivezero.md) | Not released (tech report) |
| 13 | Drive-JEPA | arXiv 2026 (2601.22032) | [notes/drive-jepa.md](notes/drive-jepa.md) | Pending release (V-JEPA base: facebookresearch/jepa) |
| 14 | SparseDriveV2 | arXiv 2026 (2603.29163), ECCV 2026 | [notes/sparsedrivev2.md](notes/sparsedrivev2.md) | Released with weights (swc-17/SparseDriveV2) |
| 15 | NAVSIM | NeurIPS 2024 (2406.15349) | [notes/navsim.md](notes/navsim.md) | Released, actively maintained (autonomousvision/navsim, 1.1k★) — source of PDMS/EPDMS used across most of the papers above |
| 16 | ARTEMIS | arXiv 2025 (2504.19580) | [notes/artemis.md](notes/artemis.md) | Genuine MoE (routed, input-dependent) |
| 17 | WAM-Diff | arXiv 2026 (2512.11872) | [notes/wam-diff.md](notes/wam-diff.md) | Genuine MoE (routed, input-dependent), single-image only |
| 18 | CoWorld-VLA | arXiv 2026 (2605.10426), Afari Intelligent Drive | [notes/coworld-vla.md](notes/coworld-vla.md) | **NOT true MoE** — 4 always-active branches fused via fixed global scalar weights, no gating/routing; released & runnable (AFARI-Research/CoWorld-VLA, 10★) |
| 19 | EMMA | arXiv 2410.23262, Waymo | [notes/emma.md](notes/emma.md) | Not released (no code/weights found) |
| 20 | Senna | arXiv 2410.22313, HUST + Horizon Robotics | [notes/senna.md](notes/senna.md) | Released & runnable (hustvl/Senna) |
| 21 | OmniDreams | arXiv 2606.03159, NVIDIA | [notes/omnidreams.md](notes/omnidreams.md) | Released, verified real (nv-tlabs/omni-dreams, 344★, CC BY 4.0) |
| 22 | GAIA-1 | arXiv 2309.17080, 2023, Wayve(UK) | [notes/gaia-1.md](notes/gaia-1.md) | Not released (tech report only) — **note: Wayve, not Waymo** |
| 23 | GAIA-2 | arXiv 2503.20523, 2025, Wayve(UK) | [notes/gaia-2.md](notes/gaia-2.md) | Not released (tech report only) |
| 24 | SAMoE-VLA | arXiv 2603.08113, 2026, Tsinghua AIR | [notes/samoe-vla.md](notes/samoe-vla.md) | Not released ("code will be released soon") — scene-level MoE, explicitly critiques DriveMoE's token-level design |
| 25 | PerceptDrive | arXiv 2607.20175, 2026, Tsinghua SIGS (under review) | [notes/perceptdrive.md](notes/perceptdrive.md) | Not found — **阅读中**(已覆盖Related Work+两阶段训练结构+消融实验，架构图/Router细节待补) — frozen priors + per-branch retention anchor + scene-conditioned router |
| 26 | ExpertAD | arXiv 2511.11740, 2025, **AAAI 2026已接收** | [notes/expertad.md](notes/expertad.md) | Not found — **阅读中，笔记仅基于摘要+对比表** — Perception Adapter + Top-4稀疏专家，8个功能专家(地图/跟踪/速度/朝向/导航) |
| 27 | Drive-KD | arXiv 2601.21288, 2026 (preprint) | [notes/drive-kd.md](notes/drive-kd.md) | Not found — **阅读中，笔记仅基于摘要+对比表** — 多教师蒸馏(非MoE)，感知-推理-规划三元组+非对称梯度投影 |
| 28 | DriveMoE | arXiv 2505.16278, 2025 (CVPR 2026) | [notes/drivemoe.md](notes/drivemoe.md) | Not verified — scene-select Vision MoE + skill-specialized Action MoE; the paper SAMoE-VLA critiques by name |
| 29 | Cluster-Aware Upcycling for MoE | CVPR 2026, LG AI Research | [notes/cluster-aware-upcycling.md](notes/cluster-aware-upcycling.md) | Not verified — semantic-cluster SVD init for sparse upcycling, not driving-specific |
| 30 | ERMoE | arXiv 2511.10971, 2025 (CVPR 2026) | [notes/ermoe.md](notes/ermoe.md) | Not verified — eigenbasis routing instead of learned gating, not driving-specific |
| 31 | Life-IQA | arXiv 2511.19024, 2025 (CVPR 2026) | [notes/life-iqa.md](notes/life-iqa.md) | Not verified — MoE feature decoupling for blind image quality, not driving-specific |
| 32 | Local Precise Refinement (Dual-Gated MoE) | arXiv 2603.13352, 2026 (CVPR 2026) | [notes/local-precise-refinement.md](notes/local-precise-refinement.md) | Not verified — token-level dual-gated MoE against spectral shift, not driving-specific |
| 33 | M4-SAM | arXiv 2605.11760, 2026 (CVPR 2026) | [notes/m4-sam.md](notes/m4-sam.md) | Not verified — MoE-LoRA into SAM2 for RGB-D video salient detection, not driving-specific |
| 34 | MoE-FD (Open Vocab Scene Graph) | CVPR 2026 | [notes/moe-fd-scene-graph.md](notes/moe-fd-scene-graph.md) | Not verified — MoE feature decoupling for scene graph generation, not driving-specific |
| 35 | MoDES | arXiv 2511.15690, 2025 (CVPR 2026) | [notes/modes.md](notes/modes.md) | Not verified — training-free dynamic expert skipping; directly relevant to the "SA-MoE doesn't save compute" gap |
| 36 | MoE-GRPO | arXiv 2603.24984, 2026 (CVPR 2026) | [notes/moe-grpo.md](notes/moe-grpo.md) | Not verified — RL (GRPO)-trained expert routing, alternative router-training signal vs. PerceptDrive's metric distillation |
| 37 | MoEActok | CVPR 2026 | [notes/moeactok.md](notes/moeactok.md) | Not verified — MoE VQ-VAE action tokenizer, experts split by skill not scene |
| 38 | MoRE (3D Geometry) | arXiv 2510.27234, 2025 (CVPR 2026) | [notes/more-3d-geometry.md](notes/more-3d-geometry.md) | Not verified — MoE-routed 3D geometry foundation model, parallels PerceptDrive's DSE |
| 39 | NESTOR | arXiv 2602.22059, 2026 (CVPR 2026) | [notes/nestor.md](notes/nestor.md) | Not verified — nested image-level + token-level MoE for PDE pretraining; independent echo of the scene/token granularity argument |
| 40 | On Token's Dilemma (LLaVA-DyMoE) | arXiv 2603.27481, 2026 (CVPR 2026) | [notes/on-tokens-dilemma.md](notes/on-tokens-dilemma.md) | Not verified — drift-aware token assignment against continual-learning routing drift |
| 41 | PromptMoE | arXiv 2603.12538, 2026 (CVPR 2026) | [notes/promptmoe.md](notes/promptmoe.md) | Not verified — MoE-driven segmentation prompt refinement, not driving-specific |
| 42 | Quant Experts | arXiv 2602.24059, 2026 (CVPR 2026) | [notes/quant-experts.md](notes/quant-experts.md) | Not verified — shared + routed LoRA experts for VLM quantization error, not driving-specific |
| 43 | SegMoTE | arXiv 2602.19213, 2026 (CVPR 2026 Oral) | [notes/segmote.md](notes/segmote.md) | Not verified — frozen SAM encoder + lightweight trainable token-level MoE; same "freeze + lightweight router" pattern as PerceptDrive, different domain |
| 44 | SMoES | arXiv 2604.23996, 2026 (CVPR 2026) | [notes/smoes.md](notes/smoes.md) | Not verified — soft modality-guided expert specialization, from Li Auto |
| 45 | Spectral MoE for Continual Learning | CVPR 2026 | [notes/spectral-moe-continual.md](notes/spectral-moe-continual.md) | Released (github.com/ouycc/Spectral_MoE) — spectral experts + dual-router against routing drift, alternative to on-tokens-dilemma's fix |
| 46 | TAG-MoE | arXiv 2601.08881, 2026 (CVPR 2026) | [notes/tag-moe.md](notes/tag-moe.md) | Not verified — task-semantic-conditioned gating for unified generation, independent echo of scene-conditioned routing in a different domain |
| 47 | Teacher-Guided Routing for Sparse Vision MoE | arXiv 2604.21330, 2026 (CVPR 2026) | [notes/tgr-moe.md](notes/tgr-moe.md) | Not verified — dense-teacher-supervised router training, distinct from Drive-KD (which has no router at all) |
| 48 | UniMMAD | arXiv 2509.25934, 2025 (CVPR 2026) | [notes/unimmad.md](notes/unimmad.md) | Not verified — MoE-in-MoE cuts MoE params ~75%; directly relevant to the "SA-MoE doesn't save compute" gap |
| 49 | VidPrism | arXiv 2605.28229, 2026 (CVPR 2026) | [notes/vidprism.md](notes/vidprism.md) | Not verified — names and addresses "expert homogenization" as a named MoE failure mode |
| 50 | YOLO-Master | arXiv 2512.23273, 2025 (CVPR 2026) | [notes/yolo-master.md](notes/yolo-master.md) | Not verified — instance-conditional adaptive compute via MoE, real-time detection |

## Synthesis(跨论文综合分析)

单篇论文精读之外,按需写的跨论文综合分析,一般是"按某篇综述/某个框架的结构,看后续论文怎么回应"这种写法。

- [representation-synthesis-2026.md](representation-synthesis-2026.md) — 按 End-to-end AD Survey §4.2(表征设计/表征学习)的框架,把 WA-JEPA / Drive-JEPA / Qwen-Drive-1.0 / DriveZero / SparseDriveV2 分别映射进去,指出 §4.2(表征学习)和 §4.3(世界模型)的边界正在消失,并提出 5 个可推进的具体方向
- [sensor-fusion-qa.md](sensor-fusion-qa.md) — 由 EMMA(Waymo)附录局限性段落引出的跨论文 Q&A:摄像头/LiDAR/雷达各自优劣、"融合"的真正目的(不是数据标准化)、为什么 VLM 驱动的模型普遍选纯视觉、传统融合 vs 接入大模型的融合成熟度差异、LiDAR 点云数据格式详解
- [moe-landscape-synthesis-2026-10.md](moe-landscape-synthesis-2026-10.md) — CVPR 2026 MoE全景图谱:驾驶专用MoE(DriveMoE/SAMoE-VLA/PerceptDrive/ExpertAD/Drive-KD等)+23篇通用MoE技术论文,按"路由颗粒度/路由训练信号/省计算量/持续学习防漂移/专家从哪来/路由失效模式"分组,并标出每个已识别的研究空子对应哪些可参考技术

## 结构

```text
notes/          每篇论文的精读笔记 (.md)
notes/img/      对应的英文总结海报 (.svg)
glossary.md     跨论文术语表,按首次出现的论文标注
*-synthesis-*.md  跨论文综合分析(见上)
```
