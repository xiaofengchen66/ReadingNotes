# DriveZero: End-to-End Driving Beyond Human Demonstrations

- **Venue / year:** Technical Report, arXiv 2609.06055, September 2026
- **Authors:** AD & Robotics, L3 Team, Xiaomi EV
- **Link:** https://arxiv.org/abs/2609.06055
- **Website:** https://xiaomiautol3.github.io/DriveZero
- **paper_matrix.csv id:** `drivezero`
- **Input / 输入:** 多视角摄像头图像（camera-only，无激光雷达）
- **Output / 输出:** 未来轨迹（trajectory proposals），由DriveRL教师监督生成

---

## TL;DR

**问题：** 所有端到端驾驶系统都在模仿人类驾驶日志，性能上限被人类示范质量和覆盖范围锁死。

**Gap：** 没有系统能在不依赖人类轨迹监督的情况下，在多个主流 benchmark 上达到 SOTA。

**Novelty：** 把驾驶拆成"感知"和"行动"两件本质不同的事，分别用最适合各自的方式预训练，再通过蒸馏合一——行动模型靠RL自主探索而非模仿人类。

**核心洞察：** 感知需要海量多样视觉数据（靠视觉大模型），行动需要闭环交互反馈（靠RL），两者学习范式根本不同，不该用同一个方式训练。

**方法：** DriveVFM 融合4个冻结视觉大模型做感知骨干；DriveRL 把真实日志转成混合智能体仿真世界，用PPO训练特权教师策略；DriveZero 用DriveVFM编码图像，向DriveRL的滚动轨迹学习（含增强目标），不用人类轨迹。

**结果：** navtest 95.3 PDMS（超过人类司机 94.8）；navhard 57.1 EPDMS；HUGSIM 46.6 HD-Score（zero-shot）；nuPlan 93.57 均分，超越Log-Replay专家。

**意义：** 首个在主要 benchmark 上无人类轨迹监督达到 SOTA 且超越人类司机分数的端到端驾驶系统（据作者所知）。定位为工业技术报告，目标是"超越人类开车"而非提出新算法。

---

## 贡献列表（按重要性排序）

**① 无人类轨迹监督的端到端驾驶**（核心结果）
DriveZero 完全不用人类轨迹作监督信号，向RL教师学习。navtest 95.3 PDMS > 人类 94.8，NAVSIMv1/v2/HUGSIM 三个 benchmark 均达 SOTA。

**② DriveRL：混合智能体闭环RL框架**（方法贡献）
把nuPlan真实日志转成混合智能体交互世界（日志回放/规则行为/学习策略三类背景车共存），用PPO训练5.7M参数特权策略。nuPlan 93.57均分，超越Log-Replay专家，non-reactive和reactive两种模式均超。

**③ DriveVFM：无标注视觉大模型融合骨干**（方法贡献）
将DINOv3、SigLIP2、SAM、Depth Anything V2四个冻结模型蒸馏为单一驾驶骨干，无需任何任务标注。

**④ 增强目标查询**（训练技巧）
对DriveRL教师施加不同驾驶意图（目标点）查询，生成日志里从未出现过的多样化轨迹作为监督，扩大训练信号覆盖范围。

---

## Novelty 分类判断

**组合创新（改进组合）**：三个组件（视觉模型融合、闭环RL、知识蒸馏）单独均非首创，但"用RL教师替代人类示范"+"混合智能体仿真世界"的具体组合方式有新意，且结果超越人类这一目标导向明确。这是工业技术报告，工程价值高于学术方法新颖性。

---

## 中文精读

### 1. 解决了什么问题

所有端到端自动驾驶——包括DriveVLM、Qwen-Drive等——都在做**模仿学习（Imitation Learning）**：给模型看人类怎么开，让它学着开。

这有一个根本上限：**人类开车不是最优解**。人类会犯错、会疲劳、会做次优决策，模型模仿得再好，也只能达到人类水平，不能超过。

DriveZero 的出发点：**能不能绕过人类示范，让模型自己探索出更优的驾驶策略？**

### 2. 新意在哪里

**核心洞察：感知和行动需要完全不同的学习范式。**

- 感知（理解世界）：靠海量多样的视觉数据，越多越好，监督信号来自图像本身 → 适合视觉大模型蒸馏
- 行动（与世界交互）：靠闭环反馈，必须看到操作结果才能学 → 适合RL，不适合在静态日志上做监督学习

把两件事分开预训练，分别用最适合的方式，最后合并 —— 这是DriveZero区别于其他工作的核心设计哲学。

对比三篇论文的根本立场：
- DriveVLM：语言推理是开好车的关键
- Qwen-Drive：统一表征是多任务的关键
- **DriveZero：RL自主探索才能超越人类**

### 3. Idea 具体落在哪里

**DriveRL的混合智能体仿真世界**是最核心的工程创新：
真实日志里的背景车是"死"的（固定轨迹）。DriveZero给每辆背景车分配独立的"行为提供者"（日志回放/规则/学习策略三种），让场景动起来，RL智能体的操作会引发其他车辆的真实反应，形成真正的闭环。

**特权教师（5.7M参数）**：极小的模型，但能看到一切（完整场景状态），靠PPO在仿真里训到极强。小而强，因为"作弊权限"弥补了参数少的劣势。

**增强目标**：教师被问"向左转该怎么开"，生成日志里没有的轨迹，监督信号从人类行为覆盖范围扩展到任意目标下的最优行为。

### 4. 最大难点在哪里

**仿真真实性（Simulation Fidelity）**问题：
RL在仿真世界里学得再好，仿真和真实世界之间总有差距（sim-to-real gap）。DriveZero用真实nuPlan日志作为仿真世界的骨架（道路、初始位置来自真实录像），一定程度缓解了这个问题，但仿真里的背景车行为仍不如真实世界复杂。

**评测局限**：主要结果来自NAVSIM（伪闭环）和nuPlan（闭环仿真），真实车辆道路测试结果未见报告。95.3分超过人类是在benchmark上，不代表真实道路上超过人类。

### 5. 局限 / 对我项目的相关性

- 不能回答自然语言问题（没有VQA能力），不适合"座舱+驾驶"一体化
- 真实道路验证缺失，benchmark分数≠真实性能
- 对CAVAS-UB数字孪生方向：混合智能体仿真世界的构建方式（把真实日志激活为可交互场景）直接相关——数字孪生本质上也在做这件事，DriveZero提供了一种具体的背景车行为建模参考

### 6. 是否能连 GitHub

**部分开放。** 有项目主页（xiaomiautol3.github.io/DriveZero），论文提交时代码未见完整开源。nuPlan数据集本身是公开的，但DriveZero的训练pipeline尚未公开发布。

### 7. 关键原文摘录（中英对照）

**① 核心动机（Abstract）**
> "Most end-to-end autonomous-driving systems learn by imitating human driving logs, leaving their learned behavior constrained by the quality and behavioral coverage of the recorded trajectories."

中译：大多数端到端自动驾驶系统通过模仿人类驾驶日志来学习，使其学到的行为受限于记录轨迹的质量和行为覆盖范围。
解读：这句话定义了整个领域的根本局限，DriveZero的全部动机都来自这一个问题。

**② 感知vs行动的学习范式分歧（System Overview）**
> "The two models call for different learning recipes: perception must understand the world, and benefits from massive and diverse visual data; action must interact with it, and requires closed-loop feedback."

中译：两个模型需要不同的学习方案：感知必须理解世界，受益于海量多样的视觉数据；行动必须与世界交互，需要闭环反馈。
解读：这是DriveZero设计的理论基础，把"感知"和"行动"在认识论层面分开是这篇论文最值得借鉴的思路。

**③ 超越人类的关键数字（Abstract）**
> "DriveZero achieves state-of-the-art performance on NAVSIMv1, NAVSIMv2 and the closed-loop HUGSIM benchmark without any human trajectory supervision: it reaches 95.3 PDMS on navtest, surpassing the human driver (94.8)."

中译：DriveZero在NAVSIMv1/v2和闭环HUGSIM上实现SOTA，且不使用任何人类轨迹监督：navtest上达到95.3 PDMS，超过人类司机（94.8）。
解读：注意限定词"without any human trajectory supervision"——这才是这个分数的真正含义，不只是分数高，而是不靠人类示范还能高。

---

## 可略过的背景（如果时间有限）

- nuPlan benchmark 的具体评测协议细节（记住"93.57均分、超Log-Replay专家"即可）
- DriveVFM 四个视觉模型的具体融合层数和超参数
- PPO的具体clip范围等训练超参数

---

## 假设、局限与质疑点

- **Sim-to-real gap 未解决**：仿真世界再真实也和路上不一样，benchmark好≠真实驾驶好
- **95.3 > 94.8 的统计显著性**：0.5分的差距在benchmark误差范围内是否显著？论文未给出置信区间
- **没有VQA能力**：DriveRL的"特权"信息在真实部署时不存在，学生模型是否真正继承了教师能力需要验证
- **增强目标的多样性上限**：augmented goals仍受限于仿真世界能表达的场景范围

---

## 一个月后记住5件事

1. **超越人类**：第一个在主流benchmark上无人类轨迹监督超过人类司机分数（95.3 vs 94.8）
2. **感知≠行动**：两件事学习范式根本不同，分开预训练是核心设计哲学
3. **DriveRL**：把真实日志变成混合智能体可交互世界，PPO训练特权教师
4. **DriveVFM**：4个冻结视觉大模型蒸馏成一个骨干，零标注
5. **组合创新，工业报告**：工程价值>学术新颖性，benchmark分数≠真实道路性能

---

## English at-a-glance summary

| Aspect | Summary |
|--------|---------|
| **Problem** | All E2E driving systems imitate human logs → performance ceiling = human quality. Humans are not optimal drivers. |
| **Novelty** | Decomposes driving into perception (needs visual diversity → VFM distillation) and action (needs closed-loop feedback → RL), pretrain each optimally, unify by distillation. |
| **Core insight** | Perception and action require fundamentally different learning recipes. RL self-exploration can surpass human demonstration ceiling. |
| **DriveRL** | Converts real nuPlan logs into mixed-agent interactive worlds (log replay + rule-based + learned policies). 5.7M privileged policy trained with PPO. nuPlan 93.57 mean score > Log-Replay expert. |
| **DriveVFM** | Distills 4 frozen VFMs (DINOv3, SigLIP2, SAM, Depth Anything V2) into one driving backbone. Zero task-specific annotations. |
| **DriveZero** | Camera-only planner. Encodes images with DriveVFM. Learns from DriveRL rollouts (not human trajectories), including augmented goal rollouts. |
| **Results** | navtest 95.3 PDMS (> human 94.8), navhard 57.1 EPDMS, HUGSIM 46.6 zero-shot HD-Score. SOTA on NAVSIMv1/v2/HUGSIM without human trajectory supervision. |
| **Limitation** | No VQA capability; sim-to-real gap unaddressed; no real-road test results; 0.5-point human gap may be within benchmark noise. |
| **Innovation type** | Engineering/combination innovation. Industrial technical report, not a novel algorithm paper. |
| **Code** | Project page exists; full training pipeline not yet open-sourced as of report date. |

---

## 与其他论文的关键对比

> 💡 这张表是核心，看完每篇论文都可以回来对照。

| | DriveVLM | Qwen-Drive | **DriveZero** |
|--|----------|-----------|--------------|
| **核心主张** | VLM推理能力→规划 | VLM表征→多任务 | **RL策略→超人类** |
| **VLM角色** | 推理引擎（语言CoT） | 共享主干（表征） | 几乎不用 |
| **学习来源** | 人类示范 + 语言标注 | 人类示范 | **RL自主探索** |
| **感知方式** | VLM直接看图 | VLM + BEV头 | 4个视觉大模型融合 |
| **能回答问题吗** | 是 | 是 | 否 |
| **能超越人类吗** | 否（上限是人类） | 否（上限是人类） | **是（RL自主探索）** |
| **核心创新类型** | 新insight | 新insight | 组合创新 |

---

## 本文问到的词

| 术语 | 首次出现位置 |
|------|------------|
| PPO（近端策略优化） | System Overview / DriveRL |
| 特权策略 / Privileged Policy | DriveRL描述 |
| 混合智能体交互世界 / Mixed-agent Interactive World | DriveRL描述 |
| 知识蒸馏 / Knowledge Distillation | DriveVFM / DriveZero |
| 增强目标 / Augmented Goals | DriveZero描述 |
| 组合创新 vs 新算法 vs 新insight | 阅读讨论 |
| 模仿学习 / Imitation Learning | 问题动机 |

