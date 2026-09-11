# 词汇表 Glossary

每次阅读论文时问到的术语，按字母排序汇总。格式：**术语（英文）**：解释 + 首次出现论文。

---

## R

**表征 / Representation**
神经网络内部对输入数据的高维数字编码。模型把图像/文本"压缩"成一串向量，向量中隐含了语义信息（场景里有什么、是什么含义），但不是人类可读的形式。"共享表征"指多个任务头（感知/规划/问答）从同一个主干模型的同一份向量出发，信息互通且只需推理一次。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**探针 / Probe**
将一个轻量模块挂在预训练模型的表征上，用来测试"表征里到底藏了多少某类信息"的实验手段。Qwen-Drive 把 BEV 感知头称为 probe，意指：如果挂上去能做好 3D 检测，就说明 VLM 表征中确实编码了 3D 场景结构。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）


## B

**BEV（Bird's Eye View，鸟瞰视图）**
将车辆周围多个摄像头的透视图像，通过几何变换和神经网络，投影拼合成一张俯视平面地图。地图中每个格子对应车周围固定大小的实际区域，标注障碍物/车道线/语义类别等。自动驾驶规划需要BEV坐标系来计算距离和轨迹，而摄像头原始图像是有畸变的透视视角，不能直接用于规划。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

## S

**SFT（Supervised Fine-Tuning，监督微调）**
用人类示范数据（"在这个场景下，正确答案是这条轨迹"）直接训练模型模仿。优点：稳定、易收敛；缺点：上限是人类示范的质量，模型无法超越示范者。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**RL（Reinforcement Learning，强化学习）**
不给标准答案，让模型自己采样多条轨迹，用评分函数（不碰障碍物、保持速度、不压线等）打分，模型根据奖励信号调整策略。优点：可以找到超越人类示范的最优解；缺点：训练不稳定，容易过拟合评测指标。Qwen-Drive RL版规划分数比SFT版高约2.5分（NAVSIM 90.7 vs 88.2）。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）


## V

**VLM（Vision-Language Model，视觉-语言模型）**
在大语言模型（LLM）前端加入视觉编码器，使模型能同时处理图像和文字输入的多模态大模型。图像先被视觉编码器压缩成向量，与文字向量拼合后送入语言模型，输出文字回答。代表模型：GPT-4V、Qwen3.5-VL、LLaVA等。Qwen-Drive 以 Qwen3.5-4B 作为 VLM 主干，外挂感知头和规划头，VLM 权重本身不改动。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

## D

**数据 Pipeline / 统一数据管道（Unified Data Pipeline）**
将来自不同来源、格式各异的多类数据集（感知标注、规划轨迹、VQA问答对）转换对齐到同一格式的预处理流程。Qwen-Drive 的统一步骤：① 坐标系对齐（统一以自车为中心）；② 轨迹重采样（统一为5秒/10Hz/(x,y,heading)）；③ VQA回答重标注（用大模型批量改写为格式和事实一致的统一风格）。统一后三类数据可以混合训练同一个模型。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）


**语义占用预测 / Semantic Occupancy Prediction**
将车辆周围三维空间划分为均匀小格子（voxel网格），对每个格子同时预测两件事：① 是否被物体占用（occupancy）；② 被什么类别的物体占用（semantic，如汽车/行人/建筑/植被等）。相比3D检测框，优势在于能表示任意形状的物体（树、墙、施工堆）且包含高度信息；相比BEV地图分割，优势在于是完整三维信息而非仅地面投影。Qwen-Drive 的 BEV 感知头三个输出之一。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）


**Explicit / Inspectable Interface（显式可检查接口）**
与神经网络内部的黑箱表征（高维向量）相对。指模型输出的是人类可直接读懂和验证的结构化结果（如3D检测框坐标、占用地图类别），而非抽象向量。Qwen-Drive 的 BEV 感知头输出即为此类接口——可以直接对照地图检查检测结果是否正确，不像内部表征完全不透明。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**Semantic（语义）**
AI领域中泛指"与含义/类别有关的高层信息"，对应低层的形状/像素/几何信息。加了"semantic"前缀意味着模型不只识别"有没有/长什么样"，还理解"是什么"。例：语义分割（每个像素是什么类别）vs 普通分割（只切区域不识别类别）。在深度学习中满天飞，因为现代模型的核心突破就是能把"含义"编码进向量。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）


**灾难性遗忘 / Catastrophic Forgetting**
神经网络在学习新任务时，旧任务的能力被"覆盖"而快速下降的现象。例：把一个通用VLM直接在驾驶数据上微调，它可能忘掉通用问答能力。常用缓解方法：分阶段训练、混合数据、冻结部分参数。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**分阶段训练 / Staged Training**
将训练过程拆成多个阶段，每阶段侧重不同数据或不同模块，避免任务之间互相干扰或产生灾难性遗忘。Qwen-Drive：先冻结VLM主干只训练外挂头，再混入通用数据联合微调。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**开环评测 / Open-loop Evaluation**
给模型看真实驾驶录像，预测轨迹后与人类实际轨迹对比。模型决策不改变场景，其他车辆按原始录像运动。便宜但不真实——现实中你的驾驶决策会影响其他车的反应。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**闭环评测 / Closed-loop Evaluation**
在仿真器里让模型真正"开车"，每个决策实时改变场景，其他车辆对自车做出响应。最接近真实驾驶，但需要高质量仿真器且计算成本高。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**伪闭环评测 / Pseudo-closed-loop Evaluation**
介于开环与闭环之间：用模型预测的轨迹重放场景，其他车辆轨迹固定不变，但检测碰撞/压线/违规等结果。NAVSIM 属于此类。比开环严格，比闭环便宜。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）


**视觉Token / Visual Token**
将图像通过视觉编码器"打碎"成一系列离散向量单元，格式与文字token相同，可直接拼入VLM的输入序列。VLM本身不能直接处理像素，视觉token是图像进入语言模型的"翻译格式"。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**自回归生成 / Autoregressive Generation**
VLM生成文字的方式：每次只预测下一个词，条件是所有输入（图像+文字prompt）加上已经生成的所有词。一个词一个词地串行生成，直到输出结束符。训练目标是最大化正确词的预测概率（即最小化交叉熵损失 L_ntp）。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**Frame-major / View-major 序列化**
多摄像头多时间帧输入的两种排列方式。帧优先（frame-major）：先排完一帧的所有摄像头再排下一帧，适合场景理解和VQA。视角优先（view-major）：先排完一个摄像头的所有时间帧再换下一个摄像头，让模型感知同一方向的时序变化，适合运动规划。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）


**Flow Matching（流匹配）**
一种生成模型训练范式，与Diffusion（扩散）类似：不直接预测输出，而是学习"如何把随机噪声逐步变成目标输出"的变换过程。Qwen-Drive的Planning Expert输入带噪声的轨迹，条件是VLM的场景理解（K,V），逐步去噪输出干净轨迹。优点：可以多次采样生成多条候选轨迹（best-of-N），再选最优。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**K, V（Keys and Values，键值对）**
Transformer注意力机制中存储信息的两个矩阵。Query（查询）用来"问问题"，Key用来"匹配问题"，Value是"匹配到之后取出的内容"。Qwen-Drive把VLM内部的K,V缓存后传给Planning Expert，相当于把VLM对场景的理解"记忆"直接传递给规划模块，而不需要规划模块重新看一遍图像。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）

**Voxel（体素）**
三维空间中的最小单元，是二维"像素（pixel）"在3D中的对应概念。把三维空间切成均匀的小立方体，每个立方体就是一个voxel。Qwen-Drive的View Transform把2D摄像头图像通过几何变换投影到3D voxel网格，再送入BEV感知头。
→ 首次问到：*Qwen-Drive-1.0*（2026-09-10）


**PPO（Proximal Policy Optimization，近端策略优化）**
强化学习中最常用的策略优化算法。核心思想：限制每次策略更新的幅度（用clip截断），防止新策略与旧策略差距过大导致训练崩溃。"Proximal（近端）"即指新旧策略要保持"近邻"关系。相比早期RL算法（如REINFORCE）更稳定，是ChatGPT、驾驶RL等大规模RL训练的标准选择。DriveZero用PPO在闭环仿真中训练DriveRL教师策略。
→ 首次问到：*DriveZero*（2026-09-10）


## DriveZero (arXiv 2609.06055, Xiaomi EV, 2026)

**Privileged Policy（特权策略）**: 在仿真环境中能看到完整真实状态（如所有车辆的精确速度、加速度、意图）的策略，相当于"作弊"——现实中传感器感知到的信息远不如它完整。用作 RL 训练的教师，学生 policy（只有真实传感器输入）通过模仿它学习，再经 RL 进一步提升。首次出现：DriveZero

**Mixed-agent Interactive World（混合智能体交互世界）**: 仿真场景中，背景车辆不再只是回放记录（log replay），而是混合了：① 日志回放（真实轨迹）、② 规则式 agent（IDM 等）、③ 学到的 policy。使场景更"活"，自我车辆的行为会真正影响背景车的反应，避免 reward hacking（找到一个只在静态场景有效的捷径）。首次出现：DriveZero

**Knowledge Distillation（知识蒸馏）**: 让小模型（学生）学习大模型/复杂模型（教师）输出的分布，而不是直接学人工标注的 ground truth。DriveZero 中：DriveVFM 用 4 个大型视觉基础模型当教师，训练一个紧凑的 perception 骨干；DriveRL 的学生 policy 蒸馏特权 policy 的行为。首次出现：DriveZero

**Augmented Goals（增广目标/辅助目标）**: 在 RL 训练中，主奖励函数之外额外加入的辅助奖励项，帮助探索和稳定训练。DriveZero 引入了感知辅助奖励（如检测到障碍物给 bonus），让 agent 在稀疏主奖励下学得更快、行为更安全。首次出现：DriveZero

**Imitation Learning（模仿学习，IL）**: 直接从人类驾驶演示数据中学习，本质是监督学习——给定观测，预测人类动作。等价于 SFT。问题：分布偏移（distribution shift）——测试时遇到训练数据未覆盖的状态。DriveZero 的核心论点之一：纯 IL 上限是人类水平，RL 可突破这个上限（95.3 PDMS > 人类 94.8）。首次出现：DriveZero（对比使用）

## Drive-JEPA (arXiv 2601.22032, 2026)

**V-JEPA / JEPA（Video Joint-Embedding Predictive Architecture，视频联合嵌入预测架构）**: LeCun 提出的学习范式：在 latent 空间预测被 mask 掉的视频块的表征，不重建原始像素。核心哲学：模型不应浪费能力预测像素噪声（树叶位置、云的纹理），只需要学习可预测的语义结构（谁在动、往哪走）。EMA 副本防 representation collapse。Drive-JEPA 把它用于驾驶视频预训练。首次出现：Drive-JEPA

**Representation Collapse（表征坍塌）**: Latent 预测模型的通病——两个 encoder 互相"作弊"，把所有输入映射到相似的常数向量，loss 降到0但什么也没学。V-JEPA 用 stop-gradient + EMA 解决：target encoder 是 online encoder 的慢速移动平均，不参与梯度更新。首次出现：Drive-JEPA

**EMA（指数移动平均，Exponential Moving Average）**: 参数更新方式：θ_target = m·θ_target + (1-m)·θ_online，m 接近1（如0.996）。Target encoder 慢慢跟随 online encoder 变化，不直接用反向传播更新，提供稳定的学习目标。首次出现：Drive-JEPA

**Proposal-Centric Planner（候选轨迹规划器）**: 同时生成 N 条候选轨迹（proposals），每条独立打分，选最优一条执行。区别于直接回归单条轨迹：能表达"左转"和"直行"两种合理选择并各自评估。Drive-JEPA 用 WADA 生成候选，EPDMS 打分。首次出现：Drive-JEPA

**Multimodal Trajectory Distillation（多模态轨迹蒸馏）**: 从人类轨迹数据 k-means 聚类出8192条轨迹词表，对每个训练场景把所有词表轨迹放进仿真器跑 EPDMS 打分，筛出高分的多条作为伪教师轨迹，和真实人类轨迹一起监督训练。打破"每个场景只有一条正确答案"的限制。首次出现：Drive-JEPA

**EPDMS（Extended PDM Score）**: NAVSIM v2 的综合评测指标，综合多项子分：碰撞、交通规则合规、车道保持、行驶方向、舒适度等。Drive-JEPA 用它给仿真候选轨迹打分，也用它评测最终性能（87.8 EPDMS SOTA）。首次出现：Drive-JEPA

**Momentum-Aware Trajectory Selection（动量感知轨迹选择）**: 选最终轨迹时，不只看安全分，还加入"帧间舒适度"项——参考上一帧选中的轨迹，惩罚和上帧差距太大的选择，避免车辆轨迹逐帧跳变导致乘客不舒适。首次出现：Drive-JEPA

**Interactive Behavior Planning（交互式行为规划）**: 规划时将其他车辆建模为"会响应自车行为的 agent"（而非沿固定轨迹走的障碍物），预测其意图，考虑多车博弈。Drive-JEPA 不做这个——它把背景车当障碍物，用 EPDMS 静态检查碰撞。真正做这个需要 Motion Forecasting 模块。首次出现：Drive-JEPA 阅读讨论

**NAVSIM**: 自动驾驶 open-loop / pseudo-closed-loop 评测基准（nuPlan/nuScenes 数据），不需要完整仿真器，用真实日志场景 + rule-based 评分。v1 指标为 PDMS，v2 扩展为 EPDMS。Drive-JEPA：v1 93.3 PDMS，v2 87.8 EPDMS（均为 SOTA）。首次出现：Drive-JEPA
