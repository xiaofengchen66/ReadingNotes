# Beyond Rule-Based Agents: Active Markov Games for Realistic Multi-Agent Interaction in Autonomous Driving

- **Venue / year:** CVPR 2026
- **Authors:** Yuan Gui, Hongchen Luo, Jiao Wang, Liqi Qu
- **Link:** https://openaccess.thecvf.com/content/CVPR2026/html/Gui_Beyond_Rule-Based_Agents_Active_Markov_Games_for_Realistic_Multi-Agent_Interaction_CVPR_2026_paper.html ([本地PDF副本](pdf/active-markov-games.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

把驾驶环境建模成主动马尔可夫博弈(AMG)：扩展传统马尔可夫博弈，让状态转移和奖励依赖于智能体不断演化的策略，捕捉自车和周围智能体之间的交互动态和策略耦合。多智能体协同进化训练机制：联合优化自车策略和一个多样化的对手策略池。CARLA无信号灯路口和长尾场景仿真中成功率接近完美(98%)，显著超过PPO/DDPG/IPPO等基线。

---

## 跟本repo已有内容的关系

多智能体博弈建模，跟MoE路由关系不大，是"交互式仿真训练"这条技术路线，收录因为核心类目相关。
