# PPT两页修改建议

---

## 一、挑战文字砍半

### Slide 1 精度对齐 — 挑战区

**当前（啰嗦）→ 修改后（砍半）**

**① 跨精度logprob偏差**
- 当前：低精度Rollout + BF16训练产生系统性logprob偏差，同一token在两条路径上算出不同的log概率。这不是随机噪声，是有方向的系统性偏移，随训练步数持续累积。
- 改为：**FP8推理 + BF16训练 → 同一token算出不同logprob → 偏差随RL循环累积不收敛**

**② MoE跨引擎路由不一致**
- 当前：MoE模型在推理引擎（如SGLang）和训练框架（如Megatron）中对相同输入选择不同Expert路径。路由导致的MoE的训练长稳问题在LLM、VLM、低精度等场景下普遍存在。
- 改为：**同一输入在SGLang走Expert 2,5 → 在Megatron走Expert 1,7 → logprob不一致 → MoE RL训练崩溃**

**③ 批次大小导致的浮点非确定性**
- 当前：浮点非结合性（a+b+c ≠ a+c+b）导致输入一样但输出不一致。RL训练中batch size动态变化，logprob在每次计算时都可能有微小抖动，经千步累积为不可忽略的策略偏移。
- 改为：**浮点加法不满足结合律 → batch size变化改变reduction顺序 → 同一输入每次算出不同结果 → RL千步累积发散**

---

### Slide 2 低精训推 — 挑战区

**① Agent并发rollout的显存墙**
- 当前：Agent RL训练需要千条rollout同时并行生成，每条rollout还是多轮交互——KV Cache随工具调用轮次持续膨胀（单Agent会话可达数GB）。显存需求是普通serving的10-100倍。
- 改为：**千并发 × 多轮膨胀 → 单Agent会话KV Cache数GB → 显存需求是普通serving的10-100倍**

**② 量化偏差被RL训练循环放大**
- 当前：Agent RL的Episode长达数十到数百步，每步的偏差通过RL的"策略→数据→梯度→策略"闭环不断自我强化，几百步后可导致训练发散。这在单轮SFT/DPO中完全不存在。
- 改为：**量化偏差 × importance ratio放大 × RL闭环自我强化 → 几百步后训练发散（SFT/DPO无此问题）**

**③ Episode分钟级放大速度瓶颈**
- 当前：Agent RL的一个Episode涉及多轮工具调用，每轮都要等环境返回结果，单条轨迹耗时分钟级。Agent轨迹的token量是数学推理的10-100倍，每步decode要读取的KV Cache也更大
- 改为：**多轮工具调用 → 单条轨迹分钟级 → 推理慢1倍=训练慢1倍 → token量是数学推理的10-100倍**

---

## 二、批不变性算子更好的图

Thinking Machines Lab博客 (https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/) 中有多张高质量图，推荐以下两张：

### 推荐图1：Figure 16 — RL训练before/after对比

这是最直接的说服力图：展示"True On-Policy RL"（用batch-invariant算子）训练完全稳定，而标准方法要么collapse要么需要importance weighting修正。

**用途**：放在创新③"批不变性算子"的技术区域，一图说明"用了→稳定，不用→崩"。

### 推荐图2：Figure 7-9 — Kernel策略对比

Figure 7展示split-reduction策略如何因batch size变化而打破batch invariance（根因图），Figure 8展示batch-invariant matmul的data-parallel策略（解法图）。

**用途**：如果空间允许，放Figure 7（根因）在挑战③区域，Figure 8（解法）在创新③区域。

### 推荐图3："1000次完全相同"的实验结果

博客中提到：1000次相同prompt的completions，启用batch-invariant后全部bitwise identical。如果有这个实验的可视化图（如hash一致性图），可以用作"银弹级"证据。

**来源链接**：
- 博客：https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/
- 代码：https://github.com/thinking-machines-lab/batch_invariant_ops
- SGLang集成：https://www.lmsys.org/blog/2025-09-22-sglang-deterministic/

---

## 三、自研"MoE模型W4A4低精RL范式"加修饰语

### 当前标题
MoE模型W4A4低精RL范式

### 问题
太泛，没有说清楚"为什么是我们做而不是直接用QeRL/MBS"。

### 修改建议

**标题改为**：
> **面向MoE长程Agent RL的W4A4低精训推范式：TIS+AQN+MBS最优组合验证**

**副标题/说明改为**：
> 针对MoE稀疏架构 × Agent长程rollout的特定负载特征，将三项独立技术（NVIDIA的AQN探索增强、Meta的MBS精度补偿、LMSYS的TIS off-policy兜底）进行系统性组合与调优，找到低精度MoE Agent RL训练的最优配置。

### 修饰要点

1. **指定模型架构**："面向MoE"——MoE模型的稀疏激活使量化误差分布与Dense模型不同，Expert路由的动态性叠加量化抖动，问题更复杂
2. **指定workload**："长程Agent RL"——不是数学推理RL（几十步），是Agent RL（几百步多轮工具调用），importance weight累积衰减更严重
3. **组合最优解的意义**："TIS+AQN+MBS最优组合验证"——三项技术各自解决一个维度（TIS兜底安全性/AQN增强探索/MBS补偿精度），但它们的交互效应未被验证过：
   - AQN的噪声注入会不会影响MBS的scale精度？
   - TIS的ratio截断会不会抵消AQN的探索增强？
   - 三者的超参（AQN衰减率、MBS宏块大小、TIS截断阈值）在MoE Agent RL负载下的联合最优配置是什么？
   - **这个组合验证是我们独有的贡献——单项技术是NVIDIA/Meta/LMSYS的，组合最优解是我们在特定负载（MoE + Agent RL）上的系统级验证**

### PPT中的完整描述

> **面向MoE长程Agent RL的W4A4低精训推范式**
> 
> 单项技术已由业界巨头验证（AQN@NVIDIA、MBS@Meta、TIS@LMSYS），但三者的组合效应在MoE稀疏架构×Agent长程rollout负载上尚无验证。
> 
> 我们的贡献：在MoE模型上系统性探索TIS+AQN+MBS的联合配置空间，验证低精度下MoE Agent RL训练的可行性与最优超参组合。
> 
> [⬜ 预留图位：MoE实验结果——BF16基线 vs W4A4 naive vs W4A4+MBS vs W4A4+MBS+AQN vs W4A4+MBS+AQN+TIS]
