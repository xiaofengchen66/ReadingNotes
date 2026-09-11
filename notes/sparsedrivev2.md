# SparseDriveV2: Scaling Trajectory Vocabulary for End-to-End Autonomous Driving

- **Venue / year:** arXiv 2026(2603.29163,2026-03-31 提交)· ECCV 2026 录用
- **Authors:** Wenchao Sun, Xuewu Lin, Keyu Chen, Zixiang Pei, Xiang Li, Yining Shi, Sifa Zheng
- **Link:** https://arxiv.org/abs/2603.29163(直接用 arXiv API 核实过)
- **Code:** https://github.com/swc-17/SparseDriveV2(已开源,含权重)
- **Input / 输入:** 多摄像头图像（环视）
- **Output / 输出:** 最优轨迹 τ* = argmax s(τ, o_t)，由factorized词表组合而成

---

## TL;DR

**问题**：Scoring-based planning依赖预定义的轨迹词表，但覆盖完整动作空间需要词表极大，内存和计算代价不可接受。  
**Gap**：现有方法要么词表太小（coarse discretization，候选轨迹之间间隔太大，找不到好的），要么动态生成候选但引入复杂性。  
**Novelty**：把轨迹分解为路径（path）和速度剖面（velocity profile）两个独立维度，各建100条词表，组合出10,000条等效覆盖，只存200个条目。  
**核心洞察**：路径（几何形状）和速度（时间演化）正交——同一条路可以跑不同速度，同一速度可以走不同形状——所以可以独立建词表后任意组合。  
**方法**：Factorized vocabulary + 联合打分（路径分 × 速度分）+ 密集词表scaling study证明越密越好且未饱和。  
**结果**：NAVSIM leaderboard SOTA，打败包括动态proposal生成方法在内的对比系统。  
**意义**：证明了static vocabulary只要足够密，不比dynamic proposals差；而factorization是让词表变密的关键工程手段。

---

## 贡献（按重要性排序）

### ① Factorized Trajectory Vocabulary（最重要）
把monolithic轨迹 τ 分解为 **path p**（几何形状，按空间间隔∆s采样）和 **velocity profile v**（速度序列，按时间间隔∆t采样）。分别建路径词表（100条）和速度词表（100条），组合覆盖10,000种轨迹。

- **为什么这是正确的分解？** 路径只关心"去哪里"（几何），速度剖面只关心"多快"（时间），两者互相独立，任意组合都是合法轨迹。
- **证据**：消融实验显示factorized比monolithic在相同存储量下显著更优。

### ② Vocabulary Scaling Study
系统实验表明：词表越密，PDMS越高，且在测试的范围内**没有饱和迹象**。这反驳了"词表够用就行"的直觉，说明持续scaling词表是有效的研究方向。

### ③ 打败Dynamic Proposal方法
对比Drive-JEPA等用动态生成候选轨迹的方法，SparseDriveV2的static vocabulary方案在足够密时性能持平或更好——说明"静态词表 vs 动态生成"的差距主要来自词表太稀疏，而不是static方法本身的局限。

---

## Novelty分类

**新方法（改进组合）**：factorized vocabulary本身的想法并非完全首创（路径/速度解耦在运动规划领域有先例），但在scoring-based end-to-end AD的框架内系统应用并做scaling study，是本文贡献。

---

## 中文精读

### 1. 解决了什么问题

End-to-end自动驾驶里的scoring-based planning需要预定义词表T = {τ_i}，N越大覆盖越全，但内存和计算正比于N。如果想覆盖所有合理的驾驶动作（不同方向 × 不同速度 × 不同曲率），N会需要达到数万甚至更多，这在实际系统里不可行。

已有的解决方案分两派：
- **稀疏静态词表**：N小（几百），计算快，但coarse discretization导致候选轨迹之间间隔太大，选出的"最好"轨迹也可能离真正最优很远。
- **动态生成候选**：比如Drive-JEPA的WADA + 8192-trajectory k-means + EPDMS scoring，覆盖更全，但系统更复杂，需要多阶段训练。

SparseDriveV2的问题：**能不能让静态词表变得足够密，同时保持计算可行？**

### 2. 新意在哪里

核心洞察是：轨迹 τ 是两个**正交维度**的乘积——

```
τ = path × velocity profile
```

- **path p = {(x_i, y_i)}**：按空间间隔∆s采样，只编码几何形状（弯不弯、弯多少、往哪个方向），没有任何时间信息。
- **velocity profile v = {v_t}**：按时间间隔∆t采样，只编码每个时间步的平均速度，不管空间形状。

因为这两个维度完全独立，可以：
1. 建100条路径（覆盖从直行到左/右大转弯的所有几何形状）
2. 建100条速度剖面（覆盖从急刹到加速的所有速度变化模式）
3. 任意组合 → 100 × 100 = **10,000条有效轨迹**

存储只需要200个条目，覆盖达到10,000条。

### 3. Idea具体落在哪里

**公式(2)**：τ* = argmax_{τ∈T} s(τ, o_t)

评分函数 s 在factorized框架下分解为路径分和速度分的联合评分。模型对每条路径和每条速度剖面分别打分，然后取最优组合。

**词表构建**：
- 路径词表：从大量真实驾驶数据里用k-means聚类提取几何形状原型
- 速度词表：同理，从数据里聚类提取速度模式
- 两者独立训练，推理时枚举所有组合打分

**Scoring model**：接受ego-centric的BEV特征 + 候选轨迹，输出每条路径/速度的分数。

### 4. 为什么Static Vocabulary能打败Dynamic Proposals？

这是本文最有意思的实验发现。

直觉上Dynamic proposal（比如Drive-JEPA的WADA生成的proposals）应该更好，因为它们是"针对当前场景定制生成"的。但SparseDriveV2发现：当词表足够密时，静态词表里总能找到一条足够接近"定制方案"的轨迹，性能差距消失。

这背后的原因是：驾驶动作空间虽然连续，但**实际有效的驾驶动作本身就很稀疏**——大多数时候你要么直行要么轻微转弯要么刹车，极端动作（超大角度急转、极端加减速）出现频率极低。只要词表覆盖了这个"实际有效子空间"，稠密静态词表就够了。

### 5. 局限 / 假设 / 质疑点

- **词表未饱和**：scaling study说词表越大越好但测试范围有限，不知道什么时候收益递减。
- **路径和速度真的完全独立吗？** 在极端情况下（比如高速急转），两者有物理耦合（离心力限制）。词表组合出来的某些极端组合可能物理上不可行，需要后处理过滤。
- **词表构建依赖数据分布**：如果测试场景的动作分布和训练数据不同（比如某些少见的复杂路口），词表覆盖可能不够。
- **评估局限**：主要在NAVSIM（pseudo-closed-loop）上评估，closed-loop表现未知。

### 6. 和其他论文的关系

| 对比维度 | SparseDriveV2 | Drive-JEPA | WA-JEPA |
|---------|---------------|------------|---------|
| 轨迹候选来源 | Static factorized vocabulary (100×100) | Dynamic WADA proposals + 8192静态词表+EPDMS筛选 | Dynamic flow matching生成 |
| 词表大小 | 10,000（等效） | 8192 | N/A（连续生成） |
| 核心贡献 | Factorization让static vocabulary变密 | WADA + multimodal distillation | V-JEPA → world-action joint prediction |
| 评估平台 | NAVSIM | NAVSIM v1+v2 | NAVSIM |
| Interactive behavior | 否 | 否 | 否（预测但不交互） |

---

## 可略过背景

- 具体的BEV特征提取架构细节（标准操作）
- NAVSIM数据集的详细介绍（已在Drive-JEPA笔记里覆盖）
- 与早期SparseDrive v1的详细对比（除非想了解演进历史）

---

## 假设 / 局限 / 质疑点

1. **路径×速度独立性假设**：在物理极限附近会被违反，高速大转弯组合可能不可行
2. **Static = Dynamic前提是词表够密**：如果计算预算不够大，结论反转
3. **NAVSIM为主要评估**：pseudo-closed-loop，不等于真实驾驶或closed-loop仿真
4. **词表scaling无饱和**：论文范围内如此，实际上限未知

---

## 一个月后记住5件事

1. **Factorize轨迹为path × velocity**，100×100=10,000覆盖，只存200条目
2. **Path按空间采样（几何），velocity按时间采样（速度）**，正交所以可独立组合
3. **Static vocabulary只要够密，可以媲美dynamic proposals**——差距主要来自词表稀疏
4. **Scaling study：词表越密越好，测试范围内未饱和**——还有提升空间
5. **评估在NAVSIM（pseudo-closed-loop）**，和Drive-JEPA、WA-JEPA在同一榜单竞争

---

## 本文问到的词

| 术语 | 含义 |
|------|------|
| **Scoring-based planning** | 预定义候选轨迹词表，用打分器选最优轨迹的规划范式 |
| **Trajectory vocabulary / 轨迹词表** | 预先聚类的N条候选轨迹集合 T = {τ_i} |
| **Coarse discretization / 粗离散化** | 词表太稀疏，候选之间间隔太大，最优选择也远离真正最优 |
| **Factorized vocabulary / 分解词表** | 把轨迹分解为path和velocity两个独立维度，各建词表后组合 |
| **Path / 路径** | 按空间间隔∆s采样的几何形状序列，只管"去哪里"不管"多快" |
| **Velocity profile / 速度剖面** | 按时间间隔∆t采样的速度序列，只管"多快"不管"去哪里" |
| **Combinatorial coverage** | 组合爆炸用于好的方向：m条path × n条velocity = m×n条覆盖 |
| **Static vocabulary** | 推理前预定义好的固定词表（对比：dynamic proposal是推理时按场景生成） |
| **Dynamic proposal** | 推理时针对当前场景动态生成候选轨迹（如Drive-JEPA的WADA） |
| **EPDMS** | Extended PDM Score，NAVSIM v2评估指标，综合碰撞/合规/舒适度 |
