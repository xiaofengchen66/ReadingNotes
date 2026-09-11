# Drive-JEPA: Video JEPA Meets Multimodal Trajectory Distillation for End-to-End Driving

```
Input:  多帧前视摄像头图像（历史帧）+ 自车状态（速度、加速度、转向指令）
Output: 未来驾驶轨迹（waypoints：BEV 坐标 + 航向角），以及多条候选轨迹 + 打分
```

- **arXiv**: 2601.22032
- **Venue / Year**: arXiv 2026（1月投出，7月修订）
- **Code**: 代码待发布（V-JEPA 基础代码：facebookresearch/jepa）
- **Benchmark SOTA**: 93.3 PDMS（NAVSIM v1）、87.8 EPDMS（NAVSIM v2）

---

## TL;DR

**问题**：端到端自动驾驶面临两个瓶颈——①视频预训练对驾驶性能提升有限（像素级重建太重、latent方法有表征坍塌风险）；②每个场景只有一条人类轨迹作为监督，模型学不到"同一路口可以左转也可以直行"的多样行为。

**Gap**：现有 latent world model（LAW、World4Drive）没有证明能从大规模视频 scale 中受益，且存在 representation collapse；现有 proposal-centric 方法（iPad）的候选轨迹只靠单条人类轨迹监督，多样性不足。

**Novelty**：将 V-JEPA（视频 JEPA）引入自动驾驶预训练，天然防 collapse；同时引入从仿真器提取的多模态伪教师轨迹（Pseudo-Teacher Trajectories）作为额外监督。

**核心洞察**：在 latent 空间预测（而非重建像素）强迫模型学习"可预测的语义结构"（谁在移动、往哪走），忽略不重要的像素噪声；而仿真器可以穷举词表中8192条轨迹的优劣，为同一场景提供多种合理走法。

**方法**：① V-JEPA 在208小时驾驶视频上预训练 ViT encoder；② Waypoint-Anchored Proposal Generation 生成多条候选轨迹；③ Multimodal Trajectory Distillation 用仿真打分的伪教师轨迹扩展监督；④ Momentum-Aware 选择最终轨迹保证帧间平滑。

**结果**：NAVSIM v1 93.3 PDMS（SOTA）；仅用 V-JEPA encoder + 轻量 decoder 的无感知版本已达 89.0 PDMS，超过此前所有方法 3 分。

**意义**：证明了高质量视觉预训练本身就能大幅提升端到端驾驶性能，且可以与仿真器知识蒸馏解耦组合。

---

## 贡献列表（按重要性排序）

1. **V-JEPA 引入自动驾驶预训练**（新方法组合）：首次将 V-JEPA 用于驾驶视频预训练，EMA encoder 天然防 representation collapse，无感知设定下单独就比 SOTA 高 3 PDMS。*证据：Table 1 消融，感知自由设定 89.0 vs Epona 86.1。*

2. **Multimodal Trajectory Distillation**（新方法）：用 k-means 从人类轨迹构建8192条词表，对每条轨迹用仿真器跑 EPDMS 打分，筛出高分伪教师轨迹同时监督 proposal 分布，打破"单一人类轨迹"上限。*证据：Table 3 消融，加入蒸馏后 PDMS +1.8。*

3. **Momentum-Aware Trajectory Selection**（新方法细节）：选轨迹时引入帧间舒适度项，用上一帧选中轨迹作为先验，减少帧间抖动，显著提升舒适性指标。*证据：Table 3，comfort 指标明显改善。*

4. **Waypoint-Anchored Deformable Attention（WADA）**（改进）：在 proposal 生成阶段，attention 不看全图而只看预测 waypoint 周围的图像特征，计算量更低且更精准。

---

## Novelty 分类判断

**新方法（方法层面的改进组合）**：不是全新算法，但将 V-JEPA + 仿真器多模态蒸馏 + proposal-centric 规划组合成一套完整框架，是**首次**把这三者结合并在 NAVSIM 上达到 SOTA 的工作。各组件本身均有前身（V-JEPA来自Meta，proposal来自iPad，词表来自VADv2），但组合方式和仿真器蒸馏是本文贡献。

---

## 中文精读

### 1. 解决了什么问题

端到端自动驾驶有两个公认瓶颈：

**瓶颈 A：视频预训练没有明显收益**。生成式方法（VaVIM、Epona）重建像素，计算太贵；latent world model（LAW）没有随视频数据增加而提升，还可能 representation collapse（表征坍塌，即模型把所有输入映射到相似向量，信息丢失）。

**瓶颈 B：单条人类轨迹监督不足**。一个场景里只记录了司机实际走的一条路，但合理的走法有很多（直行/左转/减速让行）。现有 proposal 方法（iPad）虽然生成多条候选，但所有候选都只对齐那一条人类轨迹——相当于多条路都被拉向同一个答案，多样性消失（mode collapse）。

### 2. 新意在哪里

两个核心解法：
- **V-JEPA 预训练**：不重建像素，只在 latent 空间预测被 mask 的视频块表征。EMA（指数移动平均，Exponential Moving Average）副本作为 target encoder，防止两个 encoder 坍塌到同一表征。
- **Multimodal Trajectory Distillation（多模态轨迹蒸馏，MTD）**：从大规模数据中 k-means 聚类出 8192 条轨迹词表，对每个场景跑仿真器给所有词表轨迹打 EPDMS 分，选出多条高分轨迹作为伪教师，和人类轨迹一起监督模型。

### 3. Idea 具体落在哪里

**预训练阶段（离线，一次性）**：
- 208小时驾驶视频（CoVLA + DrivingDojo + OpenScene）
- ViT encoder 用 V-JEPA 目标训练：随机 mask 视频时空块，预测被 mask 块在 target encoder 中的表征，L1 loss
- 训完只保留 encoder，扔掉 predictor

**推理阶段（实时，每帧）**：

```
历史图像 → 预训练 ViT encoder → Image Features
自车状态 → Linear → Ego Status Features
      ↓ concat
Initial BEV Proposal Queries（可学习初始化）
      ↓ ×L 层 WADA（Waypoint-Anchored Deformable Attention）
N 条候选轨迹（Proposals）
      ↓
EPDMS + 帧间舒适度打分
      ↓
最终选中轨迹 → 执行
```

**训练时额外加入**：
- 仿真器生成的伪教师轨迹集合 𝒫ₜ
- Loss = min(人类轨迹误差) + min(伪教师轨迹误差)，两项加权求和

### 4. 最大难点在哪里

**难点 1：Representation Collapse 防止**。latent 预测的通病是两个 encoder 互相"作弊"坍塌成常数。V-JEPA 用 stop-gradient + EMA 解决，但工程上 EMA 动量系数、mask 策略（时间维度 mask 比例、空间块大小）都需要仔细调。

**难点 2：多模态词表不能覆盖所有场景**（out-of-vocabulary problem）。8192 条词表是从历史数据 k-means 来的，如果测试场景出现没见过的轨迹形状（极端弯道、倒车），词表里可能没有好的参考。论文通过 proposal-centric（在线生成候选）来缓解这个问题——词表只用于蒸馏教师，实际候选是实时生成的。

**难点 3：仿真器打分速度**。对每个训练场景跑 8192 条轨迹的 EPDMS，计算量巨大，论文用 rule-based 仿真（不是神经网络）保证速度。

### 5. 局限与相关性

**局限**：
- 只用前视单摄像头，没有多摄像头或 LiDAR 融合，感知受限
- 背景车建模为"会动的障碍物"，没有 intent prediction，不具备真正的交互式规划能力（Interactive Behavior Planning）
- 词表从历史数据聚类，在分布外场景（OOD）可能伪教师质量差
- NAVSIM 是 open-loop/pseudo-closed-loop，不等于真实闭环表现

**与本笔记库相关**：
- 和 WA-JEPA（笔记 #10）同为 JEPA 在驾驶上的应用，但切入点不同：WA-JEPA 同时建模视频+动作，Drive-JEPA 专注预训练表征质量 + 多模态规划
- V-JEPA 预训练 → 更好表征 → 更好轨迹，与 Qwen-Drive 的"VLM 当共享表征主干"属于同一哲学（好表征驱动一切），但路线不同（视频自监督 vs 语言模型监督）
- 多模态轨迹蒸馏和 DriveZero 的思路有共鸣：DriveZero 用 RL + 特权策略突破单条人类轨迹上限，Drive-JEPA 用仿真器词表打分

### 6. 是否能连 GitHub 复现

代码状态待确认（论文提及将发布）。NAVSIM benchmark 本身是公开的，V-JEPA 官方代码（Meta，facebookresearch/jepa）可用于预训练参考。

### 7. 与其他论文关键对比

> 💡 这张表是核心，看完每篇论文都可以回来对照。

| | DriveVLM | Qwen-Drive | DriveZero | **Drive-JEPA** |
|--|----------|-----------|--------------|----------------|
| **核心主张** | VLM推理→规划 | VLM表征→多任务 | RL策略→超人类 | **视频预训练→更好表征→更好规划** |
| **VLM角色** | 推理引擎（CoT） | 共享主干（表征） | 几乎不用 | **不用 VLM** |
| **学习来源** | 人类示范+语言标注 | 人类示范 | RL自主探索 | **人类示范+仿真器多模态轨迹** |
| **感知方式** | VLM直接看图 | VLM+BEV头 | 4个视觉大模型融合 | **V-JEPA预训练ViT** |
| **能回答问题吗** | 是 | 是 | 否 | 否 |
| **能超越人类吗** | 否 | 否 | 是 | 未明确对比 |
| **核心创新类型** | 新insight | 新insight | 组合创新 | 新方法组合 |

---

## 可略过背景

- §2.1 里的 ALVINN、PilotNet、Transfuser、UniAD、VAD——历史沿革，知道"端到端AD历史悠久"就够
- §2.2 里的 VaVIM、Epona——知道"像素重建太重"就够，不用细看
- §2.3 里的 VADv2、GoalFlow——知道"固定词表有覆盖问题"就够

---

## 假设 / 局限 / 质疑点

1. **假设其他车不会响应自车行为**：EPDMS 打分时把背景车当"按预测轨迹走的物体"，不建模反应，在高密度交互场景（城区并线、圆形交叉路口）可能失效
2. **假设仿真器打分可以代替真实世界反馈**：rule-based EPDMS 不能捕捉所有危险情况（如传感器噪声、异常驾驶行为）
3. **前视单摄局限**：遮挡物、侧方来车等只用前视处理不足
4. **NAVSIM ≠ 真实闭环**：93.3 PDMS 是 pseudo-closed-loop，与 Bench2Drive 等完整闭环差距可能大
5. **质疑：伪教师轨迹质量**：k-means 词表8192条是否足够覆盖长尾场景？OOD 时伪教师有没有可能引入噪声监督？

---

## 一个月后记住 5 件事

1. **V-JEPA 不重建像素，预测 latent 表征**——这让它比生成式方法轻，比普通 latent world model 稳（EMA 防 collapse）
2. **单独用 V-JEPA encoder + 轻量 decoder 就 89.0 PDMS，超过前人所有方法 3 分**——说明表征质量本身的价值
3. **8192 条轨迹词表从人类轨迹 k-means 来，仿真器对每条打 EPDMS 分，高分的当伪教师**——这是让模型学到"一个场景有多种合理走法"的关键
4. **Momentum-Aware 选轨迹：考虑上一帧选了哪条，保证帧间不抖**——工程细节但对乘坐体验很重要
5. **Drive-JEPA 解决的是"看懂场景"，不解决"和旁边的车博弈"**——交互规划是另一个研究方向（Motion Forecasting + Interactive Planning）

---

## English At-a-Glance

| | |
|---|---|
| **Problem** | (1) Video pretraining gives marginal gains due to pixel reconstruction cost or latent collapse; (2) single human trajectory per scene limits planning diversity |
| **Key Insight** | Predicting in latent space (not pixels) learns semantically meaningful structure; simulators can score 8192 trajectories per scene to provide multimodal supervision |
| **Method** | V-JEPA pretraining on 208h driving video → ViT encoder; Waypoint-Anchored Proposal Generation (×L layers deformable attention); Multimodal Trajectory Distillation from k-means vocabulary + EPDMS scoring; Momentum-Aware Selection |
| **Results** | 93.3 PDMS NAVSIM v1 (SOTA); 87.8 EPDMS NAVSIM v2 (SOTA); perception-free: 89.0 PDMS (+3 vs prior SOTA) |
| **Limitations** | Single front camera; no intent prediction of other agents; k-means vocabulary may miss OOD scenarios; NAVSIM ≠ real closed-loop |
| **Code** | Pending release (V-JEPA base: facebookresearch/jepa) |

---

## 本文问到的词

| 术语 | 解释 | 首次出现 |
|------|------|---------|
| JEPA / V-JEPA（联合嵌入预测架构）| 在 latent 空间预测被 mask 部分的表征，不重建像素；V-JEPA 是视频版本 | Drive-JEPA 架构核心 |
| EMA（指数移动平均，Exponential Moving Average）| Target encoder 是 online encoder 的慢速移动平均副本，stop-gradient，防止 collapse | V-JEPA 预训练 |
| Representation Collapse（表征坍塌）| 两个 encoder 输出坍塌为常数向量，失去区分能力；latent 预测的核心风险 | Related Work 2.2 |
| Proposal-Centric Planner（候选轨迹规划器）| 同时生成多条候选轨迹，打分后选最优；区别于直接回归单条轨迹 | Drive-JEPA 核心模块 |
| WADA（Waypoint-Anchored Deformable Attention）| Cross Attention 只看预测 waypoint 附近的图像区域，减少计算量且更精准 | Proposal Generation 模块 |
| Multimodal Trajectory Distillation（多模态轨迹蒸馏）| 用仿真器评分的多条轨迹当伪教师，同时监督 proposal 分布，打破单轨迹上限 | Drive-JEPA 核心贡献 |
| EPDMS（Extended PDM Score）| NAVSIM 评测指标：综合碰撞、合规、车道保持、舒适度等多项子分的加权分 | Benchmark / 打分模块 |
| Momentum-Aware Selection（动量感知选择）| 选轨迹时引入上一帧结果作为先验，减少帧间轨迹抖动，提升舒适度 | Drive-JEPA 选择模块 |
| Interactive Behavior Planning（交互式行为规划）| 规划时对其他车的行为建模为会响应自车决策的 agent，而非静态障碍物；Drive-JEPA 不做这个 | 阅读讨论 |
| NAVSIM | 自动驾驶 open-loop / pseudo-closed-loop 评测基准，用真实日志场景 + rule-based 评分 | 实验部分 |
| Planner（规划器）| 自动驾驶中负责"决定怎么开"的模块，输入场景理解，输出未来轨迹 waypoints | 阅读讨论 |
