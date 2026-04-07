# P4-I 算子精度对齐 — 页面设计（按Zheng-v0.1.pptx布局）

---

## 标题区（顶部全宽）

**主标题**：算子精度对齐：消除训推数值分叉，让RL训练信号可信赖

**副标题**：挑战❻训推数值不一致 → 技术升级方向I → 三项关键创新

---

## 核心挑战区（上方，3个并排卡片）

**左侧标签**：核心挑战

### ① 跨精度logprob偏差 — P3挑战❻方面a
FP8 rollout + BF16训练产生系统性logprob偏差。PPO的importance ratio要求分子分母数值可比，但rollout用FP8、训练用BF16，同一token在两条路径上算出不同的log概率。偏差不是随机噪声，是有方向的系统性偏移，随训练步数持续累积。

**关键数据**：无FP8 E2E时，训推KL初始即显著偏大且随训练持续增长；有FP8 E2E后KL全程维持近零（LMSYS实测）。

### ② MoE跨引擎路由不一致 — P3挑战❻方面b
MoE模型在推理引擎（如SGLang）和训练框架（如Megatron）中对相同输入选择不同Expert路径。路由分叉→logprob不一致→importance ratio失真→RL训练崩溃。Qwen3-30B-A3B实测：无R3时前几百步即发生RL collapse。

**关键数据**：R3 Routing Replay额外开销<5%，但将训推KL散度降低近50%，完全防止RL训练崩溃。

### ③ 批次大小导致的浮点非确定性 — P3挑战❻方面c
同一模型、同一输入，不同batch size下reduction kernel（LayerNorm、Softmax、MatMul）的分块顺序不同，浮点非结合性（a+b+c ≠ a+c+b）导致输出不一致。RL训练中batch size动态变化（不同rollout长度导致padding不同），logprob在每次计算时都可能有微小抖动，经千步累积为不可忽略的策略偏移。

**关键数据**：Thinking Machines Lab实测——标准推理引擎中，仅改变batch size就导致同一prompt的输出完全不同（非温度采样导致，是浮点运算顺序导致）。SGLang集成batch-invariant算子后实现100%可复现RL训练。

---

## 关键创新区（中间，3个技术方案）

**左侧标签**：关键创新 — 从三个层面消除数值分叉

### 创新① FP8 E2E Pipeline（精度统一）

**一句话**：端到端FP8 RL流水线——采样与训练统一精度，从根源消除跨精度KL偏差。

**核心机制**：
- 将FP8从"仅推理加速"扩展到完整RL管线（前向/反向/优化器/rollout推理）
- 支持FP32 Scaling Factor确保数值稳定
- 训练前强制对齐验证（训推logprob diff < 阈值才启动训练）

**图示建议**（参考论文图）：
画一张**KL散度随训练步数变化的对比图**：
- X轴：训练步数（0-1000）
- Y轴：训推KL散度
- 红线（无FP8 E2E）：KL从~0.02起步持续上升到~0.15
- 绿线（有FP8 E2E）：KL全程维持在~0.001附近
- 标注转折点："300步后开始显著分叉"
来源：LMSYS Blog 2025.11 / arXiv:2601.18150

**来源**：LMSYS+SGLang协作 / InfiXAI / Microsoft FP8-LM
**链接**：[arXiv:2601.18150](https://arxiv.org/abs/2601.18150) | [LMSYS Blog](https://www.lmsys.org/blog/2025-11-25-fp8-rl/)

---

### 创新② R3 Routing Replay（MoE路由对齐）

**一句话**：推理时记录每个token的Expert Routing Mask，训练时Replay相同路由——用<5%开销换取MoE RL训练稳定性。

**核心机制**：
- 推理阶段：在每个MoE层记录Router的Top-K expert选择和gating weights
- 训练阶段：不重新计算路由，直接Replay推理时的路由决策
- 效果：相同输入在训练和推理中走完全相同的Expert路径
- R3在R2（基础版）基础上增加自适应Replay，适合大off-policy漂移场景

**图示建议**（参考论文图）：
画一张**MoE路由一致性对比图**：
- 左图"无R3"：同一token在SGLang走Expert 2,5,7 → Megatron走Expert 1,3,8 → logprob不一致 → RL collapse
- 右图"有R3"：SGLang记录路由mask [2,5,7] → Megatron replay [2,5,7] → logprob一致 → 训练稳定
- 底部数据：Qwen3-30B-A3B，无R3前几百步collapse；有R3全程稳定，开销<5%
来源：arXiv:2510.11370

**来源**：北京大学 / 阿里巴巴
**链接**：[arXiv:2510.11370](https://arxiv.org/abs/2510.11370)

---

### 创新③ Batch Invariant Ops（批不变性算子）

**一句话**：替换所有reduction kernel为批不变实现——无论batch size如何变化，同一输入始终产生bit-wise相同的输出，从根源消除RL训练的浮点非确定性。

**核心机制**：
- **问题根源**：LLM推理的非确定性不来自随机采样，而来自浮点非结合性——不同batch size导致reduction kernel的分块方式不同，加法顺序变化→结果变化。RL训练中batch size随rollout长度动态变化，每次logprob计算都有微小抖动
- **Batch Invariant Kernels**：重写LayerNorm、RMSNorm、Softmax、MatMul等reduction kernel，固定分块大小和reduction顺序，使输出与batch size无关
- **SGLang集成**：与CUDA Graphs、Radix Cache、Chunked Prefill兼容，性能开销仅34.35%（对比Thinking Machines原始实现61.5%开销）
- **RL训练验证**：SGLang + slime协作实现100%可复现RL训练——同一checkpoint两次运行产生bit-wise相同的reward曲线

**图示建议**（参考Thinking Machines Lab博客图）：
画一张**批不变性原理对比图**：
- 左图"标准推理"：Batch=[A,B,C]和Batch=[A,B]中，A的输出不同（reduction分块不同→浮点累加顺序不同→结果不同）
- 右图"Batch Invariant"：无论batch怎么变，A的输出始终相同（固定分块大小→固定累加顺序→结果确定）
- 底部数据：SGLang集成后性能开销34.35%，实现100%可复现RL训练
来源：Thinking Machines Lab "Defeating Nondeterminism in LLM Inference" (2025.9) / SGLang Blog (2025.9)

**来源**：Thinking Machines Lab（新加坡，估值$10亿+独角兽）+ SGLang/LMSYS（UC Berkeley）
**链接**：[Thinking Machines Lab Blog](https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/) | [SGLang Deterministic Inference Blog](https://www.lmsys.org/blog/2025-09-22-sglang-deterministic/) | [SGLang GitHub Issue #10278](https://github.com/sgl-project/sglang/issues/10278)

---

## 底部总结条（全宽）

**趋势判断**：业界正从三个层面系统性消除RL训推数值分叉——NVIDIA用FP8 E2E统一精度、小米用R3对齐MoE路由、Thinking Machines用Batch Invariant Ops消除浮点非确定性。三者互补而非替代：FP8 E2E解决跨精度偏差，R3解决跨引擎路由分叉，Batch Invariant解决跨batch数值抖动。**对NPU的启示**：算子精度对齐不是"nice-to-have"——在RL场景下，它是训练能否收敛的生死线。昇腾上实现batch-invariant算子是差异化机会。

---

## 页面布局ASCII参考（对标Zheng-v0.1.pptx）

```
┌──────────────────────────────────────────────────────────────────────┐
│ 算子精度对齐：消除训推数值分叉，让RL训练信号可信赖                      │
│ 挑战❻训推数值不一致 → 技术升级方向I → 三项关键创新                    │
├──────┬─────────────────┬─────────────────┬───────────────────────────┤
│      │ ① 跨精度偏差     │ ② MoE路由不一致  │ ③ 浮点非确定性             │
│ 核心 │ FP8推理+BF16训练 │ SGLang vs Megatron│ batch size变→输出变      │
│ 挑战 │ →logprob系统偏移 │ →Expert路径分叉   │ →浮点非结合性导致         │
│      │ KL持续增长       │ →前几百步RL crash │ →RL训练不可复现           │
├──────┼─────────────────┼─────────────────┼───────────────────────────┤
│      │                 │                 │                           │
│ 关键 │ ① FP8 E2E       │ ② R3 Routing    │ ③ Batch Invariant        │
│ 创新 │    Pipeline      │    Replay       │    Ops                   │
│      │                 │                 │                           │
│ 背书 │ [KL散度对比图]   │ [路由一致性图]   │ [批不变性原理对比图]       │
│      │ 红线:无E2E KL↑   │ 左:不一致→crash │ 左:标准→batch变输出变     │
│      │ 绿线:有E2E KL≈0  │ 右:R3→稳定     │ 右:BI→batch变输出不变     │
│      │                 │ 开销<5%         │ 开销34%,100%可复现RL      │
│      │ NVIDIA/LMSYS    │ 小米/北大        │ Thinking Machines/SGLang  │
├──────┴─────────────────┴─────────────────┴───────────────────────────┤
│ 趋势：三个层面系统性消除训推数值分叉——NVIDIA统一精度、小米对齐路由、      │
│ Thinking Machines消除浮点非确定性。三者互补。NPU启示：batch-invariant    │
│ 算子是昇腾差异化机会。                                                  │
└──────────────────────────────────────────────────────────────────────┘
```
