---
title: 对世界模型的阶段性总结
publishDate: 2026-9-21 19:18:00
description: 'WM'
tags:
  - WM
language: '中文'
---

世界模型是什么，他和MBRL有什么区别，我倾向于怎么分类，不同部分的研究问题，以及未来感兴趣的研究方向。

[WM_report](/WM/WM_report.pdf)

最近刷到很多帖子唱衰WM，说WM没有前景，而确实现在WM的概念五花八门，没有形成统一的概念。而在我看来，WM就是一个把表示学习结合到状态预测的学习结构。目前更火热的讨论范围好像是集中在生成式的世界模型上，而我目前感兴趣的好像还是更偏向于agent学习的方向上，那我是不是逃过一劫口诛笔伐[doge]

世界模型的争议不是没有愿意，消耗的算力资源大于LLM，而成果并不显著，sim2real差距明显等等问题，都会让人觉得没有实用价值。而我现在也在对统计学引导的AI风向产生怀疑，目前AI能解决千禧难题，但是无法完全学会分辨一张图里有几根手指，后面这个问题，以及持续学习等问题应该是由同样的范式解决么？

---

update：进展到世界模型的持续学习：

[WM_Continual_Learning](/WM/WM_Continual_Learning.pdf)

**Problem setup and evaluation**

- **WM-CL compositional benchmark:** studies whether earlier tasks help with later combinations while measuring forgetting. https://arxiv.org/abs/2609.22055

**Route A: learning reusable knowledge from scratch**

- **DRAGO:** combines synthetic rehearsal with exploration of relevant past states to preserve dynamics knowledge for later goals. https://proceedings.mlr.press/v267/fu25f.html
- **LLWM:** combines task-conditioned latent dynamics, generative replay, and behavior learning for continual visual control. https://arxiv.org/abs/2303.06572
- **ARROW:** combines short-term and long-term replay buffers to balance current learning with coverage of earlier experience. https://arxiv.org/abs/2603.11395
- The compositional benchmark also evaluates real replay, EWC, PackNet, and progressive modular learning.

**Route B: reusing and adapting pretrained knowledge**

- **DINO-WM:** reuses frozen DINOv2 spatial features and learns action-conditioned dynamics from trajectories. https://proceedings.mlr.press/v267/zhou25t.html
- **Newt:** builds a generalist world-model agent through demonstration pretraining and multitask online learning, then studies adaptation to new tasks. https://www.nicklashansen.com/NewtWM/
- **SPREAD:** freezes a pretrained world model and updates LoRA adapters to adapt to environmental changes. https://doi.org/10.1109/LRA.2026.3688061
- **JEPA-TTT:** freezes the encoder and reward head while updating the dynamics predictor through self-supervised learning and replay across episodes. https://arxiv.org/abs/2610.00722

**Knowledge access and prediction failures**

- **Pretrained VLAs are Surprisingly Resistant to Forgetting — ICML 2026 Oral:** large pretrained VLAs with small replay can show zero forgetting in some settings, and old skills can recover quickly using old-task data. https://proceedings.mlr.press/v306/liu26bu.html
- **The World Model Remembers, the Actor Forgets:** finds retained world-model information alongside actor forgetting in a DreamerV3 setting with preserved replay. https://arxiv.org/abs/2607.19749
- **Hallucination in World Models is Predictable and Preventable:** connects prediction failures to data-coverage gaps and uses diagnostic signals to guide targeted data collection and adaptation. https://arxiv.org/abs/2606.27326