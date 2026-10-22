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
| 26 | ExpertAD | arXiv 2511.11740, 2025, **AAAI 2026已接收** | [notes/expertad.md](notes/expertad.md) | Not found — **精读完成，⚠已修正**：MoE只在Prediction模块(8专家Top-4路由)，Perception段是特征重加权非MoE，Planning段完全复用基线——abstract"spans perception and planning"的表述有误导性，不构成全程MoE证据 |
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
| 51 | EVLF | arXiv 2603.07476, 2026 (CVPR 2026) | [notes/evlf.md](notes/evlf.md) | Released (github.com/wenqi-cai297/earlyfusion-for-dd) — early vision-language fusion for dataset distillation, not driving-specific |
| 52 | MDM (Vision-Language Dataset Distillation) | arXiv 2605.23482, 2026 (CVPR 2026) | [notes/mdm-vl-dataset-distill.md](notes/mdm-vl-dataset-distill.md) | Not verified — weight-space interpolation of fine-tuned models forms a mixed teacher, parallels SAMoE-VLA's parameter merge |
| 53 | DisCa | arXiv 2602.05449, 2026 (CVPR 2026) | [notes/disca.md](notes/disca.md) | Not verified — learnable feature caching for video diffusion, relevant to real-time driving world models |
| 54 | Reward Forcing | arXiv 2512.04678, 2025 (CVPR 2026) | [notes/reward-forcing.md](notes/reward-forcing.md) | Not verified — EMA-Sink streaming video generation; alternative to H_BEV staleness fix (continuous EMA update vs. one-shot or trigger-based refresh) |
| 55 | Transition Matching Distillation | arXiv 2601.09881, 2026 (CVPR 2026, NVIDIA/NYU) | [notes/transition-matching-distill.md](notes/transition-matching-distill.md) | Not verified — video diffusion distilled via few-step transition matching |
| 56 | Dual-Teacher Distillation (Earth Observation) | arXiv 2602.19863, 2026 (CVPR 2026) | [notes/dual-teacher-earth-obs.md](notes/dual-teacher-earth-obs.md) | Not verified — multi-teacher distillation, not driving-specific |
| 57 | Continual Distillation of Teachers from Different Domains | CVPR 2026 | [notes/continual-distill-teachers.md](notes/continual-distill-teachers.md) | Released (github.com/Nicolas1203/continual_distillation) — UKT/UKF framework, parallels MoE routing-drift problem in distillation terms |
| 58 | SigLino | arXiv 2512.20157, 2025 (CVPR 2026) | [notes/siglino.md](notes/siglino.md) | Not verified — **精读完成**，multi-teacher distillation into MoE students + distilled experts reused to initialize a downstream grounding VLM — closest overall mechanism match to the zhuzixin idea found so far |
| 59 | 4D-RGPT | arXiv 2512.17012, 2025 (CVPR 2026, NVIDIA) | [notes/4d-rgpt.md](notes/4d-rgpt.md) | Not verified — frozen-expert distillation into an MLLM for 4D understanding, same pattern as PerceptDrive's Perception Provider |
| 60 | Adaptive Video Distillation | arXiv 2603.21864, 2026 (CVPR 2026) | [notes/adaptive-video-distill.md](notes/adaptive-video-distill.md) | Not verified — names oversaturation/temporal collapse as video-distillation failure modes |
| 61 | AMap | arXiv 2512.19150, 2025 (CVPR 2026) | [notes/amap.md](notes/amap.md) | Not verified — **driving**, "distill-from-future" paradigm, alternative to PerceptDrive's self-predicted future conditioning for the H_BEV staleness gap |
| 62 | Deformable Gaussian Occupancy (DeGO) | arXiv 2605.28587, 2026 (CVPR 2026) | [notes/deformable-gaussian-occupancy.md](notes/deformable-gaussian-occupancy.md) | Not verified — **driving**, deformable-motion occupancy distillation, parallels PerceptDrive's DSE |
| 63 | Foundry (3D Foundation Models for Edge) | arXiv 2511.20721, 2025 (CVPR 2026) | [notes/foundry-3d-edge.md](notes/foundry-3d-edge.md) | Not verified — edge-deployment distillation, directly relevant to the "SA-MoE doesn't save compute" gap |
| 64 | Progressive Mask Distillation | CVPR 2026 | [notes/progressive-mask-distill.md](notes/progressive-mask-distill.md) | Not verified — curriculum-style progressive distillation for video representation |
| 65 | SemLT3D | arXiv 2604.18476, 2026 (CVPR 2026) | [notes/semlt3d.md](notes/semlt3d.md) | Not verified — **driving**, language-guided MoE routing by semantic affinity for long-tail safety-critical classes |
| 66 | WPT (World-to-Policy Transfer) | arXiv 2511.20095, 2025 (CVPR 2026) | [notes/wpt-world-policy.md](notes/wpt-world-policy.md) | Not verified — **driving**, online world-model distillation into a lightweight policy, 4.9x faster inference; most directly relevant to the "SA-MoE doesn't save compute" gap |
| 67 | Beyond Weak Supervision (Camouflaged Object Detection) | CVPR 2026 | [notes/beyond-weak-supervision-cod.md](notes/beyond-weak-supervision-cod.md) | Not verified — MLLM-guided graded distillation, not driving-specific |
| 68 | GKD (Semantic Segmentation) | arXiv 2603.02554, 2026 (CVPR 2026) | [notes/gkd-semantic-seg.md](notes/gkd-semantic-seg.md) | Not verified — two-stage freeze-then-adapt distillation, same pattern as PerceptDrive's Stage 1/Stage 2 split |
| 69 | LOREAL | CVPR 2026 | [notes/loreal.md](notes/loreal.md) | Not verified — dual-student resolution-invariant prompt self-distillation, not driving-specific |
| 70 | Masters (VLM Distillation) | arXiv 2512.22238, 2025 (CVPR 2026, NVIDIA) | [notes/masters-vlm-distill.md](notes/masters-vlm-distill.md) | Not verified — mask-progressive RL distillation for edge VLM deployment |
| 71 | Prompt-Anchored Vision-Text Distillation (Lifelong ReID) | CVPR 2026 | [notes/prompt-anchored-reid.md](notes/prompt-anchored-reid.md) | Released (github.com/zu-zi/PAD) — frozen text-anchor + new-slot/freeze-old-slot against catastrophic forgetting, third alternative to the routing-drift gap |
| 72 | SelecTKD | arXiv 2510.24021, 2025 (CVPR 2026) | [notes/selectkd.md](notes/selectkd.md) | Not verified — not-all-tokens-equal distillation weighting, echoes SAMoE-VLA's scene-vs-token granularity argument |
| 73 | SATtxt (Satellite Imagery) | arXiv 2602.22613, 2026 (CVPR 2026) | [notes/sattxt-satellite.md](notes/sattxt-satellite.md) | Not verified — spectral-to-RGB distillation, not driving-specific |
| 74 | Beta-KD (MLLM) | arXiv 2603.21426, 2026 (CVPR 2026) | [notes/beta-kd-mllm.md](notes/beta-kd-mllm.md) | Released (github.com/Jingchensun/beta-kd) — uncertainty-aware teacher-reliance weighting, useful if distilling multiple unequally-reliable priors |
| 75 | VL-Eraser | CVPR 2026 | [notes/vl-eraser.md](notes/vl-eraser.md) | Not verified — vacuum distillation + LoRA deletion for VLM unlearning, not driving-specific |
| 76 | VOLD | arXiv 2510.23497, 2025 (CVPR 2026) | [notes/vold.md](notes/vold.md) | Not verified — GRPO + on-policy distillation; found cold-start alignment is a precondition for transfer to work at all |
| 77 | Chorus | arXiv 2512.17817, 2025 (CVPR 2026 Oral) | [notes/chorus-3dgs.md](notes/chorus-3dgs.md) | Not verified — multi-teacher pretraining with shared encoder + teacher-specific projectors, clean reference case for teacher-type design vs. Drive-KD's triad |
| 78 | Expert-Teacher-Student (Domain Adaptive Detection) | CVPR 2026 | [notes/expert-teacher-student-dom-adapt.md](notes/expert-teacher-student-dom-adapt.md) | Not verified — three-tier expert/teacher/student structure, extends PerceptDrive's frozen-provider pattern with an extra domain-adaptation tier |
| 79 | GTR-Turbo | arXiv 2512.13043, 2025 (CVPR 2026) | [notes/gtr-turbo.md](notes/gtr-turbo.md) | Not verified — **merged-checkpoint-as-free-teacher directly parallels SAMoE-VLA's parameter merge**; quantifies the compute savings from weight-space tricks |
| 80 | AVION | CVPR 2026 | [notes/avion-aerial-vlm.md](notes/avion-aerial-vlm.md) | Not verified — aerial-imagery VLM prompt distillation, BEV-adjacent viewpoint but not driving-specific |
| 81 | ActiveAD | arXiv 2403.02877, 2024 (CVPR 2026) | [notes/activead.md](notes/activead.md) | Not verified — planning-oriented active learning, data efficiency, not MoE/driving-mechanism relevant |
| 82 | Beyond Rule-Based Agents (Active Markov Games) | CVPR 2026 | [notes/active-markov-games.md](notes/active-markov-games.md) | Not verified — multi-agent co-evolutionary traffic simulation |
| 83 | CausalVAD | arXiv 2603.18561, 2026 (CVPR 2026) | [notes/causalvad.md](notes/causalvad.md) | Not verified — sparse causal intervention against spurious correlations |
| 84 | CogDriver | arXiv 2509.00789, 2025 (CVPR 2026, Li Auto) | [notes/cogdriver.md](notes/cogdriver.md) | Not verified — sparse temporal memory for decision coherence; fifth alternative to the H_BEV staleness gap |
| 85 | ColaVLA | arXiv 2512.22939, 2025 (CVPR 2026) | [notes/colavla.md](notes/colavla.md) | Released (github.com/pqh22/ColaVLA) — only two VLM forward passes per decision; directly answers "SA-MoE doesn't save compute" |
| 86 | Diffusion Forcing Planner | arXiv 2606.11019, 2026 (CVPR 2026) | [notes/diffusion-forcing-planner.md](notes/diffusion-forcing-planner.md) | Not verified — history/current/future segments get independent noise levels; alternative future-conditioning design to PerceptDrive's |
| 87 | Drive My Way | arXiv 2603.25740, 2026 (CVPR 2026) | [notes/drive-my-way.md](notes/drive-my-way.md) | Released (dmw-cvpr.github.io) — user-embedding personalization for VLA driving |
| 88 | DrivePI | arXiv 2512.12799, 2025 (CVPR 2026) | [notes/drivepi.md](notes/drivepi.md) | Released (github.com/happinesslz/DrivePI) — unified 4D MLLM for understanding+perception+prediction+planning at only 0.5B params; cleanest full-pipeline reference for the zhuzixin idea |
| 89 | DriveVLN | CVPR 2026 | [notes/drivevln.md](notes/drivevln.md) | Not verified — mapless VLN extended to driving |
| 90 | Driving on Registers (DrivoR) | arXiv 2601.05083, 2026 (CVPR 2026) | [notes/driving-on-registers.md](notes/driving-on-registers.md) | Not verified — camera-aware register tokens compress multi-view features, 3x throughput; same compression-token pattern as PerceptDrive's Query Bank |
| 91 | DVGT | arXiv 2512.16919, 2025 (CVPR 2026, Tsinghua/Xiaomi EV) | [notes/dvgt.md](notes/dvgt.md) | Not verified — driving visual geometry transformer, same category as VGGT (PerceptDrive's geometry teacher) |
| 92 | E3AD | arXiv 2512.04733, 2025 (CVPR 2026) | [notes/e3ad.md](notes/e3ad.md) | Not verified — emotion-aware VLA for personalized driving |
| 93 | Efficient Equivariant Transformer (DriveGATr) | arXiv 2604.01466, 2026 (CVPR 2026, Waabi) | [notes/equivariant-transformer-selfdriving.md](notes/equivariant-transformer-selfdriving.md) | Not verified — SE(2)-equivariant agent modeling without costly relative positional encoding |
| 94 | EventDrive | arXiv 2606.18242, 2026 (CVPR 2026) | [notes/eventdrive.md](notes/eventdrive.md) | Not verified — **driving, genuine modality-routing MoE** (event-stream vs. frame-based), another expert-partition dimension (by sensor modality) |
| 95 | GuideFlow | arXiv 2511.18729, 2025 (CVPR 2026) | [notes/guideflow.md](notes/guideflow.md) | Not verified — constrained flow matching for planning, complements SAMoE-VLA/WAM-Flow's flow-matching trajectory generation |
| 96 | HG-Lane | arXiv 2603.10128, 2026 (CVPR 2026) | [notes/hg-lane.md](notes/hg-lane.md) | Released (github.com/zdc233/HG-Lane) — adverse-weather lane scene generation, data augmentation |
| 97 | HybridDriveVLA | CVPR 2026 | [notes/hybriddrivevla.md](notes/hybriddrivevla.md) | Not verified — Visual CoT + ToT waypoint evaluation for VLA |
| 98 | KnowVal | arXiv 2512.20299, 2025 (CVPR 2026) | [notes/knowval.md](notes/knowval.md) | Not verified — driving knowledge graph + value model |
| 99 | LEAD | arXiv 2512.20563, 2025 (CVPR 2026, NVIDIA) | [notes/lead.md](notes/lead.md) | Released (github.com/kesai-labs/lead) — names the learner-expert visibility/uncertainty asymmetry problem, a caution for any frozen-teacher distillation design |
| 100 | Learning to Drive is a Free Gift | arXiv 2602.22091, 2026 (CVPR 2026) | [notes/learning-to-drive-free-gift.md](notes/learning-to-drive-free-gift.md) | Not verified — multi-modal teacher pseudo-supervision from unposed video, alternative route to building perception priors vs. direct distillation |
| 101 | MindDriver | arXiv 2602.21952, 2026 (CVPR 2026) | [notes/minddriver.md](notes/minddriver.md) | Released (github.com/hotdogcheesewhite/MindDriver) — progressive semantic→spatial→trajectory reasoning, curriculum-style design |
| 102 | Neuro-Cognitive Reward Modeling | arXiv 2603.25968, 2026 (CVPR 2026) | [notes/neuro-cognitive-reward.md](notes/neuro-cognitive-reward.md) | Not verified — EEG-guided RL reward shaping |
| 103 | Open-Ended Instruction Realization (Multi-Planner Scheduling) | arXiv 2604.08031, 2026 (CVPR 2026) | [notes/open-ended-instruction-realization.md](notes/open-ended-instruction-realization.md) | Not verified — scheduling-centric decoupling of slow semantic reasoning from fast control, same efficiency problem ColaVLA addresses |
| 104 | PanDA | arXiv 2604.19379, 2026 (CVPR 2026) | [notes/panda-3d-panoptic.md](notes/panda-3d-panoptic.md) | Not verified — unsupervised domain adaptation for multimodal 3D panoptic segmentation |
| 105 | ParkGaussian | arXiv 2601.01386, 2026 (CVPR 2026) | [notes/parkgaussian.md](notes/parkgaussian.md) | Not verified — 3DGS parking scene reconstruction |
| 106 | Percept-WAM | arXiv 2511.19221, 2025 (CVPR 2026) | [notes/percept-wam.md](notes/percept-wam.md) | Not verified — **⚠ name collision with PerceptDrive, different paper/team/architecture — see note in file** |
| 107 | Prune2Drive | arXiv 2508.13305, 2025 (CVPR 2026) | [notes/prune2drive.md](notes/prune2drive.md) | Not verified — plug-and-play multi-view VLM token pruning, no retraining needed; directly answers "SA-MoE doesn't save compute" |
| 108 | Reliable Policy Transfer | CVPR 2026 | [notes/reliable-policy-transfer.md](notes/reliable-policy-transfer.md) | Not verified — one uncertainty signal drives attention/entropy/transfer alignment jointly, parallels Beta-KD's uncertainty weighting |
| 109 | ResAD | arXiv 2510.08562, 2025 (CVPR 2026) | [notes/resad.md](notes/resad.md) | Not verified — predicts residual from an inertial prior rather than absolute trajectory |
| 110 | RLFTSim | arXiv 2605.19033, 2026 (CVPR 2026, Huawei) | [notes/rlftsim.md](notes/rlftsim.md) | Not verified — RL fine-tuning for traffic simulation realism |
| 111 | SafeDrive | arXiv 2602.18887, 2026 (CVPR 2026) | [notes/safedrive.md](notes/safedrive.md) | Not verified — instance-level sparse world model for safety reasoning, finer granularity than PerceptDrive's scene-level future conditioning |
| 112 | Scaling-Aware Data Selection (MOSAIC) | arXiv 2604.08366, 2026 (CVPR 2026, NVIDIA) | [notes/scaling-aware-data-selection.md](notes/scaling-aware-data-selection.md) | Not verified — scaling-law-guided data mixture selection |
| 113 | Sensor2Sensor | arXiv 2605.22809, 2026 (CVPR 2026, Waymo) | [notes/sensor2sensor.md](notes/sensor2sensor.md) | Not verified — cross-embodiment sensor-style video conversion via 4DGS + diffusion |
| 114 | SGDrive | arXiv 2601.05640, 2026 (CVPR 2026) | [notes/sgdrive.md](notes/sgdrive.md) | Released (github.com/LogosRoboticsGroup/SGDrive) — scene-agent-goal cognitive hierarchy, alternative decision-layering to SAMoE-VLA's CMCA/DrivePI's unified structure |
| 115 | SimScale | arXiv 2511.23369, 2025 (CVPR 2026 Oral, OpenDriveLab) | [notes/simscale.md](notes/simscale.md) | Released — large-scale real-world simulation data augmentation |
| 116 | SpaceDrive | arXiv 2512.10719, 2025 (CVPR 2026, Mercedes-Benz) | [notes/spacedrive.md](notes/spacedrive.md) | Not verified — spatial info as positional encodings instead of text tokens |
| 117 | Spatial Retrieval Augmented AD | arXiv 2512.06865, 2025 (CVPR 2026) | [notes/spatial-retrieval-ad.md](notes/spatial-retrieval-ad.md) | Not verified — offline map-image retrieval as a plug-and-play extra input, no new sensors needed |
| 118 | The Blind Spot of Adaptation (DEA) | arXiv 2604.04857, 2026 (CVPR 2026) | [notes/blind-spot-adaptation.md](notes/blind-spot-adaptation.md) | Not verified — **精读完成，⚠已修正**：experts are trained end-to-end, NOT distilled from external models — matches the "frozen backbone + lightweight routing" *shape* (good for the forgetting angle) but not the "distill other MoEs" mechanism |
| 119 | TrafficAlign | arXiv 2606.29097, 2026 (CVPR 2026) | [notes/trafficalign.md](notes/trafficalign.md) | Not verified — LLM-generated adversarial/long-tail traffic scenarios |
| 120 | Unifying Language-Action Understanding and Generation (LinkVLA) | arXiv 2603.01441, 2026 (CVPR 2026) | [notes/unifying-language-action.md](notes/unifying-language-action.md) | Not verified — shared discrete codebook for language+action tokens, -86% inference time |
| 121 | Unleashing VLA Potentials (ELF-VLA) | arXiv 2603.01063, 2026 (CVPR 2026) | [notes/unleashing-vla-potentials.md](notes/unleashing-vla-potentials.md) | Not verified — structured failure-diagnostic RL feedback instead of scalar reward, relevant if training a router via RL like MoE-GRPO |
| 122 | VGGDrive | arXiv 2602.20794, 2026 (CVPR 2026, Xiaomi EV) | [notes/vggdrive.md](notes/vggdrive.md) | Not verified — plug-and-play frozen-3D-model-to-VLM geometric injector, same freeze+inject pattern as PerceptDrive |
| 123 | WAM-Flow | arXiv 2512.06112, 2025 (CVPR 2026, Fudan) | [notes/wam-flow.md](notes/wam-flow.md) | Released — discrete flow-matching planner; **⚠ "WAM" name collision with PerceptDrive's World-Action Model Backbone — unrelated paper, see note in file** |

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
