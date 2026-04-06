# P4-I 算子精度对齐 — 页面设计（按Zheng-v0.1.pptx布局）

---

## 标题区（顶部全宽）

**主标题**：算子精度对齐：消除训推数值分叉，让RL训练信号可信赖

**副标题**：挑战❻训推数值不一致 → 技术升级方向I → 三项关键创新

---

## 核心挑战区（上方，3个并排卡片）

**左侧标签**：核心挑战

### ① 跨精度logprob偏差
FP8 rollout + BF16训练产生系统性logprob偏差。PPO的importance ratio要求分子分母数值可比，但rollout用FP8、训练用BF16，同一token在两条路径上算出不同的log概率。偏差不是随机噪声，是有方向的系统性偏移，随训练步数持续累积。

**关键数据**：无FP8 E2E时，训推KL初始即显著偏大且随训练持续增长；有FP8 E2E后KL全程维持近零（LMSYS实测）。

### ② MoE跨引擎路由不一致
MoE模型在推理引擎（如SGLang）和训练框架（如Megatron）中对相同输入选择不同Expert路径。路由分叉→logprob不一致→importance ratio失真→RL训练崩溃。Qwen3-30B-A3B实测：无R3时前几百步即发生RL collapse。

**关键数据**：R3 Routing Replay额外开销<5%，但将训推KL散度降低近50%，完全防止RL训练崩溃。

### ③ 跨硬件算子数值差异
CANN（昇腾）与CUDA（NVIDIA）的LayerNorm、AllReduce等算子在第5-6位有效数字存在差异。预训练能容忍（最终收敛到同一loss），但RL的importance ratio放大机制让微小差异经千步累积为可观测的策略偏移。

**关键数据**（L8实测910B）：LayerNorm精度差异导致reward曲线300步后与GPU偏移2-3个百分点，根因定位花了一周。

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

### 创新③ 数值一致性监控与校准

**一句话**：将训推logprob的KL散度作为健康指标实时监控——超阈值报警，从"事后排查"转为"事前预防"。

**核心机制**：
- **实时KL监控**：每N步计算训练侧和推理侧对同一batch的logprob KL散度，作为数值健康指标
- **Kurtosis早期预警**：监控QKV激活分布的峰度（Kurtosis），峰度突变预示loss spike
- **Flash Attention版本锁定**：不同版本FA的数值行为有微妙差异，锁定版本避免隐性漂移
- **算子精度档案**：逐算子建立CANN vs CUDA的数值diff基线（目前已完成5个RL关键路径算子）

**图示建议**：
画一张**监控仪表盘示意图**：
- 上方：KL散度实时曲线（绿色正常区 → 黄色预警区 → 红色报警区）
- 中间：算子精度热力图（LayerNorm🔴高差异 / Softmax🟡中差异 / Embedding🟢低差异 / AllReduce🔴高差异）
- 下方：一行状态条"Flash Attention v2.5.8 锁定 ✓ / CANN 8.0.RC3 基线 ✓"
来源：Scaling FP8 Training (ICLR'25) / 工程实践

**来源**：多来源工程实践 + ICLR'25学术论文
**链接**：[Scaling FP8 Training](https://proceedings.iclr.cc/paper_files/paper/2025/file/f48b5133e89854a9e97cc22a6db83f25-Paper-Conference.pdf)

---

## 底部总结条（全宽）

**趋势判断**：业界正从"训练完了再查精度问题"转向"训练全程数值可观测+自动干预"。FP8 E2E已成为高效RL训练的标配，MoE路由对齐是MoE模型做RL的前置条件。**对NPU的启示**：算子精度对齐不是"nice-to-have"——在RL场景下，它是训练能否收敛的生死线。CANN vs CUDA的逐算子精度档案是华为独有的差异化资产。

---

## 页面布局ASCII参考（对标Zheng-v0.1.pptx）

```
┌──────────────────────────────────────────────────────────────────────┐
│ 算子精度对齐：消除训推数值分叉，让RL训练信号可信赖                      │
│ 挑战❻训推数值不一致 → 技术升级方向I → 三项关键创新                    │
├──────┬─────────────────┬─────────────────┬───────────────────────────┤
│      │ ① 跨精度偏差     │ ② MoE路由不一致  │ ③ 跨硬件算子差异           │
│ 核心 │ FP8推理+BF16训练 │ SGLang vs Megatron│ CANN vs CUDA             │
│ 挑战 │ →logprob系统偏移 │ →Expert路径分叉   │ →第5-6位有效数字差异      │
│      │ KL持续增长       │ →前几百步RL crash │ →reward 300步后偏移2-3%   │
├──────┼─────────────────┼─────────────────┼───────────────────────────┤
│      │                 │                 │                           │
│ 关键 │ ① FP8 E2E       │ ② R3 Routing    │ ③ 数值监控与校准           │
│ 创新 │    Pipeline      │    Replay       │                           │
│      │                 │                 │                           │
│ 用统 │ [KL散度对比图]   │ [路由一致性图]   │ [监控仪表盘示意图]         │
│ 一精 │ 红线:无E2E KL↑   │ 左:不一致→crash │ KL曲线+算子热力图          │
│ 度消 │ 绿线:有E2E KL≈0  │ 右:R3→稳定     │ +版本锁定状态              │
│ 除偏 │                 │ 开销<5%         │                           │
│ 差   │ LMSYS/SGLang    │ PKU/Alibaba     │ 工程实践/ICLR'25          │
├──────┴─────────────────┴─────────────────┴───────────────────────────┤
│ 趋势：从"训练完查精度"→"全程数值可观测+自动干预"。FP8 E2E已成标配，     │
│ MoE路由对齐是MoE RL前置条件。NPU启示：CANN精度档案是华为差异化资产。    │
└──────────────────────────────────────────────────────────────────────┘
```
