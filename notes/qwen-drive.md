# Qwen-Drive-1.0: An Initial Step towards a Vision-Language Foundation Model for Autonomous Driving

- **Venue / year:** Technical Report, arXiv 2609.00111, 2026
- **Authors:** Qwen Team, Huazhong University of Science and Technology
- **Link:** https://arxiv.org/abs/2609.00111
- **GitHub:** https://github.com/QwenLM/Qwen-Drive-1.0
- **HuggingFace:** https://huggingface.co/Qwen/Qwen-Drive-1.0-4B
- **paper_matrix.csv id:** `qwen-drive`
- **Input / 输入:** 多路环视摄像头图像（多帧历史）+ 文字 Prompt（任务描述/问题）
- **Output / 输出:** 三类并行输出——① 3D感知结果（检测框/语义占用地图/BEV地图）② 未来轨迹点（5秒,10Hz,(x,y,heading)）③ 自然语言回答（驾驶VQA / 通用VQA）

---

## TL;DR

**问题：** 传统自动驾驶感知、规划、问答三个任务各用专门模型，割裂、冗余、缺乏常识推理。

**Gap：** 已有驾驶VLM要么只做VQA不做规划，要么只做规划不做3D感知，没有在单一预训练VLM框架内统一三者。

**Novelty：** 以不改变VLM架构为约束，外挂BEV感知头（探针）+ Planning Expert（Flow Matching规划），共享VLM表征驱动三任务。

**核心洞察：** 强大的通用VLM表征已隐含了足够的3D场景理解，BEV感知头不需要自己"理解"场景，只需把VLM表征里的3D信息"读出来"（probe）。

**方法：** Qwen3.5-4B作主干，视觉编码器提取多视角多帧特征，两个外挂头分别做BEV感知和Flow Matching规划；分阶段训练避免灾难性遗忘；统一数据pipeline对齐三类异构数据。

**结果：** NAVSIM PDMS 90.7（RL）、WOD-E2E RFS 7.91、Driving VQA全面领先同量级模型，通用VLM能力基本保留。

**意义：** 首个在单一预训练VLM内统一3D感知+VQA+规划的工作（据作者所知）；为"座舱+驾驶一体化"部署提供可行方案。

---

## 贡献列表（按重要性排序）

**① 共享VLM架构下的三任务统一框架**（核心贡献）
不改Qwen3.5-4B任何权重结构，外挂BEV感知头和Planning Expert，三任务共享同一份表征。证据：README架构图，模型目录结构（VLM 9.1GB为主干，感知头0.5GB，规划头2.1GB）。

**② BEV感知头作为"表征探针"**（方法贡献）
将感知头定位为"探测VLM表征中3D信息"的科学验证工具，不只是功能模块。nuScenes上达43.95 mAP / 60.99 mIoU，OpenScene上43.45 mAP / 71.27 mIoU，与专用视觉3D检测器高度竞争。

**③ Flow Matching规划专家 + RL优化**（方法贡献）
从加噪轨迹去噪，条件是VLM缓存的K,V。SFT版NAVSIM 88.2，RL版90.7（best-of-6 91.4）。RL版直接优化评测目标，超越人类示范上限。

**④ 统一数据Pipeline**（工程贡献）
将三类异构数据（感知标注/规划轨迹/VQA问答）对齐：坐标系统一、轨迹重采样为5s/10Hz/(x,y,heading)、用大模型批量重标注VQA回答。支撑多任务联合训练。

**⑤ 分阶段训练方案**（训练贡献）
先冻结VLM主干只训外挂头，再混入通用数据联合微调，避免灾难性遗忘。Driving VQA大幅超越Qwen3.5-4B基座，通用VQA（MMBench/MMMU等）基本持平。

---

## Novelty 分类判断

**新方法（改进组合）**：三任务统一框架本身不是首创概念，但"以不改VLM架构为约束"+"BEV头作探针"+"Flow Matching规划"的组合在单一预训练VLM框架下首次实现三任务统一，是已有技术的有价值组合创新。

---

## 中文精读

### 1. 解决了什么问题

传统自动驾驶是"感知模型 + 规划模型 + （没有VQA）"三件套，各模块割裂：感知模型（如BEVFormer）擅长3D几何，不会语言推理；规划模块只吃感知输出，不理解语义；没有任何模块能回答"前方场景有什么危险"这类开放问题。

更深的问题：BEV类感知模型（BEV Transformer是主体，是核心大脑）的表征是"窄的"——只服务驾驶感知任务，无法复用到语言问答或常识推理。这种割裂导致：①长尾场景（路上倒树、动物横穿）缺乏常识兜底；②无法部署在"座舱+驾驶"一体化平台（同一个硬件既要控车又要做语音助手）。

Qwen-Drive要解决的是：**能不能以一个通用VLM为主体，不改其架构，让它同时支撑3D感知、VQA、规划三个任务？**

### 2. 新意在哪里

**核心新意：主体换人。**

以前：BEV Transformer是主体，负责理解3D场景，规划是附件。
现在：VLM（Qwen3.5-4B）是主体，负责理解一切，BEV Transformer退化为"翻译器"——把VLM表征里已有的3D信息读出来并转成结构化输出。

这个角色倒转背后的技术赌注是：强大的通用VLM在海量图文预训练后，表征里已经隐含了丰富的3D场景信息（"前方有辆车"不只是语言知识，也包含了空间几何感知）。BEV感知头作为"探针（probe）"，验证了这一假设。

第二个新意：**Flow Matching规划**。不直接预测轨迹坐标，而是从随机噪声轨迹出发，以VLM缓存的K,V（场景理解记忆）为条件，逐步去噪到干净轨迹。好处是可以best-of-N采样，多条候选里选最优。

### 3. Idea 具体落在哪里

**架构三件套（Figure 3）：**

左侧BEV感知头：图像→视觉编码器→两路并行（VLM多尺度特征 + View Transform体素特征）→合并→BEV Transformer→三个输出头（Occ/Detection/Map）。两路特征合并是关键：VLM提供语义理解，View Transform提供几何投影，合并后比任何一路单独更强。

右侧Planning Expert：VLM正常推理，缓存内部Attention的K,V（场景记忆）传给规划头；规划头输入带噪声轨迹+时间步+自车状态，通过4层GQA+AdaLN，以K,V为条件逐步去噪，输出干净轨迹。

**多视角输入序列化：** 给每个图像token贴视角标签（[front]/[back]/...）和帧标签（frame:k），用普通词汇token，不改架构。VQA用帧优先排列，规划用视角优先排列——同一方向的时序帧相邻，模型更容易感知物体运动趋势。

### 4. 最大难点在哪里

**数据统一**是核心工程难点，不在模型结构：

- 感知数据（nuScenes/OpenScene等）坐标系各异，类别标签不统一
- 规划数据（nuPlan/WOD-E2E等）轨迹采样率/时长/格式不同
- VQA数据（DriveLM/LingoQA等）回答风格五花八门，存在格式不一致和事实错误

统一方案：坐标系对齐（自车中心）+ 轨迹重采样（5s/10Hz）+ 大模型批量重标注VQA回答。这套pipeline的质量直接决定多任务联合训练的上限。

**评测可靠性**是第二难点：三种评测设置（开环/伪闭环/闭环）严格程度差异很大，只报开环指标容易高估真实性能。论文在三种设置下都测是可信度的保障，但AlpaSim闭环结果只说"promising potential"，说明闭环仍有差距。

### 5. 局限 / 对我项目的相关性

- 4B参数，24GB+ GPU才能推理，车载部署成本不低
- RL训练只在部分推理模式（reasoning planning）下有效，直接规划模式仍用SFT版
- 闭环评测结果保守（只说promising），真实驾驶验证未见报告
- 对CAVAS-UB数字孪生方向：**共享表征多任务**的设计思路可迁移——仿真环境里的感知/预测/问答同样可以共享一个VLM主干，不需要三套独立模型

### 6. 是否能连 GitHub

**可以复现核心功能。** 代码、权重（4B SFT+RL规划头+感知头）均已开源，HuggingFace可直接下载。附带demo数据（4个WOD-E2E场景+6个感知帧），24GB GPU可跑。VQA和规划两种模式均可运行。训练代码未完整开放（只有推理），完整数据pipeline未开源。

### 7. 关键原文摘录（中英对照）

**① 三个设计原则的集中表述（Section 1）**
> "We argue that a practical vision-language foundation model for driving should satisfy three design requirements. First, the pretrained VLM architecture should remain unchanged to preserve ease of use. Second, an explicit perception probe should expose and evaluate 3D scene information rather than relying on textual spatial reasoning alone. Third, the model should acquire driving scene-understanding knowledge while retaining most of its general-purpose capabilities."

中译：我们认为，一个实用的驾驶视觉语言基础模型应满足三个设计要求：①预训练VLM架构保持不变；②用显式感知探针暴露3D场景信息，而非只依赖语言空间推理；③学习驾驶知识的同时保留大部分通用能力。
解读：这三条原则就是整篇论文的设计约束，所有架构选择都能追溯到这三条。读论文时任何"为什么这样设计"的问题，先回来对照这三条。

**② 探针定位（Section 2.2）**
> "It serves as a probe of the 3D information accessible from the shared representations and provides an explicit, inspectable interface to 3D scene structure."

中译：它作为共享表征中可访问的3D信息的探针，提供了一个对3D场景结构的显式可检查接口。
解读："探针"二字是论文对BEV感知头角色的核心定位——不是主动理解，而是被动读取已有信息。"显式可检查"对应黑箱表征的不透明性——感知头让人类能验证模型到底看到了什么。

**③ 开源代码调用示例（README）**
```python
result = model.run(InferenceMode.REASONING_PLANNING, scene=scene, num_samples=6)
print(result.trajectories.shape)   # (6, 50, 3) -> (x, y, heading), 5s at 10Hz
```
解读：`num_samples=6`对应best-of-6采样——采6条候选轨迹选最优，这是RL版规划头的标准用法。输出shape `(6, 50, 3)`直接说明：6条候选、50个时间步（5秒×10Hz）、每步(x,y,heading)三维。

---

## 可略过的背景（如果时间有限）

- Section 2.1 中关于GQA/GDN层数的具体超参数（架构细节，不影响理解核心思路）
- Appendix中各benchmark的详细评测协议（除非要复现实验）
- 与各baseline的逐指标对比表格（记住"Driving VQA全面领先，通用VQA基本持平"的结论即可）

---

## 假设、局限与质疑点

- **假设"VLM表征隐含3D信息"的普适性未验证**：只在Qwen3.5上做了实验，换一个VLM基座（如LLaMA）结论是否成立未知
- **BEV感知头性能与专用模型仍有差距**：论文说"highly competitive"，但具体数字显示与BEVFormer等专用模型仍有差距，"competitive"的边界需看具体表格
- **RL训练只优化评测目标**：NAVSIM PDMS 90.7可能存在对评测指标的过拟合，真实驾驶场景表现未验证
- **数据pipeline的重标注质量依赖大模型**：VQA回答由大模型批量重写，引入新的偏差来源

---

## 一个月后记住5件事

1. **主体换人**：VLM是大脑，BEV Transformer退化为几何翻译器/探针
2. **共享表征三任务**：感知/规划/VQA共用一份Qwen表征，各挂一个小头
3. **Flow Matching规划**：从噪声轨迹去噪，可以best-of-N采样
4. **分阶段训练防遗忘**：先冻结VLM主干练头，再混通用数据微调
5. **三种评测严格程度**：开环 < 伪闭环（NAVSIM）< 闭环（AlpaSim）

---

## English at-a-glance summary

| Aspect | Summary |
|--------|---------|
| **Problem** | Traditional AD uses separate specialist models for perception, planning, and QA — siloed, redundant, no common-sense reasoning. No existing driving VLM unifies all three within a single pretrained VLM. |
| **Novelty** | Keeps pretrained Qwen3.5-4B unchanged; attaches a BEV perception head (probe) and a Flow Matching Planning Expert — first work to unify 3D perception + VQA + planning in one pretrained VLM. |
| **Core insight** | A strong general VLM already encodes 3D scene structure in its representations; the BEV head doesn't need to "understand" — it just reads out what's already there. BEV Transformer goes from brain → translator. |
| **Key trick** | Flow Matching: plan by denoising a noisy trajectory conditioned on VLM's cached K,V (scene memory). Enables best-of-N sampling. RL further optimizes beyond human demonstrations. |
| **Hardest part** | Unified data pipeline: align heterogeneous perception annotations, resample trajectories to one format (5s/10Hz/(x,y,heading)), re-annotate VQA responses with LLM for consistency. |
| **Results** | NAVSIM PDMS 90.7 (RL), WOD-E2E RFS 7.91, Driving VQA dominates peers at same scale; general VQA (MMBench/MMMU) largely preserved. |
| **Limitation** | Closed-loop results only "promising"; RL optimizes benchmark metrics (potential overfitting); BEV head still trails specialist models; 24GB+ GPU needed. |
| **Code / GitHub** | **Released and runnable.** Weights on HuggingFace (9.1GB VLM + 2.1GB planner + 0.5GB perception). Demo data included. Training pipeline not fully open-sourced. |

---

## 本文问到的词

| 术语 | 首次出现位置 |
|------|------------|
| 表征 / Representation | 架构介绍部分 |
| 探针 / Probe | Abstract / Section 2.2 |
| BEV（Bird's Eye View，鸟瞰视图） | Section 2.2 |
| SFT（监督微调） | 性能表格 |
| RL（强化学习） | 性能表格 |
| VLM（视觉语言模型） | Abstract |
| 统一数据Pipeline | Section 2.4 |
| 语义占用预测 / Semantic Occupancy Prediction | Section 2.2 |
| Semantic（语义） | 贯穿全文 |
| Explicit / Inspectable Interface（显式可检查接口） | Section 2.2 |
| 灾难性遗忘 / Catastrophic Forgetting | Section 2.3（训练设计动机） |
| 分阶段训练 / Staged Training | Section 2.3 |
| 开环评测 / Open-loop Evaluation | Section 4 |
| 闭环评测 / Closed-loop Evaluation | Section 4 |
| 伪闭环评测 / Pseudo-closed-loop | Section 4 |
| 视觉Token / Visual Token | Section 2.1 |
| 自回归生成 / Autoregressive Generation | Section 2.1 |
| Frame-major / View-major 序列化 | Section 2.1 |
| Flow Matching（流匹配） | Section 2.2 |
| K, V（键值对） | Section 2.2 / Figure 3 |
| Voxel（体素） | Section 2.2 / Figure 3 |

