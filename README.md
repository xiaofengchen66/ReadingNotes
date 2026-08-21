# ReadingNotes

个人论文精读笔记库。每篇论文一个 Markdown 文件,固定结构:

- 中文精读:解决了什么问题 / 新意在哪里 / idea 具体落在哪里 / 最大难点在哪里 / 局限与相关性 / 是否能连 GitHub 复现
  - 关键概念第一次出现时,中文后面用英文括号注释(方便之后用英文讲解/组会汇报)
  - 关键原文摘录(中英对照):5 句左右最关键的原文逐字引用 + 中文翻译 + 批注,并尽量把论点和论文里的具体表格/公式数字对应起来
- 英文一目了然总结:一张大尺寸结构化 SVG 海报(问题/新意/架构图/难点/证据/代码可用性)+ 图解读(逐框讲解)+ 中英对照表格

## Index

| # | Paper | Venue / Year | Notes | Code/GitHub |
|---|-------|--------------|-------|--------------|
| 1 | DriveVLM | CoRL 2024 | [notes/drivevlm.md](notes/drivevlm.md) | Not released (project page only) |
| 2 | DriveMLM | Tech report 2023 (v3 2025) | [notes/drivemlm.md](notes/drivemlm.md) | Repo exists, placeholder only |
| 3 | LoRA | ICLR 2022 | [notes/lora.md](notes/lora.md) | Released & usable (microsoft/LoRA) |
| 4 | Adaptive Mixtures of Local Experts | Neural Computation 1991 | [notes/mixture-of-local-experts.md](notes/mixture-of-local-experts.md) | N/A (pre-GitHub) |
| 5 | Alpamayo-R1 | arXiv 2025 (v2 2026), NVIDIA | [notes/alpamayo-r1.md](notes/alpamayo-r1.md) | Weights + inference code released |
| 6 | An Open Approach to Autonomous Vehicles (Autoware) | IEEE Micro 2015 | [notes/autoware.md](notes/autoware.md) | Open & maintained 10+ years |

## 结构

```text
notes/          每篇论文的精读笔记 (.md)
notes/img/      对应的英文总结海报 (.svg)
```
