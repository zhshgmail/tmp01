# P4-C 低精RL训推 — 页面设计（按Zheng-v0.1.pptx布局）

> 从技术升级方向C（推理加速）中独立出来，聚焦低精度RL训推的技术创新

---

## 标题区（顶部全宽）

**主标题**：低精RL训推：量化不是精度妥协，而是探索增强

**副标题**：挑战❶长序列推理开销 + 挑战❻训推数值不一致 → 低精度训推的三项关键创新

---

## 核心挑战区（上方，3个并排卡片）

**左侧标签**：核心挑战

### ① 推理显存瓶颈 — P3挑战❶方面b
70B模型全精度（BF16）推理需约140GB显存，单卡放不下。Agent RL训练中90%算力在rollout推理，推理效率直接决定训练数据供给速率。不量化则rollout并发度受限，训练数据饥饿。

**关键数据**：QeRL（NVIDIA，ICLR'26）首次在单张H100 80GB上实现32B模型的RL训练，靠的就是NVFP4量化。

### ② 量化精度损失放大 — P3挑战❻方面a
传统认知：量化损失精度。但在RL场景，量化噪声的影响和预训练/SFT完全不同——RL的importance ratio对logprob敏感，量化引入的数值偏差会被ratio放大。MXFP4（OCP标准）与NVFP4（NVIDIA私有）之间有约10%精度差距。

**关键数据**：MBS+OAS将MXFP4与NVFP4的精度差距从~10%降至<1%（Meta, arXiv:2603.08713）。

### ③ 量化噪声 vs 探索的意外关系 — P3挑战❹方面a
QeRL发现了一个反直觉现象：在RL训练中，适度的量化噪声反而**增强探索**——量化后的模型不仅不比BF16差，还能超越16-bit LoRA甚至逼近全参数微调。但噪声需要动态调控，否则后期会干扰exploitation。

**关键数据**：QeRL的NVFP4+LoRA在数学推理任务上超越vanilla BF16 LoRA训练，且接近全参数微调性能（NVIDIA, ICLR'26）。

---

## 关键创新区（中间，3个技术方案）

**左侧标签**：关键创新 — 从三个层面让低精度在RL中可用且有益

### 创新① AQN（Adaptive Quantization Noise）— 自适应量化噪声

**一句话**：将量化噪声从"不得不忍受的副作用"转化为"可调控的探索增强工具"——早期高噪声促进探索，后期低噪声保障收敛。

**核心机制**：
- 在量化权重中注入channel-wise高斯噪声，噪声幅度通过指数衰减调度器动态控制
- 噪声合并到LayerNorm参数中，将加性噪声转化为乘性噪声（对RL探索更有效）
- 早期训练：高噪声→增强探索多样性，防止策略过早坍缩
- 后期训练：噪声衰减→稳定exploitation，保障收敛

**图示建议**（参考QeRL论文）：
- QeRL论文中的**探索-收敛对比图**：X轴训练步数，Y轴reward/accuracy，BF16 LoRA vs NVFP4+AQN。展示AQN在早期因探索增强而更快上升，后期因噪声衰减而稳定收敛

**来源**：**NVIDIA**（QeRL, ICLR'26）
**链接**：[arXiv:2510.11696](https://arxiv.org/abs/2510.11696) | [GitHub NVlabs/QeRL](https://github.com/NVlabs/QeRL) | [NVIDIA Research](https://research.nvidia.com/labs/eai/publication/qerl/)

---

### 创新② MBS（Macro Block Scaling）— 宏块缩放

**一句话**：纯软件方案将开放标准MXFP4的精度差距从10%压到<1%——不改硬件，让MXFP4逼近NVIDIA私有NVFP4的精度。

**核心机制**：
- **问题**：MXFP4（OCP开放标准，32元素共享1个scale）精度不如NVFP4（NVIDIA私有，16元素独立scale），差距约10%
- **MBS方案**：在MXFP4的微块（32元素）之上增加一层宏块缩放（更粗粒度的高精度scale），更好保留异常值
- **OAS（Overflow-Aware Scaling）**：配合MBS，增加动态范围减少溢出误差
- **效果**：MXFP4+MBS+OAS精度差距降至<1%，GEMM开销仅6.2%，硬件面积节省12%

**图示建议**（参考MBS论文）：
- 论文中的**MXFP4 vs NVFP4精度对比图**：柱状图展示不同模型/任务上，原始MXFP4 vs MXFP4+MBS vs NVFP4的精度，MBS将差距从10%压到<1%

**来源**：**Meta**（AI and Systems Co-Design团队）
**链接**：[arXiv:2603.08713](https://arxiv.org/abs/2603.08713)

---

### 创新③ TIS+AQN+MBS综合方案 — MoE模型低精RL训推

**一句话**：在MoE模型上综合使用TIS（off-policy修正）+ AQN（探索增强）+ MBS（精度补偿），实现低精度下的稳定Agent RL训练。

**核心机制**：
- **TIS（Truncated Importance Sampling）**：截断极端importance ratio，防止低精度引入的数值偏差导致梯度爆炸
- **AQN**：动态调控量化噪声，将精度损失转化为探索收益
- **MBS**：在MXFP4格式下补偿精度，让开放标准量化逼近私有格式
- **三者协同**：TIS兜底安全性 + AQN增强探索 + MBS补偿精度 = 低精度MoE RL训练的完整方案

**图示位置**：
> **⬜ 留白区域：我们的MoE实验结果图**
> 
> （此处预留一张图的位置，展示TIS+AQN+MBS在MoE模型上的综合实验效果）
> 
> 图表内容建议：X轴训练步数，Y轴reward/pass rate，对比：
> - BF16基线（全精度）
> - MXFP4 naive（直接量化，精度崩）
> - MXFP4 + MBS（精度恢复）
> - MXFP4 + MBS + AQN（探索增强）
> - MXFP4 + MBS + AQN + TIS（完整方案，稳定且超越基线）

**来源**：**我们的工作**（基于NVIDIA QeRL + Meta MBS + LMSYS TIS的综合方案）

---

## 底部总结条（全宽）

**趋势判断**：低精度RL训推正从"省显存的权宜之计"演变为"增强探索的主动策略"。NVIDIA用QeRL证明量化噪声增强RL探索（ICLR'26），Meta用MBS让开放标准MXFP4逼近私有NVFP4精度。**我们的贡献**：在MoE模型上首次验证TIS+AQN+MBS三技术协同，实现低精度下稳定且高效的Agent RL训练。

---

## 页面布局ASCII参考（对标Zheng-v0.1.pptx）

```
┌──────────────────────────────────────────────────────────────────────┐
│ 低精RL训推：量化不是精度妥协，而是探索增强                              │
│ 挑战❶推理开销 + 挑战❻数值不一致 → 三项关键创新                       │
├──────┬─────────────────┬─────────────────┬───────────────────────────┤
│      │ ① 推理显存瓶颈    │ ② 量化精度损失放大│ ③ 量化噪声vs探索          │
│ 核心 │ 70B BF16=140GB   │ MXFP4 vs NVFP4  │ 反直觉：适度噪声          │
│ 挑战 │ 单卡放不下        │ 精度差距~10%     │ 增强RL探索               │
│      │ rollout并发受限   │ ratio放大偏差    │ QeRL超越BF16 LoRA        │
├──────┼─────────────────┼─────────────────┼───────────────────────────┤
│      │                 │                 │                           │
│ 关键 │ ① AQN           │ ② MBS           │ ③ TIS+AQN+MBS            │
│ 创新 │ 自适应量化噪声    │ 宏块缩放         │ MoE综合方案              │
│      │                 │                 │                           │
│ 背书 │ [QeRL训练曲线]   │ [MXFP4精度对比]  │ [⬜ 我们的MoE实验图]      │
│      │ AQN早期探索增强   │ MBS将差距        │ TIS兜底+AQN探索          │
│      │ 后期噪声衰减收敛  │ 从10%压到<1%     │ +MBS精度=完整方案         │
│      │ 32B单H100可跑    │ GEMM开销仅6.2%   │                           │
│      │ NVIDIA ICLR'26  │ Meta 2026       │ 我们的工作                │
├──────┴─────────────────┴─────────────────┴───────────────────────────┤
│ 趋势：低精RL从"省显存权宜"→"增强探索主动策略"。NVIDIA证明量化噪声增强    │
│ 探索，Meta让MXFP4逼近NVFP4。我们：MoE上TIS+AQN+MBS三技术协同首验。    │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Side Notes：技术点详细参考

**AQN（Adaptive Quantization Noise）**
- 全称：自适应量化噪声
- 来源：NVIDIA（QeRL论文，ICLR'26）
- 核心：在NVFP4量化权重中注入动态调控的高斯噪声，指数衰减调度
- 关键发现：量化噪声在RL中增强探索（与SFT中的负面影响相反）
- 效果：32B模型单H100可RL训练，超越BF16 LoRA
- 论文：[arXiv:2510.11696](https://arxiv.org/abs/2510.11696) | [GitHub](https://github.com/NVlabs/QeRL)

**MBS（Macro Block Scaling）**
- 全称：宏块缩放
- 来源：Meta（AI and Systems Co-Design团队，Geonhwa Jeong等）
- 核心：在MXFP4微块之上增加宏块级高精度scale，配合OAS增加动态范围
- 关键发现：纯软件方案将MXFP4精度差距从~10%压到<1%
- 效果：GEMM开销6.2%，硬件面积节省12%
- 论文：[arXiv:2603.08713](https://arxiv.org/abs/2603.08713)

**TIS（Truncated Importance Sampling）**
- 全称：截断重要性采样
- 来源：LMSYS/Miles框架 + 蚂蚁Ling Team（IcePop中的MIS变体）
- 核心：截断极端importance ratio到安全范围，防止off-policy梯度爆炸
- 在低精场景的作用：量化引入的数值偏差会产生极端ratio，TIS做安全兜底
- 参考：[verl Rollout Correction文档](https://verl.readthedocs.io/en/latest/algo/rollout_corr.html) | [HuggingFace TRL Issue #4493](https://github.com/huggingface/trl/issues/4493)

**QeRL框架**
- 全称：Quantization-enhanced Reinforcement Learning
- 来源：NVIDIA Efficient AI Lab（ICLR'26）
- 整体框架：NVFP4量化 + LoRA微调 + AQN探索增强
- 核心洞察："量化增强RL"——与SFT中量化损害性能的认知相反
- 论文：[arXiv:2510.11696](https://arxiv.org/abs/2510.11696)
