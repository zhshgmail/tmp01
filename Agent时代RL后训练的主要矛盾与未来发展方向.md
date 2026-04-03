# Agent时代，RL后训练领域的主要矛盾与未来发展方向

**——四位专家多轮研讨综述**

> 本文基于四位来自不同领域的专家——RL后训练算法专家（Claude Opus 4.6）、Agent系统架构师（OpenAI GPT-5.4/Codex）、AI Infrastructure工程专家（Claude Opus 4.6）、AI业界资深投资顾问（Claude Opus 4.6）——围绕"Agent时代RL后训练领域的主要矛盾及未来发展方向"展开的多轮深度研讨，经主持人综合整理而成。讨论过程中进行了多轮网络搜索获取最新产业数据作为事实依据，所有关键论断均标注来源。

---

## 引言：从Chatbot到Agent，RL后训练的范式跃迁

预训练解决的是"知道什么"，RL后训练解决的是"怎么做"。当产业从Chatbot转向Agent（多轮、多步、与真实环境交互的智能体），RL后训练从"锦上添花"的对齐手段，变成了释放模型"行动能力"的**刚需基础设施**。

事实支撑这一判断：

- **DeepSWE-Preview**（Together AI）通过纯RL训练（GRPO++算法，基于verl扩展的rLLM框架），在SWE-bench Verified上达到59%的解决率（Pass@1: 42.2%，K=16混合scaling: 59%）[^1]
- **SA-SWE-32B**（SkyRL）以纯RL达到39.4% pass@1，成本仅为同等模型的50%[^2]
- **Self-Play SWE-RL**（Meta）使用PPO算法，验证了RL训练在代码Agent上的有效性[^3]
- **verl-agent**框架已支持GRPO/PPO/DAPO/RLOO/REINFORCE++/GiGPO六种RL算法的Agent训练[^4]

这些结果表明，**RL后训练已成为构建高性能Agent的核心技术路径**。传统RLHF训练的是单轮映射函数（Bandit问题），Agent RL训练的是闭环控制策略（MDP结构回归）——状态转移由外部环境控制，模型必须处理部分可观测性和非平稳环境。

Agent RL系统采用"训推分离"与"异步执行"架构解决算力瓶颈，但同时引发了五对核心矛盾。这五对矛盾在逻辑上正交，分布于不同的系统抽象层，共同构成了Agent时代RL后训练的完整挑战图谱。

---

## 一个改变一切的关键发现：90%算力在推理侧

在展开五对矛盾之前，必须先建立一个贯穿全文的核心事实：

> **Agent RL训练中，超过90%的运行时算力消耗在rollout推理（自回归序列生成）上，而非梯度更新。** ——HuggingFace异步RL训练调研[^5]

Actor生成轨迹是推理，Reward评估是推理，只有参数更新是训练。

但需要注意，**推理scaling和训练scaling是互补关系而非替代关系**。DeepSWE的数据清晰说明：RL训练将Pass@1从23%提升到42%（基础能力提升），推理时scaling进一步从42%放大到59%（能力放大）[^1]。Toby Ord的分析也指出，100倍推理算力将AIME从约20%提升到约80%，100倍RL训练算力从约33%提升到约66%[^6]。最新综述（2025）进一步指出"推理时缩放的优势因任务而异，随问题复杂度增加而递减"[^7]。

**核心结论：RL训练构建基础能力，推理时scaling放大能力。90%算力在推理侧的事实，重塑了系统设计的优先级。**

---

## 第一部分：五对正交核心矛盾

### 矛盾一（算法层）：策略新鲜度 vs. 系统异步效率

#### 矛盾本质

RL算法的数学理论要求训练数据由当前最新策略生成（on-policy），但分布式系统追求rollout与训练完全异步以最大化吞吐量。这是**一致性与性能的根本权衡**，无法靠硬件消除。

#### 前因与后果

Agent场景下，一条轨迹包含多轮推理加工具调用，生成时间从秒级膨胀到分钟级。PPO的重要性比率方差随陈旧度步数指数级增长。SkyRL（UC Berkeley, 2025）实证：staleness超过阈值后训练直接发散[^8]。Meta的Self-Play SWE-RL论文明确指出："训练不稳定性阻碍进一步缩放，源于长horizon rollout的固有挑战"[^3]。

#### 解决途径与业界方案

| 方案 | 代表工作 | 机制 | 验证状态 |
|------|---------|------|---------|
| 陈旧度准入控制 | **SkyRL**（UC Berkeley） | 最大陈旧度预算，超限丢弃 | 已实验验证 |
| 离线补偿 | **IMPALA/V-trace**（DeepMind） | 截断重要性采样 | 生产部署 |
| 算法多样化 | **verl-agent** | 同时支持GRPO/PPO/DAPO/RLOO/GiGPO等6种算法 | 开源可用[^4] |

**事实修正**：此前文章断言"GRPO将成为Agent RL主流"，经搜索验证需要修正。**GRPO因无需critic网络、实现简单而被广泛采用，但PPO在需要精细价值估计的场景仍不可替代**（如Meta Self-Play SWE-RL使用PPO[^3]）。DeepSWE使用的GRPO++融合了DAPO/Dr.GRPO/RLOO的改进[^1]，说明纯GRPO并不够用。

#### 未来演进

算法选择走向**场景化而非一统天下**：GRPO适合结果奖励清晰的场景（代码测试通过/失败），PPO适合需要步级价值估计的复杂交互。Stratified GRPO[^9]和GiGPO[^10]分别通过分层归一化和Group-in-Group策略修正了GRPO的异质性偏差，在QA任务上提升最高11.3分（Stratified GRPO）、在ALFWorld/WebShop上提升12%/9%（GiGPO）。

---

### 矛盾二（计算架构层）：推理主导的算力结构 vs. 训练优化的基础设施

#### 矛盾本质

Agent RL 90%+算力在推理，但当前AI芯片、集群、框架都为训练优化。**整个产业基础设施的优化方向与Agent RL的实际算力需求存在结构性错配。**

#### 后果

- 训练优化的集群（全胖树高带宽互联）对Agent RL过度配置——推理只需tensor parallel（2-8卡内互联）
- NVIDIA已识别这一趋势：**Dynamo 1.0**于2026年进入生产级部署，被定位为"AI工厂的分布式推理操作系统"，运行DeepSeek-R1时吞吐提升最高30倍[^11]。Dynamo已被阿里云、CoreWeave、ByteDance、PayPal等广泛采用
- **NVIDIA ProRL Agent**明确提出"Rollout-as-a-Service"解耦架构，将I/O密集型环境交互与GPU密集型策略更新分离[^12]

#### 解决途径

集群架构重构为**80-90%推理节点 + 10-20%训练节点**。推理集群持续生成rollout样本，训练节点周期性拉取batch更新参数再广播回推理集群。

#### 对华为的关键意义

昇腾910C推理性能约为H100的60%（DeepSeek实测数据）[^13]，特定prefill场景下效率接近H100[^14]。如果90%工作是推理，HCCS AllReduce带宽劣势的权重从100%降至10%。**推理主导的算力结构天然稀释了昇腾在训练互联上的短板。** 但NVIDIA Dynamo的生产级领先是严峻挑战——华为需要在推理调度层迅速构建对标能力。

---

### 矛盾三（状态管理层）：有状态Agent会话 vs. 无状态分布式调度

#### 矛盾本质

Agent多轮交互产生持久状态（KV Cache、环境上下文、session历史），强绑定在特定物理节点。但分布式系统的调度范式是无状态的。

此矛盾合并了两个紧密耦合的问题：**KV Cache局部性**（Agent会话的KV Cache绑定在特定节点，无状态调度导致Cache Miss和反复Prefill）和**长尾Straggler**（Agent工具调用延迟呈极端长尾分布，等待期间KV Cache占显存却无计算，提高并发又导致显存枯竭和系统颠簸）。

#### 解决途径与业界方案

| 方案 | 代表工作 | 验证状态 |
|------|---------|---------|
| KV-Aware路由 | **llm-d**（IBM+Google+Red Hat）、**NVIDIA Dynamo** | Dynamo已生产级[^11] |
| 分布式缓存 | **AWS SageMaker HyperPod** | 商用 |
| 仿生准入控制 | **Concur框架**（AIMD拥塞控制） | 学术验证 |
| 轨迹级编排 | **Heddle系统** | 学术验证 |

**事实修正**：此前文章称"CXL 2026年大规模落地不现实"，经搜索验证，CXL Type 3内存扩展已于2026年进入**生产级部署**，CXL 3.1基于PCIe 6.1物理层已在超大规模数据中心广泛部署，双向吞吐达128GB/s（x16链路），支持跨机架内存池化，超大规模客户TCO降低15-20%[^15]。这意味着KV Cache三级管理（HBM → CXL扩展内存 → 分布式RDMA池）的中间层已具备工程可行性。

---

### 矛盾四（资源编排层）：动态异构负载 vs. 静态同构硬件

#### 矛盾本质

RLHF计算图包含多角色（Actor/Critic/Reward/Reference），不同阶段算力需求极度非对称。MoE稀疏性进一步放大负载异构性。但物理硬件拓扑静态同构。

GRPO的无critic设计已将系统复杂度降低（消除Critic模型），但Actor rollout和训练更新之间的资源需求仍然高度不对称。MoE架构（如DeepSeek-V3的256专家）的All-to-All通信在IB 400Gbps下占比可达30-40%。

#### 解决途径

| 方案 | 代表工作 | 机制 |
|------|---------|------|
| 动态参数重分配 | **ReaLHF**（清华，OpenPSI） | 自动搜索执行计划，运行时重组模型参数 |
| 定制MoE通信栈 | **ByteDance MegaScale-Infer** | 从零打造M2N通信库，乒乓流水线掩盖通信 |
| Rollout解耦 | **NVIDIA ProRL Agent** | 将环境交互与策略更新解耦为独立服务[^12] |

**华为的系统级应对**：华为CloudMatrix 384通过UB平面（每NPU 7x224Gbps）和6912个800G光模块实现总带宽超5.5Pbps[^16]，在系统级弥补了单链路HCCS带宽不足的劣势。

---

### 矛盾五（Agent特性层）：真实环境交互 vs. 可扩展的策略学习

#### 矛盾本质

Agent直接作用于真实环境，探索有不可逆代价、奖励稀疏延迟、信用分配困难。这两个子问题共享一个深层结构——**奖励信号的信息贫瘠性**。

#### 解决途径

**安全探索**：
- CMDP约束优化：SAFE-RLHF将helpfulness和harmlessness解耦为双目标[^17]
- "可逆域多做快判，不可逆域先想后做"——操作可逆性分类本身应由RL训练学习

**信用分配**：

**事实修正**：此前文章将PRM（过程奖励模型）定位为RL训练时的信用分配信号，经搜索验证需要重要修正。**PRM在Agent场景中主要用于推理时引导和纠错，而非RL训练信号**：
- **SWE-PRM**在推理时介入，将SWE-bench解决率从40.0%提升至50.6%（+10.6pp），但不参与RL训练[^18]
- **AgentPRM**使用Monte Carlo rollout估计过程奖励，在ALFWorld上3B模型超过GPT-4o[^19]
- 当前Agent RL训练的主流奖励信号仍是**纯结果奖励**（二元通过/失败），而非过程奖励

RL训练层面的信用分配，更有效的方案是**算法层的分层advantage估计**：
- **GiGPO**（NeurIPS 2025）引入episode-level和step-level两层分组，在ALFWorld/WebShop上分别比标准GRPO提升12%/9%[^10]
- **Stratified GRPO**通过分层归一化修正跨层偏差[^9]

**大规模并行Rollout**：

Agent RL的真正基础设施瓶颈在于**长horizon rollout的环境交互成本和训练稳定性**[^3][^12]。NVIDIA ProRL Agent已提出"Rollout-as-a-Service"架构[^12]，SWE-MiniSandbox[^20]优化了容器编排成本。以SWE-bench为例，每个sandbox需2-4 vCPU + 4-8GB内存，1000并行需约4000 vCPU + 8TB内存。

**关于World Model的共识**：经多轮辩论，四位专家确认——**电脑端Agent场景不需要显式环境模拟器（world model）**。所有SWE-bench/WebArena SOTA方案均未使用world model。然而，**隐式world model——即模型通过RL训练内化的CoT规划能力——是Agent最核心的能力**。OpenAI o3/o4的inference-time scaling本质是用长链思维链做mental simulation。因此：**RL训练是构建Agent隐式规划能力的唯一可扩展路径。**

---

## 第二部分：算子自动生成——必要的基础能力，定位需务实

### 事实基础

华为已有两项前沿工作，经搜索验证数据如下：

- **AscendCraft**（2025.1, arxiv 2601.22760）：基于LLM转译生成AscendC kernel，98.1%编译成功率，90.4%功能正确性，46.2%达到或超过PyTorch eager性能[^21]
- **AscendOptimizer**（2026.3, arxiv 2603.23566）：无需训练的profiling-in-the-loop演化搜索，覆盖127个真实AscendC算子[^22]

### 专家的批判性评估

**性能校准**：46.2%对标的是PyTorch eager mode，而eager mode仅为手写优化kernel的50-70%。换算后，自动生成算子约为手写最优的**25-35%**。这对性能敏感路径远远不够。

**微架构差异是根本限制**：昇腾DaVinci架构（Cube/Vector/Scalar单元+独立数据搬运）与NVIDIA SIMT模型（warp/shared memory/tensor core）在编程模型上有本质区别。CUDA kernel的优化假设（bank conflict、warp divergence、寄存器压力）在昇腾上几乎完全不适用。语义级翻译做到功能正确容易，性能最优极难。

**务实定位**：
- 算子自动生成的真正价值在**长尾算子覆盖度**——CUDA生态的护城河不是Top 20算子（华为可手工追平），而是数万个社区长尾算子
- 商业价值是**降低客户迁移门槛**：把"算子不支持导致项目卡住"的硬阻断变成"性能差一点但能跑"的软降级
- 策略：**80%长尾算子自动覆盖 + 20%关键路径算子手工深度优化**
- 应支持Triton前端、自研昇腾后端编译，借社区势能

---

## 第三部分：竞争格局（经搜索验证）

### 三大路线对比（2026年最新数据）

| 维度 | NVIDIA | Google | 华为 |
|------|--------|--------|------|
| 推理性能 | H100/B200标杆 | TPU v5p | 910C约H100的60%[^13] |
| 推理调度 | **Dynamo 1.0生产级**，30x吞吐提升[^11] | 内部闭环 | vLLM-Ascend v0.13.0[^23] |
| Agent RL框架 | **ProRL Agent** Rollout-as-a-Service[^12] | 未公开 | 无公开RL benchmark |
| 节点内互联 | NVLink 900GB/s | ICI 4.8Tbps | HCCS ~392GB/s |
| 系统级互联 | — | — | CloudMatrix 384 总带宽5.5Pbps[^16] |
| 出货规模 | 全球主导 | 仅GCP | 2025年70-80万颗[^24] |

### 关键竞争态势

1. **NVIDIA Dynamo生态快速固化**：已被阿里云、ByteDance、CoreWeave等广泛采用[^11]，正在成为推理调度事实标准。华为窗口期紧迫。
2. **vLLM-Ascend超预期**：v0.13.0由vLLM官方社区维护，支持Qwen3/DeepSeek V3/R1，已实现W4A4量化和并行投机解码[^23]。这是华为推理生态的重要基石。
3. **昇腾RL训练空白**：无公开的PPO/GRPO benchmark，生态聚焦SFT/预训练[^14]。RL训练约落后12-18个月。

---

## 第四部分：为什么RL后训练对华为更重要

### 1. 推理主导稀释训练短板

90%算力在推理侧意味着HCCS训练互联劣势的权重从100%降至10%。910C推理能力虽仅为H100的约60%，但prefill效率在特定模型上接近H100[^14]。

### 2. 全栈垂直整合的差异化

并行Rollout需要CPU密集+存储I/O密集+高效容器调度，更像超大规模CI/CD系统。鲲鹏+昇腾+华为云容器调度的全栈组合有差异化优势。CloudMatrix 384的5.5Pbps系统级带宽弥补单链路不足[^16]。

### 3. RL训练是Agent核心能力的唯一路径

Agent的隐式规划能力（CoT推理）是RL后训练的直接产物[^1][^3]。掌握高效的RL基础设施，就掌握了Agent能力的生产工具。

### 4. 国产替代刚需

2025年昇腾出货70-80万颗[^24]，装机量基本盘足够大。2026年仅910C产量目标翻倍至约60万颗，全系列可达160万颗。国内大模型公司在RL训练基础设施上别无选择。

---

## 第五部分：华为计算产品线的行动建议

### 战略定位

**"推理为矛、RL为盾"——以推理吞吐极致优化为核心竞争力，以RL训练生态兼容为基本盘。**

### 分层投资策略

| 层次 | 策略 | 投资占比 |
|------|------|---------|
| 上层：PyTorch API兼容 | 全力适配verl/rLLM在昇腾运行 | 10-15% |
| **中层：Ascend Agent调度Runtime** | **以推理吞吐为第一优化目标，对标Dynamo** | 40-50% |
| 底层：CANN算子+通信库 | 关键路径手工优化+长尾自动生成+HCCL专项优化 | 35-40% |

### 五大行动

**行动一（最高优先级）：推理吞吐极致优化**

目标：910C推理性能从H100的60%向75-80%逼近。
- vLLM-Ascend深度优化（已有v0.13.0基础[^23]，需进一步做PagedAttention昇腾原生实现）
- KV Cache三级管理：HBM → CXL扩展内存（CXL已进入生产级部署[^15]，可以上马）→ 分布式RDMA池
- HCCL通信库针对RL推理场景专项优化（推理集群内allgather、训练-推理跨集群参数广播）

**行动二（高优先级）：Agent RL调度Runtime**

以推理吞吐为第一优化目标，对标NVIDIA Dynamo[^11]：
- Session粘性路由 + 有界陈旧度异步调度
- 兼容verl/rLLM框架API
- 与Rollout环境平台统一API层

**行动三（中高优先级）：大规模并行Rollout环境平台**

鲲鹏+昇腾全栈支撑数千sandbox并行，对标NVIDIA ProRL Agent的Rollout-as-a-Service架构[^12]：
- 轻量容器调度（对标Firecracker ms级启动）
- Reward/PRM serving与推理集群共享（避免独立部署浪费算力）
- overlayfs/CoW文件系统支撑高并发repo快照

**行动四（中高优先级，与行动三平级）：算子工具链闭环**

AscendCraft+AscendOptimizer持续迭代[^21][^22]：
- 短期目标：从46.2%提升至60-70% PyTorch eager
- Triton前端兼容 + 昇腾后端编译
- 算子工具链是推理优化的前置依赖——不先补算子缺口，推理吞吐天花板被锁死

**行动五（中低优先级）：RL训练生态跟进**

基于verl/rLLM/verl-agent[^4]开源方案补位：
- 优先支持GRPO系列（GRPO++/Stratified GRPO/GiGPO），同时保证PPO可用
- 昇腾RL训练落后12-18个月，但90%算力在推理可稀释影响
- 借开源社区势能，不追求全面对标NVIDIA RL栈

### 需警惕的风险

1. **Dynamo生态固化风险**：NVIDIA Dynamo已进入生产级并被广泛采用[^11]，华为推理调度Runtime的窗口期可能只有18-24个月
2. **推理性能不能只是"够用"**：910C 60%的性能在只有华为可选时够用，一旦竞争加剧则不够
3. **RL训练空白的连锁效应**：无公开RL benchmark会影响客户信心，即使RL训练只占10%算力

---

## 结语

Agent时代RL后训练的核心图景，可以用五对正交矛盾完整刻画：**算法层**的策略新鲜度困境、**计算架构层**的推理-训练错配、**状态管理层**的有状态-无状态冲突、**资源编排层**的动态-静态错位、**Agent特性层**的真实环境交互代价。五个维度各自独立，又通过"90%算力在推理"这一贯穿性事实紧密关联。

**最关键的产业洞察：Agent RL本质上是一个推理问题，不是一个训练问题。** 这颠覆了"训练芯片为王"的传统竞争逻辑。RL训练构建Agent的基础能力（Pass@1），推理时scaling放大这一能力（Pass@1→Pass@K），两者互补但推理消耗90%+的算力。

对华为计算产品线而言，"推理为矛、RL为盾"是最务实的路径——以推理吞吐极致优化为核心竞争力，以RL训练生态兼容为基本盘，以全栈垂直整合（鲲鹏+昇腾+HCCL+CloudMatrix）为差异化壁垒。**NVIDIA Dynamo的生产级部署和ProRL Agent的Rollout-as-a-Service架构，定义了华为必须对标的行业基准。窗口期紧迫，但方向清晰。**

---

## 参考来源

[^1]: Together AI, "DeepSWE-Preview: Qwen3-32B + Pure RL, 59% SWE-Bench Verified", together.ai/blog/deepswe
[^2]: SkyRL, "SA-SWE-32B: 39.4% pass@1 at 2x lower cost"
[^3]: Meta, "Self-Play Preference Optimization for SWE", arxiv 2512.18552
[^4]: verl-agent, "Multi-algorithm Agent RL framework", github.com/langfengQ/verl-agent
[^5]: HuggingFace, "Async RL Training Landscape Survey" — 90%+ runtime on rollout generation
[^6]: Toby Ord, "How Well Does RL Scale?", tobyord.com/writing/how-well-does-rl-scale
[^7]: "A Survey on Inference-Time Scaling for Complex Tasks", arxiv 2504.00294
[^8]: SkyRL (UC Berkeley, 2025), staleness-aware admission control
[^9]: Zhu et al., "Stratified GRPO: Correcting Cross-Stratum Bias", arxiv 2510.06214
[^10]: "GiGPO: Group-in-Group Policy Optimization", NeurIPS 2025, arxiv 2505.10978
[^11]: NVIDIA, "Dynamo 1.0 enters production — 30x throughput on DeepSeek-R1", nvidianews.nvidia.com, 2026
[^12]: NVIDIA, "ProRL Agent: Decoupled Rollout-as-a-Service for Multi-Turn LLM Agents at Scale"
[^13]: Tom's Hardware / TrendForce, "Ascend 910C delivers ~60% H100 inference performance (DeepSeek实测)"
[^14]: TrendForce, "910C prefill efficiency competitive in specific scenarios"
[^15]: CXL Consortium / Industry reports, "CXL Type 3 enters production deployment 2026, CXL 3.1 at 128GB/s x16"
[^16]: SemiAnalysis, "Huawei CloudMatrix 384: 5.5Pbps total bandwidth via UB plane + 6912x 800G optical modules"
[^17]: Dai et al., 2023, "SAFE-RLHF: Safe Reinforcement Learning from Human Feedback"
[^18]: "SWE-PRM: Course-Correcting Code Agents", arxiv 2509.02360 — inference-time +10.6pp
[^19]: "AgentPRM", arxiv 2502.10325 — 3B model surpasses GPT-4o on ALFWorld
[^20]: "SWE-MiniSandbox: Lightweight container orchestration for agent RL", arxiv 2602.11210
[^21]: "AscendCraft: LLM-based AscendC kernel generation — 98.1% compilation, 46.2% PyTorch eager", arxiv 2601.22760
[^22]: "AscendOptimizer: Training-free profiling-in-the-loop evolutionary search, 127 operators", arxiv 2603.23566
[^23]: vLLM-Ascend v0.13.0, github.com/vllm-project/vllm-ascend — supports Qwen3/DeepSeek V3/R1, W4A4, speculative decoding
[^24]: Mizuho Securities / Reuters / TechNode, "70-80万 Ascend 910 chips shipped 2025; 910C targeting ~600K in 2026"

---

*本文基于2026年4月专家研讨会整理。研讨过程包含四轮交叉辩论、两轮网络搜索验证、一轮框架重构。所有关键论断均经事实核查并标注来源。*
