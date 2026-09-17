# E2E 自动驾驶研究方向笔记

> **整理自**：2026-09-17 论文阅读与研究讨论
> **状态**：思路整理，待文献精查后确认 novelty

---

## 一、E2E 自动驾驶最新论文地图（2024–2026）

### VLM / LLM 驱动路线（主流趋势）

| 论文 | 机构 | 特点 |
|------|------|------|
| **EMMA** (2024) | Waymo | Gemini 多模态，把轨迹当 token 文本生成，纯 camera |
| **Senna** (2024) | HUST | Vicuna-7B → discrete action → E2E，camera-only |
| **DriveLM** (2024) | OpenDriveLab/上海AI | 图-文 QA 驱动规划，scene graph reasoning |
| **SOLVE** (CVPR 2025) | — | Language-Vision + E2E 网络协同 |
| **AppleVLM** (arXiv 2602.04256) | — | VLM 感知增强 + 规划 |
| **OpenDriveVLA** (AAAI 2026) | — | 大型 Vision-Language-Action 模型端到端 |
| **AutoDrive-P³** (arXiv 2603.28116) | — | 感知-预测-规划链式思维 + 强化微调 |

### 非 VLM 规划头创新路线

| 论文 | 特点 |
|------|------|
| **ARTEMIS** (arXiv 2504.19580) | Autoregressive + MoE，camera+LiDAR，NAVSIM 87.0 PDMS |
| **VECTOR-Drive** (arXiv 2605.08830) | VLM + Trajectory Expert Routing，类似 ARTEMIS 思路 |
| **Fast-dDrive** (arXiv 2605.23163) | Block-Diffusion VLM，高效扩散规划 |
| **GuideFlow** (CVPR 2026) | Constraint-Guided Flow Matching，扩散规划新范式 |
| **PRIX** (arXiv 2507.17596) | 从原始像素端到端规划 |

### 强化学习 / 闭环路线

| 论文 | 特点 |
|------|------|
| **READ** (ICLR 2025) | 安全增强 RL + E2E |
| **Centaur** (2025) | Test-time training，闭环鲁棒性 |
| **Post-Training in E2E AD** (arXiv 2607.08072) | 统一视角看 E2E 的后训练方法 |

### 综述 / 大范式

| 论文 | 特点 |
|------|------|
| **The Era of End-to-End Autonomy** (arXiv 2603.16050) | 从规则到大驾驶模型的范式转变综述 |
| **Latent Chain-of-Thought World Modeling** (arXiv 2512.10226) | 世界模型 + 链式思维 |
| **Drive by Hindsight and Foresight** (arXiv 2609.08217) | 分层记忆 + 工具推理 |

### 2025–2026 主要趋势总结

1. **VLA（Vision-Language-Action）成主流**：不只是 VLM 输出文字，而是直接输出动作/轨迹
2. **Post-training / RLHF 上车**：RL Fine-tuning 让规划更符合人类偏好（安全、舒适）
3. **Diffusion + Flow Matching 做规划头**：GuideFlow（CVPR 2026）是代表
4. **闭环评测取代开环**：NAVSIM、nuPlan 等 reactive simulation 成标准
5. **Expert Routing / MoE 思路扩散**：ARTEMIS 之后，VECTOR-Drive 也做专家路由

**建议精读路线**：UniAD → EMMA → Senna → ARTEMIS → OpenDriveVLA → GuideFlow（CVPR 2026）

---

## 二、OpenDriveVLA vs UniAD 对比分析

### 架构对应关系

| 模块 | UniAD | OpenDriveVLA | 说明 |
|------|-------|--------------|------|
| BEV Encoder | BEVFormer | BEV Encoder | 提取空间特征 |
| Agent 表示 | TrackFormer | TrackQFormer | 表示交通参与者 |
| Map 表示 | MapFormer | MapQFormer | 表示道路地图 |
| 规划器 | Planning Head | VLA Model | 生成驾驶轨迹 |

### 关键区别

- **UniAD**：Planning-oriented，各模块串联优化，"以规划为终点"
- **OpenDriveVLA**：Language-conditioned，把场景/地图/agent 转为 visual token，接受自然语言指令

接口语义本质不同：
- UniAD 的 TrackFormer 输出 → **结构化 query 向量**（tensor 传递）
- OpenDriveVLA 的 TrackQFormer 输出 → **语言对齐的 visual token**（token 传递给 VLM）

### 关键问题（需查原文确认）

1. BEV Encoder 是否复用已有模型（看 Implementation Details → Backbone）
2. VLA 是否端到端梯度更新（看 Training，找 "frozen" / "joint training"）
3. Token 接口如何实现（看 Architecture → BEV features 到 LLM input 的那张图）

> **注意**：2025-2026 的 **BEV Encoder + QFormer → 大模型** 三段式已成默认范式。
> 真正的 novelty 在于 QFormer 的设计选择和 VLA 的训练策略，而不是整体结构像不像。

---

## 三、研究方向构思

### 方向一：World Model + VLA + MoE（适合探索新架构）

**论文题目候选**：World-Model-Guided Mixture-of-Experts for Autonomous Driving

核心想法：传统 VLA 根据当前场景直接预测轨迹。若模型先预测未来 1–3 秒场景变化，再用 Router 根据当前 + 未来环境选择专家，是否能改善规划质量？

```
Camera / LiDAR / Ego State
        ↓
    World Model（预测未来 1–3 秒）
        ↓
  Lightweight Router（当前+未来场景）
        ↓
Urban Expert | Highway Expert | Adverse Weather Expert
        ↓
  Trajectory Planning
```

研究问题：
- 未来场景预测是否能改善专家选择？
- World Model 预测误差会不会导致 Router 选错？
- 能否只训练 Router 和少量适配参数，冻结其他专家？

**难点**：World Model 训练成本高；需证明预测未来确实改善了驾驶表现
**定位**：博士第二、三年，作为方向三的自然延伸

---

### 方向二：LiDAR-Guided VLA Fine-Tuning（侧重多模态感知与鲁棒性）

**论文题目候选**：LiDAR-Guided Adaptation of Vision-Language-Action Models for Robust Driving

核心想法：目前 VLA 依赖 Camera，但夜间/雨雪/遮挡环境视觉信息不可靠。用 LiDAR 几何信息在训练阶段指导 Camera-only VLA。

```
训练时：Camera + LiDAR → Teacher Model → Geometry-aware Distillation → Camera-only VLA
推理时：Camera-only VLA（不需要 LiDAR）
```

蒸馏目标选择（三选一）：

| 方式 | 说明 | 难度 |
|------|------|------|
| Feature-level 蒸馏 | Student Camera BEV feature 靠近 Teacher Camera+LiDAR BEV feature | 中 |
| Geometry-aware 蒸馏 | Teacher 预测深度图/占用图作为 pseudo label | 高，故事最好讲 |
| Trajectory-level 蒸馏 | Teacher 轨迹分布作为 soft label，KL divergence | 低，几何信息最稀疏 |

**更强的 Novelty 角度（关键洞察）**：

> "在模型执行 '在雨天左转避开行人' 这类**语言条件指令**时，
> LiDAR 几何信息的引入是否改善了指令执行成功率？"

这把 LiDAR 蒸馏 + VLA 语言理解绑在一起，是任何 BEV 蒸馏论文都没有回答过的。

**VLA 训练样本格式**：

```
输入：
  Camera     → RGB images
  LiDAR      → Point cloud
  Ego State  → Speed / Pose
  Command    → "Turn left"（语言指令）

Ground Truth：
  Future trajectory: (x₁,y₁), ..., (x₆,y₆)

还需要：传感器标定参数 + 统一时间戳 + 坐标系定义
```

**注意**：nuScenes 本身无自然语言指令，需使用 **DriveLM-nuScenes**。

**数据策略**：
- 先用 nuScenes/DriveLM-nuScenes 做可行性验证（1–2 个月）
- 再用 CARLA adverse weather 做核心实验（1–2 个月）
- CARLA 是**证明核心假设的必要条件**（nuScenes 雨天数据极少）

---

### 方向三：Risk-Aware VLA with Lightweight Expert Routing（推荐第一篇论文）

**论文题目候选**：Risk-Aware Expert Routing for Efficient Autonomous Driving

核心想法：不按 Urban/Highway/Weather 手工划分专家，而是让 Router 根据**驾驶风险**和**模型不确定性**动态选择专家。

```
BEV Tokens + Ego State + Command
              ↓
       Risk-Aware Router
   风险特征 + 场景特征 + 不确定性
              ↓
    ┌─────────┴──────────┐
Normal Expert        Safety Expert
    └─────────┬──────────┘
              ↓
  Trajectory + Safety Evaluation
```

具体研究内容：
1. 正常场景使用轻量专家，复杂场景调用专门规划专家
2. Router 利用预测碰撞风险、轨迹不确定性等特征进行选择
3. 冻结专家，只训练 Router 或 Adapter
4. 比较性能、延迟、显存和参数量

**与 ARTEMIS 的差异**：ARTEMIS 用手工设计的 Comfort/Speed/Command intent 路由；
本方向把 routing signal 换成**模型预测的 risk**——这是核心差异化。

**Router feature 设计**（不建议 scalar risk score + 阈值）：

```python
routing_features = concat([
    TTC,              # Time-to-collision，基于预测轨迹估算
    scene_complexity, # BEV 中 agent 数量、遮挡率
    uncertainty,      # ensemble variance 或 MC dropout
])
router = LightweightMLP(routing_features)
```

**第一个实验**：
1. 用 OpenDriveVLA 或 Senna 作为冻结 backbone
2. 加 Normal Head（轻量 MLP）+ Safety Head（保守 trajectory refiner）
3. 训练 Risk Router：输入 BEV tokens + TTC 估算 + agent 数量
4. 在 NAVSIM 上跑，重点关注 long-tail scenario 的 PDMS

**投稿目标**：三个月出结果 → IROS 2027 或 ECCV 2026 workshop

---

## 四、三个方向定位总结

| 方向 | 核心研究问题 | 主要挑战 | 建议时机 |
|------|------------|---------|---------|
| World Model + MoE | 未来预测能否改善专家路由？ | 计算成本、预测误差 | 博士二三年 |
| LiDAR-Guided VLA | 几何监督能否改善语言条件驾驶？ | 跨模态对齐、数据配对 | 博士一二年 |
| **Risk-Aware MoE** | 能否以低计算成本实现可靠专家选择？ | 风险定义、路由稳定性 | **第一篇论文** |

**演化路线**：方向三 → 加入 LiDAR 几何作为 risk 特征（方向二的元素）→ 加入 World Model 预测作为 routing signal（方向一），故事完整。

---

## 五、关键 Gap 分析：Planning-Oriented Distillation

### 现有工作的局限

BEV 蒸馏论文（BEV-LGKD、BEVDepth、UniDistill）的隐含假设：
> "几何越准确，下游越好"

这在**规划任务**中不一定成立——路灯的精确 3D 位置对轨迹规划几乎无价值，但行人速度方向极其关键。

### 真正的研究问题

> **哪些几何信息对 VLA 规划决策是 planning-critical 的？
> 如何只蒸馏这部分而不是全部 BEV 几何？**

### 具体技术：Planning-Relevance Scoring

```
LiDAR BEV (Teacher)
        ↓
Planning-Relevance Scoring
（用规划 loss 的梯度反向标记哪些 BEV region 对决策有影响，类似 GradCAM）
        ↓
Selective Distillation
（只蒸馏 high-relevance region 的 feature）
        ↓
Camera-only VLA (Student)
```

**Story**：不是用 LiDAR 让 Camera 看得更准，而是用 LiDAR 告诉 Camera
"什么几何信息对驾驶决策重要"——**以规划为导向的蒸馏，而非以感知为导向的蒸馏**。

### 文献核查（待确认 gap 是否成立）

搜索关键词：
- `"planning-oriented" + "knowledge distillation" + "autonomous driving"`
- `"LiDAR-to-camera" + "VLA" OR "vision-language-action"`
- `"modality distillation" + "end-to-end driving"`

---

## 六、数据集选择建议

| 数据集 | 优点 | 缺点 | 适用场景 |
|--------|------|------|---------|
| **nuScenes + DriveLM** | VLA 论文主流 benchmark，有语言标注 | 雨天数据少 | 方法可行性验证（首选） |
| Waymo Open | 大规模，真实 | VLA baseline 稀少，对比困难 | — |
| KITTI | 入门简单 | 太老，无语言 | 快速原型 |
| Argoverse 2 | 适合轨迹预测 | Sensor + Motion 两个子集不通用 | 运动预测 |
| **CARLA** | 可控天气，重复生成 | 仿真与真实的 domain gap | Adverse weather 核心实验 |

---

## 七、待阅读论文

- [ ] **VECTOR-Drive** (arXiv 2605.08830)：与方向三最接近的 baseline，必读
- [ ] **OpenDriveVLA** (AAAI 2026)：理解 Token 接口设计
- [ ] **DriveLM**：了解 nuScenes 语言标注格式
- [ ] **GuideFlow** (CVPR 2026)：Flow Matching 做规划头的新范式
- [ ] **Post-Training in E2E AD** (arXiv 2607.08072)：RL fine-tuning 综述

## 八、待办事项

- [ ] 文献核查：planning-oriented distillation + VLA 是否已有直接相关工作
- [ ] 联系导师讨论方向三的可行性和计算资源
- [ ] 检查 DriveLM-nuScenes 语言标注格式，确认可否直接用于 VLA 训练
