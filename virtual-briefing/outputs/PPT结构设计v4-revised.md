# Agentic RL 后训练洞察 — PPT结构设计 v4-revised（融合版）

> 基于v4 + 原始PPT技术回拉融合
> 核心原则：原始技术（基础设施层/系统层）与v4技术（算法前沿层）互补而非替代
> 每个挑战应有多层解法（系统层/框架层/算法层），不能只有一层
> 范围：**Coding/SE Agent**（诚实界定scope）
> 团队定位：帮助客户在NPU上开箱即用地完成Agent后训练任务

---

## 技术融合对照表

| 原始技术 | 原始定位 | v4状态 | 2026最新状态 | 融合决策 | 理由 |
|---------|---------|--------|-------------|---------|------|
| **PRIME-RL**（PrimeIntellect） | 挑战①系统架构层 | 删除，被ProRL Agent替代 | **极度活跃**：INTELLECT-3（106B MoE）已发布，PRIME-RL成为PrimeIntellect Lab全栈平台核心；Dynamo 1.0列其为生产部署用户 | **拉回，更新至2026状态** | 与ProRL Agent互补：PRIME-RL解决去中心化异步训练架构问题，ProRL Agent解决rollout-as-a-service问题。两者不在同一栈层竞争 |
| **SkyRL**（UC Berkeley） | 挑战①训练框架层 | 删除，被CSO替代 | **极度活跃**：SkyRL-Agent（2025.11）发布多轮Agent训练框架；与Harbor集成（2026.2）；SA-SWE-32B达SWE-bench 39.4%；有off-policy correction机制 | **拉回但更新**：从"staleness控制单点"升级为"多轮Agent RL训练框架" | SkyRL已从单一staleness控制进化为完整Agent RL训练框架，与ProRL Agent是直接竞争/互补关系。提供框架层视角 |
| **DAPO**（ByteDance/清华） | 挑战①训练算法层 | 删除，被CISPO替代 | **仍然前沿且有后续**：DAPO→DISPO→CISPO演进链清晰；DISPO在AIME'24达61.04%（vs CISPO 55.42%，DAPO 50.21%）；DAPO的clip-higher机制被DeepSWE直接采用 | **拉回但重构**：呈现DAPO→DISPO→CISPO演进链，而非单独列DAPO | DAPO不是被CISPO"替代"，而是CISPO的前身。v4已有CISPO，增加演进链说明即可。DeepSWE采用DAPO机制证明其持续影响力 |
| **Dynamo**（NVIDIA） | 挑战②推理调度层 | 降级为表格一行 | **已达生产级**：Dynamo 1.0于2026.3.16发布，KV-aware routing实测20x TTFT提升+4x端到端延迟降低；ByteDance/Tencent/PrimeIntellect等已生产部署 | **拉回，恢复为重点技术** | 从GTC 2025论文升级为生产级产品，是Agent推理基础设施的事实标准。与vllm-ascend直接相关——昇腾推理引擎需要对标的核心目标 |
| **LMCache** | 挑战②存储/缓存层 | 删除 | **已达生产级**：P2P共享从实验升级为生产（2026.1）；与Dynamo 1.0原生集成；与vLLM Production Stack深度整合；支持多模态模型 | **拉回，恢复为重点技术** | 已成为Dynamo生态核心组件。Agent多轮对话的KV cache复用是刚需，LMCache是当前最成熟方案 |
| **TraCT+CXL** | 挑战②硬件/内存层 | 删除 | **学术前沿**：论文级（2025.12），基于Dynamo框架实现；CXL 4.0路线图明确；TTFT降低9.8x | **拉回但标注为远期方向** | 代表硬件互联下一代方向。昇腾HCCS互联是天然适配点，但CXL商用仍需时间。作为远期方向保留 |
| **SALT**（Amazon） | 挑战③归因层 | 保留 | 保持不变 | **已保留** | v4已有，无需改动 |
| **SWE-PRM** | 挑战③奖励建模层 | 删除 | **持续活跃**：SWE-PRM在SWE-bench Verified上+10.6pp；AgentPRM（2025.11）提出promise+progress双维度，8x计算效率提升；ToolPRMBench专用评测出现 | **拉回但升级**：从SWE-PRM单技术升级为"Agent PRM方向"（含SWE-PRM、AgentPRM） | Agent任务的PRM研究在2025-2026爆发式增长，不再是v4中说的"基本不存在"。直接支撑评估挑战的解法层 |
| **HiPER** | 挑战③探索策略层 | 删除 | **2026.2新论文**：ALFWorld 97.4%、WebShop 83.3%（+6.6%/+8.3% SOTA）；HiMAC（2026.3）、SkillRL（2026.2）等跟进工作涌现 | **拉回** | 分层策略是Agent长程任务的结构性解法。v4缺少探索策略层方案，HiPER填补空白 |

**融合结论**：9个原始技术中，8个被拉回（1个已保留不动）。其中3个直接恢复（PRIME-RL、Dynamo、LMCache），3个拉回但更新/重构（SkyRL、DAPO演进链、SWE-PRM升级），1个拉回标注远期（TraCT+CXL），1个直接恢复（HiPER）。

---

## 页码总览与逻辑流

```
P1   Agent时代已来              <- 定调：产品/开源/资本三维爆发（不变）
P2   Coding Agent RL是当前sweet spot <- scope收窄到coding/SE领域（不变）
P3   为什么必须是Agentic RL     <- 动因：harness->模型能力迁移 + RL不可或缺（不变）
P4   三大结构性变化 -> 三大挑战  <- 核心推导页（不变）
P5   技术地图                   <- 【重大修改】融合原始9技术+v4技术，多层解法完整呈现
P6   挑战①：新鲜度与归因        <- 【重大修改】融合系统/框架/算法三层解法
P7   挑战②：有状态推理与高效采集 <- 【重大修改】融合推理调度/缓存/硬件+Rollout-as-a-Service
P8   挑战③：奖励评估与探索策略   <- 【重大修改】融合归因/PRM/分层策略+harness质量
P9   横切约束：精度与一致性      <- 不变
P10  我们的差异化：确定性RL训练  <- 不变
P11  行动路线图                 <- 微调：技术映射图更新
P12  收束：竞争定位与客户价值   <- 不变
```

**逻辑链**：现象(P1) -> Coding Agent RL是sweet spot(P2) -> 为什么RL(P3) -> 变化->挑战(P4) -> 导航(P5) -> 三挑战多层解法(P6-P8) -> 横切约束(P9) -> 差异化卖点(P10) -> 行动路线图(P11) -> 竞争定位(P12)

**与v4的关键变化**：
- P5技术地图从6项前沿技术扩展为12+项，覆盖系统/框架/算法/硬件多栈层
- P6-P8每个挑战页融合多层解法，恢复"每个挑战在不同栈层都有解法"的结构价值
- 总页数保持12页不变——不机械恢复原始15页，而是在12页框架内做内容融合

---

## P1 | 定调层：Agent时代已来

**与v4完全一致，不变。**

---

## P2 | 过渡层：Coding/SE Agent RL是当前Sweet Spot

**与v4完全一致，不变。**

---

## P3 | 动因层：为什么必须是Agentic RL

**与v4完全一致，不变。**

---

## P4 | 核心推导层：三大结构性变化 -> 三大挑战

**与v4完全一致，不变。**

> **框架标注**：以下三分法是**我们的工程分析框架**（按RL循环的评估->归因->采集三环节切分），不是学术界的标准分类。

### 推导逻辑

**左半页：三个结构性变化**

| # | LLM RL | Agent RL | 变化本质 |
|---|--------|----------|---------|
| 1 | 轨迹=几十到几百token | 轨迹=几千到几万token，跨多轮工具调用 | **轨迹长度量级跃升** |
| 2 | 每条轨迹秒级生成 | 每条轨迹分钟级（环境执行、API调用） | **单条经验成本爆炸** |
| 3 | 正确性可验证（数学/代码有标准答案） | 正确性模糊多标准（开放任务无唯一正解） | **评估从确定变为模糊** |

**右半页：三大变化直接对应三大挑战**

```
变化1 轨迹长度跃升 --> 挑战1 归因（Credit Assignment）
变化2 单条经验成本爆炸 --> 挑战2 采集（Experience Collection）  
变化3 评估从确定到模糊 --> 挑战3 评估（Reward Evaluation）
```

**挑战间耦合说明**（v4已有，保留）：
> 三大挑战并非完全正交。技术地图（P5）在表格中显式标注了cross-cutting trade-off。

**实战优先级说明**：
> 上述排序是因果链顺序，不是落地优先级。实战中采集（ProRL Agent）排第一——"公共底座不通，上层都是空谈"。

---

## P5 | 技术地图【重大修改：融合多层解法】

**标题**：技术地图：三大挑战 x 多层解法 x 成熟度/局限/跨挑战影响

**v4-revised核心修改**：
1. **恢复原始PPT"每个挑战在不同栈层都有解法"的结构价值**
2. **融合原始9技术 + v4新增技术，按栈层分层呈现**
3. **保留v4的cross-cutting trade-off列和成熟度标注**
4. **新增"2026最新进展"列，体现技术演进**

### 挑战1：归因 / 新鲜度困境

| 栈层 | 前沿解法 | 成熟度 | 2026最新进展 | 已知局限 | 跨挑战影响 | NPU适配 |
|------|---------|--------|-------------|---------|-----------|---------|
| **系统架构层** | **PRIME-RL**（PrimeIntellect，去中心化异步RL） | **生产级** | INTELLECT-3（106B MoE）已发布；Lab全栈平台上线；Dynamo 1.0生产部署用户 | 依赖H200集群；去中心化模式与客户集中式集群场景不完全匹配 | 系统层异步代价低 -> 减轻框架/算法层对staleness的补偿压力 | 未适配 |
| **训练框架层** | **SkyRL-Agent**（UC Berkeley NovaSky） | **框架级** | SA-SWE-32B达SWE-bench 39.4%；Harbor集成（2026.2）；off-policy correction + truncated IS | 与ProRL Agent功能部分重叠；对rollout环境管理能力不如ProRL | 框架层staleness控制 -> 提高归因数据质量 | 未适配 |
| **训练框架层** | **A-3PO**（staleness-aware proximal policy） | 论文级 | 1.5B/8B模型验证，1.8x训练加速，staleness-aware插值 | 仅验证reasoning任务，未验证Agent场景 | 与SkyRL的off-policy correction互补 | 未适配 |
| **训练算法层** | **DAPO -> DISPO -> CISPO 演进链** | 框架级 | DISPO达AIME'24 61.04%（vs CISPO 55.42%，DAPO 50.21%）；DAPO clip-higher被DeepSWE采用 | CISPO后期entropy崩塌（DISPO证实）；100步+长轨迹importance weight衰减 | entropy崩塌影响评估稳定性；长轨迹weight衰减影响归因质量 | 未适配 |
| **归因算法层** | **CSO**（关键步骤检测+验证） | 论文级 | 8B模型追平GPT-4.1（GAIA-Text-103） | 需PRM；额外计算开销高 | 加剧采集瓶颈 | 未适配 |
| **归因算法层** | **SALT**（轨迹图step-level归因） | 论文级 | Amazon，plug-and-play | 依赖轨迹间状态共享；高随机环境退化 | 影响较小（轻量级） | 未适配 |

### 挑战2：采集 / 有状态难题

| 栈层 | 前沿解法 | 成熟度 | 2026最新进展 | 已知局限 | 跨挑战影响 | NPU适配 |
|------|---------|--------|-------------|---------|-----------|---------|
| **Rollout服务层** | **ProRL Agent**（NVIDIA，Rollout-as-a-Service） | **框架级** | 2026.3开源；Qwen3-8B SWE-bench 9.6->18.0%；token-in/token-out一致性 | 纯CUDA；HTTP通信延迟；Singularity容器 | **公共底座**——所有上层RL算法受益 | **未适配（第一优先级）** |
| **推理调度层** | **NVIDIA Dynamo 1.0**（KV-aware routing） | **生产级** | 2026.3.16正式发布；KV-aware routing 20x TTFT提升+4x E2E延迟降低；ByteDance/Tencent/PrimeIntellect等已生产部署；与vLLM/SGLang/LMCache原生集成 | NVIDIA生态锁定；NIXL库依赖NVLink/RDMA | Agent多轮对话KV复用的调度基础 | **需对标适配** |
| **KV缓存共享层** | **LMCache**（跨引擎P2P KV cache共享） | **生产级** | P2P共享已从实验升级为生产（2026.1）；与Dynamo 1.0原生集成；支持多模态；吞吐2.0GB/s | 依赖NIXL传输路径（NVIDIA硬件优化） | KV cache集群化 -> 提高rollout吞吐（减少重复prefill） | **需对标适配** |
| **硬件互联层** | **TraCT+CXL**（CXL共享内存KV传输） | 论文级（远期） | 2025.12论文；基于Dynamo实现；TTFT降低9.8x，P99降低6.2x | CXL商用硬件尚未大规模可得；rack-scale限制 | 软件层优化到极限后的硬件突破方向 | **远期机会（HCCS天然适配点）** |
| **异步RL框架层** | **Forge+CISPO**（MiniMax，异步off-policy RL） | 框架级 | 比DAPO快2x达同等性能 | entropy崩塌；长轨迹weight衰减 | 长轨迹weight衰减影响归因质量 | 未适配 |

### 挑战3：评估 / 奖励失控

| 栈层 | 前沿解法 | 成熟度 | 2026最新进展 | 已知局限 | 跨挑战影响 | NPU适配 |
|------|---------|--------|-------------|---------|-----------|---------|
| **归因算法层** | **SALT**（Amazon，轨迹图step-level归因） | 论文级 | plug-and-play，可叠加在GRPO上 | 依赖轨迹间状态共享 | 轻量，影响小 | 未适配 |
| **奖励建模层** | **SWE-PRM / AgentPRM**（步级过程奖励模型） | **论文级->框架级** | SWE-PRM：SWE-bench +10.6pp；AgentPRM（2025.11）：promise+progress双维度，8x计算效率；ToolPRMBench专用评测出现 | SWE-PRM依赖闭源模型；Agent PRM训练数据获取仍困难 | PRM质量直接决定归因和采集的方向 | 未适配 |
| **探索策略层** | **HiPER**（分层plan-execute RL） | 论文级 | 2026.2发布；ALFWorld 97.4%、WebShop 83.3%（+6.6%/+8.3% SOTA）；HiMAC、SkillRL等跟进 | 双层模型增加部署复杂度；planner和executor耦合调优 | 分层结构改善归因（中间子目标提供稠密信号） | 未适配 |
| **Harness工程层** | **Composer 2**（Cursor，高保真仿真harness中RL） | 产品级 | RL-trained self-summarization，compaction error降50% | 依赖强base+B300；sandbox != production | harness质量决定归因和采集的方向 | 不适用 |
| **Harness工程层** | **Anthropic Harness Patterns**（5种Agent模式） | 工程实践级 | 设计参考 | 无代码/无benchmark | 无直接trade-off | 不适用 |

### 横切约束

| 栈层 | 前沿解法 | 成熟度 | 已知局限 | 跨挑战影响 | NPU适配 |
|------|---------|--------|---------|-----------|---------|
| 全栈 | 训推一致性/确定性计算/算子验证 | **原型中** | batch-invariant原型阶段；AllReduce是关键瓶颈（开销20-50%） | 精度不一致放大所有挑战的误差 | **原型中** |

### 基础设施层已有能力（实线）

- MindSpeed RL：基础RL训练（GRPO）在384卡NPU集群上跑通，支持DeepSeek-R1-671B规模
  - **说明**：目前验证的是**reasoning任务**的RL训练，不是Agent RL。从reasoning RL到Agent RL需要rollout管理、sandbox交互、多轮状态管理
- vllm-ascend：推理引擎NPU适配，sleep mode已初步解决，hybrid engine刚打通
  - **说明**：支持标准LLM serving，Agent RL rollout特征（动态batch、prefix caching跨轮复用、工具调用context挂起恢复）未优化
  - **关键对标**：Dynamo 1.0 + LMCache的KV-aware routing + P2P cache共享是当前GPU生态事实标准。vllm-ascend需在架构层面对标这些能力
- 5个RL关键路径算子CANN vs CUDA精度比对档案

### 前沿技术层待建能力（虚线，按优先级排序）

```
【优先级1】ProRL Agent Rollout-as-a-Service昇腾适配
          -> 公共底座，补MindSpeed RL缺的rollout和环境交互
【优先级1b】Dynamo 1.0 KV-aware routing能力对标
          -> vllm-ascend需具备KV cache元数据暴露、跨节点迁移、prefix-aware路由
          -> LMCache P2P共享机制的昇腾对标
【优先级2】确定性RL训练（batch-invariant算子 + 精度基线）
          -> 差异化卖点，关键瓶颈AllReduce（阈值30%）
【优先级2b】SWE-bench参考harness实现
          -> 确保客户用得起来，与确定性训练形成双轮驱动
【优先级3】CISPO/Forge异步RL框架评估
          -> DAPO->DISPO->CISPO演进链中选择最稳定方案
【优先级3b】SkyRL-Agent框架评估
          -> 与ProRL Agent在Agent RL训练场景的对比评估
【优先级4】CSO/SALT归因技术NPU验证
          -> 待Agent PRM生态成熟后再投入
【优先级4b】SWE-PRM/AgentPRM奖励建模评估
          -> Agent PRM方向2025-2026快速发展，值得跟踪
【远期】TraCT+CXL硬件互联方向
          -> HCCS天然适配点，待CXL商用成熟
```

---

## P6 | 挑战1：归因与新鲜度困境【重大修改：多层解法融合】

**标题**：挑战一：长轨迹归因 + 异步数据新鲜度 — 系统/框架/算法三层协同

**v4-revised核心修改**：
1. **恢复原始PPT的"系统架构层-训练框架层-训练算法层"三层结构**
2. **融合PRIME-RL（系统层）、SkyRL（框架层）、DAPO演进链（算法层）与CSO/SALT（归因层）**
3. **呈现从系统到算法的完整防线**

### 问题描述

Agentic RL的归因和新鲜度困境有两个维度：
1. **归因难**：Agent轨迹几十步到上百步，trajectory级奖励太粗，无法区分"碰巧做对"和"真正学会"
2. **新鲜度低**：异步架构下rollout生成时的参数在训练端已过期，Agent episode分钟级放大陈旧度

两个维度叠加：陈旧数据 + 粗粒度归因 = 梯度方向既不准又不新。

### 第一道防线：系统架构层 — PRIME-RL

**来源**：PrimeIntellect（美国），PRIME-RL开源框架
**2026状态**：**生产级** — INTELLECT-3（106B MoE，512xH200）已发布；Lab全栈平台上线；被Dynamo 1.0列为生产部署用户

**核心思路**：从系统架构层面让"从生成到训练"的延迟足够短
- 去中心化调度：worker间P2P协同，消除中心化调度器瓶颈
- 1000+ GPU原生异步RL：async-only设计，是"always off-policy"的哲学
- FSDP2训练 + vLLM推理 + FP8降内存
- TOPLOC（rollout验证）+ SHARDCAST（权重高效广播）

**与v4技术的关系**：PRIME-RL解决的是**训练架构**的异步问题（如何让千卡集群高效异步训练），ProRL Agent解决的是**rollout服务**的解耦问题（如何让rollout生命周期独立管理）。两者在不同栈层互补。

**NPU启示**：异步训练调度能力、跨节点通信效率直接影响Agentic RL数据新鲜度。PRIME-RL的SHARDCAST权重广播机制值得昇腾对标。

### 第二道防线：训练框架层 — SkyRL-Agent + A-3PO

**SkyRL-Agent**（UC Berkeley NovaSky）：
- 2026状态：SA-SWE-32B达SWE-bench Verified 39.4%（2x成本降低）
- 核心机制：transition-based轨迹记录（每次LLM调用记录input/output/log_prob），off-policy correction（truncated IS + token-level/sequence-level ratio），异步pipeline dispatcher（1.55x加速）
- 工具增强：AST-based search tool集成，提升rollout Pass@K

**A-3PO**（staleness-aware proximal policy approximation）：
- 2026状态：1.5B/8B模型验证，1.8x训练加速
- 核心机制：用log-probability空间插值近似proximal policy，无需额外forward pass；staleness-aware插值权重——越新鲜的数据权重越大

**框架层小结**：SkyRL-Agent提供完整的多轮Agent RL训练框架，A-3PO提供staleness-aware的算法级补偿。两者共同构成框架层的"保鲜期"管理。

### 第三道防线：训练算法层 — DAPO -> DISPO -> CISPO演进链

**演进链**：
```
DAPO (2025.3)                    DISPO (2026.2)               CISPO (2026.3)
ByteDance/清华                    学术                         MiniMax
Clip-Higher防熵崩塌               解耦正负样本裁剪              抛弃trust region
Dynamic Sampling过滤噪声           AIME'24: 61.04%             快2x达同等性能
Token-level Loss降长序列噪声                                   但entropy崩塌风险
AIME'24: 50.21%                                               AIME'24: 55.42%
```

**关键认知**：这不是"DAPO被CISPO替代"，而是一条持续演进的路线：
- DAPO的Clip-Higher机制**被DeepSWE直接采用**（v4 P2已提到），证明其持续影响力
- DISPO在AIME'24达61.04%，目前效果最佳但稳定性待验证
- CISPO训练效率最高但有entropy崩塌风险
- **选型建议**：Gate 1-2阶段用DAPO（最稳定），Gate 3评估DISPO（最高效果）

### 第四道防线：归因算法层 — CSO + SALT

**与v4完全一致，保留不变。**

- **CSO**：精度高但前提重（需PRM，加剧采集），投入优先级低于采集和横切约束
- **SALT**：轻量plug-and-play，可叠加在GRPO上，场景受限但成本可忽略

### 本页小结

归因与新鲜度困境不是单一技术能解决的，需要四道防线协同：

```
系统架构层（PRIME-RL）：让异步代价在架构层面最小化
       ↓ 仍有residual staleness
训练框架层（SkyRL/A-3PO）：给数据贴保鲜期，过期丢弃/补偿
       ↓ 仍有不完美数据
训练算法层（DAPO/DISPO/CISPO）：提高算法对不完美数据的鲁棒性
       ↓ 仍有粗粒度归因
归因算法层（CSO/SALT）：从trajectory级细化到step级
```

---

## P7 | 挑战2：有状态推理与高效采集【重大修改：多层解法融合】

**标题**：挑战二：每条轨迹都很贵 — 从Rollout服务到推理基础设施的全栈优化

**v4-revised核心修改**：
1. **保留ProRL Agent为第一优先级（Rollout服务层）**
2. **恢复Dynamo + LMCache为推理基础设施层的核心对标目标**
3. **保留TraCT+CXL为远期硬件方向**
4. **呈现"Rollout服务 -> 推理调度 -> KV缓存 -> 硬件互联"四层解法**

### 问题描述

训推分离把环境交互全放到推理侧，三重复杂度叠加：
1. 环境并发管理（千级sandbox）
2. KV cache局部性 vs 调度灵活性
3. 工具调用长尾延迟拖拽整条rollout

### 第一层：Rollout服务层 — ProRL Agent（第一优先级）

**与v4完全一致，保留不变。**

**ProRL Agent — Rollout-as-a-Service**（NVIDIA，2026.3，**第一优先级**）：
- 来源：arXiv 2603.18815
- 核心：将rollout生命周期作为独立HTTP服务从训练循环解耦；Token-in/token-out一致性保证
- 效果：Qwen3-8B SWE-Bench Verified 9.6% -> 18.0%
- 为什么是第一优先级（不变）：公共底座 + NVIDIA刚开源 + 与vllm-ascend复用度最高

**Gate 1量化标准**（不变）：

| Check项 | 标准 | 依据 |
|---------|------|------|
| 单条rollout完成时间 | **<30分钟** | L7数据 |
| 并发稳定性 | **4条rollout同时稳定运行** | L7 |
| Sandbox适配 | Singularity->Docker适配方案设计文档 | L4 |
| 存储I/O观测 | rollout环境操作I/O延迟分布数据 | L6 |

### 第二层：推理调度层 — NVIDIA Dynamo 1.0【恢复】

**来源**：NVIDIA，GTC 2025发布，2026.3.16 Dynamo 1.0生产版发布
**2026状态**：**生产级** — ByteDance/Tencent/PrimeIntellect/CoreWeave等已生产部署；被定位为"AI Factory的推理操作系统"

**核心能力**：
- **KV-aware Router**：Radix Tree追踪全集群KV分布，计算cache overlap score，请求路由到已有缓存的节点
- **NIXL**：点对点KV cache传输库（支持NVLink/RDMA/GPU Direct Storage）
- **KVBM**（KV Block Manager）：多层级内存管理（GPU VRAM -> CPU RAM -> SSD）
- **Prefill-Decode分离**：prefill和decode阶段独立调度
- **实测**：20x TTFT提升 + 4x端到端延迟降低

**与ProRL Agent的关系**：ProRL Agent的推理后端可以跑在Dynamo上。Dynamo解决"推理引擎怎么高效调度"，ProRL Agent解决"rollout生命周期怎么管理"。

**NPU启示**：这是vllm-ascend需要对标的核心目标。具体包括：
- KV cache元数据暴露：让路由层知道哪个节点有哪段KV
- 跨节点KV迁移：高效的KV cache传输能力
- Prefix-aware路由：Agent多轮对话prefix复用率极高

### 第三层：KV缓存共享层 — LMCache【恢复】

**来源**：开源项目，集成vLLM/SGLang/KServe/Dynamo
**2026状态**：**生产级** — P2P共享从实验升级为生产（2026.1）；与Dynamo 1.0原生集成；支持多模态模型

**核心能力**：
- **P2P共享**：引擎间直接传输KV cache，打破节点孤岛
- **NIXL集成**：使用NVIDIA低延迟传输库，支持GPU/CPU/外部存储间高速KV移动
- **Cache Offloading**：GPU VRAM -> CPU RAM多级卸载
- **Connector接口**：vLLM动态加载KV connector（2025.6），标准化存取API
- **实测**：多轮QA 3-10x吞吐提升

**关键逻辑**：KV cache不应是引擎私有资源，而应是集群级的共享池。Agent场景95%+ context可复用。

**NPU启示**：vllm-ascend需支持标准化KV cache存取接口。LMCache的P2P共享机制是昇腾推理栈需要对标的关键能力。

### 第四层：硬件互联层 — TraCT+CXL【恢复，标注远期】

**来源**：学术论文（2025.12），基于Dynamo框架实现
**2026状态**：论文级，CXL商用硬件尚未大规模可得

**核心方案**：
- CXL共享内存池：GPU通过CXL load/store直接访问共享内存，绕过NIC
- 两阶段同步：粗粒度CXL DMA + 细粒度load/store
- **实测**：TTFT降低9.8x，P99延迟降低6.2x

**关键逻辑**：软件层优化到极限后，硬件互联升级才能打破天花板。

**NPU启示（远期）**：昇腾HCCS互联架构天然适合做KV cache跨节点共享，是硬件差异化机会。待CXL商用成熟后，这一方向值得投入。

### 本页小结

采集效率优化是四层全栈问题：

```
Rollout服务层（ProRL Agent）：rollout生命周期独立管理，token一致性
       ↓ rollout需要推理引擎
推理调度层（Dynamo 1.0）：KV-aware routing，请求去已有缓存的节点
       ↓ 节点间KV需要共享
KV缓存共享层（LMCache）：P2P跨引擎KV cache共享，打破孤岛
       ↓ 软件优化到极限
硬件互联层（TraCT+CXL）：CXL共享内存绕过NIC，硬件级突破
```

**落地节奏**：Gate 1聚焦ProRL Agent（Rollout服务层），Gate 2-3逐步对标Dynamo+LMCache（推理基础设施层），远期跟踪TraCT+CXL（硬件层）。

---

## P8 | 挑战3：奖励评估与探索策略【重大修改：多层解法融合】

**标题**：挑战三：奖励可信吗？探索够高效吗？— 归因/建模/策略/Harness四层解法

**v4-revised核心修改**：
1. **保留v4的Harness即奖励核心洞察和reward hacking讨论**
2. **恢复SWE-PRM/AgentPRM（奖励建模层）和HiPER（探索策略层）**
3. **呈现从归因到策略的完整解法栈**

### 问题描述（与v4一致）

开放式Agent任务的正确性判定不像数学题有标准答案。三层风险叠加：
1. **奖励稀疏**：长轨迹中大部分步骤无信号
2. **Reward hacking**：并行探索越多越容易发现奖励漏洞
3. **Harness偏差**：harness定义的"成功"与真实用户需求之间的系统性偏差

### 第一层：归因算法层 — SALT

**与v4完全一致（P6已有详述）。**
- 轨迹图推算step-level advantage，plug-and-play
- 解决"归因太粗"问题

### 第二层：奖励建模层 — SWE-PRM / AgentPRM【恢复并升级】

**原始PPT定位**：SWE-PRM（步级纠错）
**v4状态**：删除（v4认为"Agent任务PRM基本不存在"）
**2026实际状态**：Agent PRM方向在2025-2026**爆发式增长**

**SWE-PRM**（学术，2025）：
- 核心：训练轻量PRM，在rollout中实时检测冗余/循环/未终止并做纠正
- 效果：SWE-bench Verified +10.6pp（从40.0%到50.6%）
- 机制：滑动窗口监测 + 错误模式分类法（taxonomy-guided）
- 关键价值：不等失败再纠正，**过程中实时干预**

**AgentPRM**（学术，2025.11）：
- 核心：同时捕获每步的**promise**（达成目标概率）和**progress**（步间依赖性）
- 训练方法：TD-based estimation + GAE，比传统MC方法更高效
- 效果：8x计算效率提升；step-level beam search引导Agent
- 关键价值：解决了传统PRM"独立评估每步"的局限，捕获步间依赖

**ToolPRMBench**（2026.1）：专门评测工具调用Agent PRM的benchmark出现，说明领域已有标准化评测趋势。

**PRM方向修正**：v4判断"Agent任务PRM基本不存在"在2026.Q1已经过时。虽然Agent PRM仍远不如数学PRM成熟，但方向明确且进展迅速。

### 第三层：探索策略层 — HiPER（分层plan-execute RL）【恢复】

**来源**：arXiv 2602.16165（2026.2）
**核心问题**：flat policy在长程任务中credit需要跨整条轨迹传播，导致优化不稳定、归因低效

**核心方案**：
- **分层架构**：高层planner分解子目标 + 低层executor执行并估计advantage
- **分层优势估计（HAE）**：在planning和execution两层分别做credit assignment
- 数学证明：HAE提供无偏梯度估计，且方差严格低于flat GAE

**效果**：
- ALFWorld 97.4%（+6.6% vs SOTA）
- WebShop 83.3%（+8.3% vs SOTA）
- 长程任务提升尤为显著

**跟进工作**：HiMAC（2026.3）、SkillRL（2026.2）等分层/递归策略方向活跃

**NPU启示**：双层模型编排调度——planner和executor可能需要不同规模的模型，需要灵活的多模型推理调度能力。

### 第四层：Harness工程层（与v4一致）

**Composer 2 + Anthropic Harness Patterns**：不变。

### SWE-bench Reward Hacking风险（与v4一致）

**风险分析**和**缓解策略**：与v4完全一致，保留不变。

### 关键洞察：确定性训练 + Harness质量 = 双轮驱动（与v4一致）

不变。

### 本页小结

奖励评估与探索策略是四层问题：

```
归因算法层（SALT）：从trajectory级细化到step级
       ↓ step级信号仍可能不准
奖励建模层（SWE-PRM/AgentPRM）：过程中实时检测纠正 + promise/progress双维度评估
       ↓ 奖励信号有了，探索效率呢？
探索策略层（HiPER）：分层plan-execute，中间子目标提供稠密反馈
       ↓ 训练方向对吗？
Harness工程层（Composer 2/Patterns）：harness质量决定训练方向
```

---

## P9 | 横切约束：精度与一致性

**与v4完全一致，不变。**

（包括AllReduce瓶颈数据、L8性能阈值、BF16 vs FP16发现、CANN Next预期管理等全部保留。）

---

## P10 | 我们的差异化：确定性RL训练

**与v4完全一致，不变。**

（包括确定性训练 + Harness质量 = 双轮驱动、竞争态势、技术路径等全部保留。）

---

## P11 | 行动路线图（v4-revised微调）

**标题**：分阶段路线图：1月跑通 -> 3月Demo -> 6月发布

**v4-revised微调**：技术映射总图更新，反映融合后的多层技术参照。

### Gate 1：1个月后（~2026年5月初）

**与v4完全一致，不变。**（ProRL Agent单机跑通，4项量化check。）

### Gate 2：3个月后（~2026年7月）

**与v4完全一致，不变。**（SWE-bench Agent RL全流程跑通 + 参考harness + NPU-hour数据。）

### Gate 3：6个月后（~2026年10月）

**与v4基本一致，微调**：

1. 可复现的SWE-bench Agent RL benchmark结果，能对外发布
2. batch-invariant推理集成进vllm-ascend推理流水线
3. RL关键路径算子精度基线完整覆盖
4. 多维评估信号集成（SWE-bench基础reward + 代码质量分 + diff最小化）
5. **【v4-revised新增】vllm-ascend推理调度能力对标Dynamo 1.0核心特性**：KV-aware routing + prefix-aware调度。不求完全对标，但Agent多轮rollout的KV复用需要基本支持
6. （可选）与vLLM/SGLang社区联合发布昇腾确定性RL训练结果

### 技术映射总图【更新】

```
━━━ 基础设施层已有能力（实线）━━━
训练侧  <-- MindSpeed RL基础RL训练（GRPO，384卡，671B规模）
              ! 目前是reasoning RL，不是Agent RL
推理侧  <-- vllm-ascend推理引擎
              ! 标准LLM serving，Agent rollout推理特征未优化
              ! 对标目标：Dynamo 1.0 + LMCache（GPU生态事实标准）
精度侧  <-- 5个RL关键路径算子CANN/CUDA精度比对档案

--- 前沿技术层待建能力（虚线，按优先级排序）---
【优先级1】ProRL Agent Rollout-as-a-Service昇腾适配
          -> 公共底座，补MindSpeed RL缺的rollout和环境交互
【优先级1b】Dynamo KV-aware routing + LMCache P2P共享 能力对标
          -> vllm-ascend推理调度层的核心升级方向
          -> Agent多轮rollout的KV复用效率直接影响采集成本
【优先级2】确定性RL训练（batch-invariant算子 + 精度基线）
          -> 差异化卖点，关键瓶颈AllReduce（阈值30%）
【优先级2b】SWE-bench参考harness实现
          -> 确保客户用得起来，与确定性训练形成双轮驱动
【优先级3】DAPO/DISPO/CISPO演进链选型 + Forge异步框架评估
          -> Gate 1-2用DAPO（稳定），Gate 3评估DISPO（最优效果）
【优先级3b】SkyRL-Agent框架能力对比评估
          -> 与ProRL Agent在Agent RL场景互补/竞争关系待厘清
【优先级4】CSO/SALT归因 + SWE-PRM/AgentPRM奖励建模 NPU验证
          -> Agent PRM方向快速发展，跟踪评估
【优先级4b】HiPER分层策略技术评估
          -> 分层plan-execute对长程Agent任务价值明确，待模型推理调度能力就绪
【远期】TraCT+CXL硬件互联方向
          -> HCCS天然适配点，待CXL商用成熟
```

### 资源需求（与v4一致）

| 方向 | 人力 | 时间 | 技能栈 |
|------|------|------|--------|
| ProRL Agent Rollout-as-a-Service昇腾适配 | 5-8人 | 3个月PoC，6个月可用 | 推理引擎、容器编排、HTTP服务 |
| 确定性RL训练（batch-invariant + 精度基线） | 3-5人 | 3-4个月算子集成，6个月全路径覆盖 | 算子开发、CANN内核、精度调优 |
| harness参考实现 | 不需额外大量人力 | Gate 2阶段 | ProRL Agent已含SWE-bench harness集成 |
| 总计 | **8-13人** | 并行推进 | |

---

## P12 | 收束：竞争定位与客户价值

**与v4完全一致，不变。**

（包括竞争定位、目标客户画像、内部版vs外部版叙事、软化过渡话术等全部保留。）

---

## 附：v4 -> v4-revised修改对照表

| 维度 | v4 | v4-revised | 修改理由 |
|------|----|-----------|---------| 
| P5技术地图 | 6项前沿技术 | **15+项技术，按栈层分层** | 恢复"每个挑战在不同栈层都有解法"的结构价值 |
| P5基础设施对标 | 未提Dynamo/LMCache | **Dynamo 1.0+LMCache列为推理侧核心对标** | 两者已达生产级，是GPU生态事实标准 |
| P5优先级排序 | 无1b优先级 | **新增优先级1b：Dynamo+LMCache能力对标** | 推理基础设施对标与ProRL Agent适配同等重要 |
| P6挑战1 | CSO+SALT两技术 | **PRIME-RL(系统)+SkyRL/A-3PO(框架)+DAPO链(算法)+CSO/SALT(归因)四层** | 恢复多层防线结构 |
| P6 DAPO呈现 | CISPO单独列出，DAPO在P2提及 | **DAPO->DISPO->CISPO演进链完整呈现** | DAPO不是被CISPO替代，而是演进链的起点 |
| P7挑战2 | ProRL Agent + Forge/CISPO | **ProRL Agent(Rollout)+Dynamo(调度)+LMCache(缓存)+TraCT(硬件)四层** | 恢复"推理调度-缓存-硬件"全栈 |
| P7 Dynamo | 技术地图表格一行 | **恢复为重点技术，含2026.3生产数据** | Dynamo 1.0已生产发布，不应降级 |
| P7 LMCache | 删除 | **恢复，含2026生产级数据** | P2P已从实验升级为生产 |
| P7 TraCT+CXL | 删除 | **恢复，标注远期方向** | HCCS天然适配，值得远期跟踪 |
| P8挑战3 | SALT + Harness洞察 | **SALT(归因)+SWE-PRM/AgentPRM(建模)+HiPER(策略)+Harness(工程)四层** | 恢复奖励建模和探索策略两个关键层次 |
| P8 SWE-PRM | 删除（"Agent PRM不存在"） | **恢复并升级为PRM方向（含AgentPRM）** | 2025-2026 Agent PRM方向爆发，v4判断已过时 |
| P8 HiPER | 删除 | **恢复，含2026.2最新数据** | 分层策略填补v4探索策略层空白 |
| P11技术映射 | 5个优先级 | **9个优先级层级（含远期）** | 技术面扩展后映射更完整 |
| P11 Gate 3 | 无推理调度对标 | **新增Dynamo核心特性对标** | Gate 3时vllm-ascend应具备基本KV-aware routing |
| 总页数 | 12页 | **12页不变** | 在12页框架内做内容融合，不膨胀页数 |
| P1-P4, P9-P10, P12 | - | **完全不变** | 这些页面的内容和逻辑不受技术融合影响 |

---

## 附：融合版技术全景（一张图）

```
                    Agentic RL 后训练全栈技术地图
                    
挑战1: 归因/新鲜度          挑战2: 采集/有状态         挑战3: 评估/探索
========================  ========================  ========================
系统架构层                 Rollout服务层              归因算法层
  PRIME-RL [生产]           ProRL Agent [框架]**       SALT [论文]
  去中心化异步千卡           Rollout-as-a-Service      轨迹图step-level
  INTELLECT-3验证           第一优先级                 plug-and-play
------------------------  ------------------------  ------------------------
训练框架层                 推理调度层                  奖励建模层
  SkyRL-Agent [框架]        Dynamo 1.0 [生产]**       SWE-PRM [论文->框架]
  多轮Agent RL框架           KV-aware routing          过程实时纠错
  off-policy correction     20x TTFT提升              AgentPRM [论文]
  A-3PO [论文]              LMCache [生产]**           promise+progress
  staleness-aware 1.8x     P2P KV cache共享           ToolPRMBench评测
------------------------  ------------------------  ------------------------
训练算法层                 硬件互联层                  探索策略层
  DAPO->DISPO->CISPO       TraCT+CXL [论文]          HiPER [论文]
  [框架] 演进链              CXL共享内存               分层plan-execute
  Clip-Higher防熵崩塌       TTFT降9.8x                ALFWorld 97.4%
  CISPO快2x但有崩塌风险     远期:HCCS适配点           WebShop 83.3%
------------------------  ------------------------  ------------------------
归因算法层                                           Harness工程层
  CSO [论文]                                          Composer 2 [产品]
  关键步骤检测                                         高保真仿真RL
  SALT [论文]                                         Harness Patterns
  (同上)                                              5种Agent模式

========================  ========================  ========================
                横切约束：训推一致性/确定性计算 [原型中]
                AllReduce瓶颈: 20-30% / 40-50%  阈值: 30%以内
========================  ========================  ========================
                基础设施层已有能力
                MindSpeed RL (reasoning RL)  |  vllm-ascend  |  5算子精度比对
```

**说明**：[生产] = 已有生产部署，[框架] = 有可运行框架/代码，[论文] = 论文级验证。** = 核心对标目标。
