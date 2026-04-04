# Agentic RL 后训练洞察 — PPT结构设计 v3

> 基于v2 + 虚拟洞察汇报全6轮领导质疑反馈 + L8实战数据 + L1分阶段gate机制
> 范围：Agent System（电脑端Agent，如coding/browsing/工具使用）
> 团队定位：帮助客户在NPU上开箱即用地完成Agent后训练任务
> **核心修改原则**：压缩业界综述（P5-P13），扩大"我们做什么"（P14-P16），区分"已有能力"和"待建能力"

---

## 页码总览与逻辑流

```
P1   Agent时代已来              ← 定调：产品/开源/资本三维爆发（数据点全部修正）
P2   Agent RL正在成为主流       ← 【新增】补上v2缺环：从"Agent落地"到"Agent RL落地"的多案例论证
P3   为什么必须是Agentic RL     ← 动因：harness→模型能力迁移 + RL不可替代性（措辞修正）
P4   三大结构性变化 → 三大挑战  ← 核心推导页（标注"我们的分析框架"）
P5   技术地图                   ← 导航：3挑战 × 业界解法 × 成熟度/局限/NPU状态
P6   挑战①归因：CSO + SALT      ← 压缩为一页，标注前提条件和适用范围
P7   挑战②采集：ProRL Agent     ← 第一优先级单独展开，CISPO/Forge压缩为辅助
P8   挑战③评估：Harness即奖励   ← Composer 2 + Harness Patterns压缩为一页
P9   横切约束：精度与一致性      ← 【扩展】L8实战数据融入，从"约束"升级为"差异化基础"
P10  我们的差异化：确定性RL训练  ← 【新增】从横切约束提炼出的独立卖点页
P11  行动路线图                 ← 【重新设计】分阶段gate + 资源需求 + 第一个交付物
P12  竞争定位与客户价值         ← 【新增】"让国内客户能做起来" + 客户画像
```

**逻辑链**：现象(P1) → Agent RL成为主流(P2) → 为什么RL(P3) → 变化→挑战(P4) → 导航(P5) → 三挑战精炼(P6-P8) → 横切约束(P9) → 差异化卖点(P10) → 行动路线图(P11) → 竞争定位(P12)

**与v2的关键变化**：
- 新增P2补逻辑缺环，新增P10差异化卖点页，新增P12竞争定位页
- P5-P8从原来的9页（P5-P13）压缩为4页
- P9-P12从原来的2页（P14-P15）扩展为4页
- 总页数从15页变为12页，"我们做什么"的比重从13%提升到33%

---

## P1 | 定调层：Agent时代已来

**标题**：智能涌现：Agent从概念验证走向规模落地

**核心信息**：产品、开源、资本三维同步爆发。全部数据点经过校验修正。

**关键数据点（2026.Q1，已修正）**：
- Cursor Composer 2（2026.3）：基于Kimi K2.5做RL后训练，在部分benchmark上超越Opus 4.6；GPT-5.4在Terminal-Bench达75.1分，竞争格局激烈
  - **修正说明**：v2仅提Composer 2超Opus，被L1指出cherry-picking。现并列GPT-5.4数据，叙事从"谁最强"转为"后训练战场已开"
- OpenAI Codex CLI开源（Apache 2.0，约6.7万星）
  - **修正说明**：v2引用7.3万星，L4指出不准。已校准至最新数据
- 据The Information报道，Anthropic领导层**讨论**在RL环境上的年度投入超$1B
  - **修正说明**：v2措辞暗示"已投入"，L5指出原文是"leaders discussed"。明确标注为讨论中规模
- 广义训练算力（预训练+后训练）占总算力约40%，其中后训练占比持续攀升
  - **修正说明**：v2表述为"后训练算力占40%"，L2指出口径偷换。修正为分层表述

**底部过渡句**：Agent产品在爆发，但爆发背后的驱动力是什么？——下一页回答：RL驱动的Agent正在成为主流。

---

## P2 | 【新增】过渡层：Agent RL正在成为主流

**标题**：从SFT到RL：Agent能力提升的方法论正在收敛

**核心信息**：补上v2的逻辑缺环——P1证明"Agent在落地"，但Agent落地不等于Agent RL落地。本页用多案例证明"RL驱动的Agent正在成为主流趋势"。

> **v3新增原因**：L5在Phase 2指出P1→P2存在逻辑跳跃，演讲者在回应中承认"推导链缺了一环"。

### 多案例论证矩阵

| 公司/项目 | Agent类型 | RL角色 | 关键结果 | 时间 |
|-----------|----------|--------|---------|------|
| Cursor Composer 2 | Coding Agent | 核心训练方法（基于K2.5做大规模RL） | 部分benchmark超Opus 4.6 | 2026.3 |
| OpenAI Codex | Software Engineering Agent | SFT+RL混合，RL做闭环优化 | Terminal-Bench 75.1% | 2026.3 |
| DeepSWE | 开源SE Agent | 纯RL训练（GRPO），无SFT数据 | SWE-bench Verified 40%+ | 2026.2 |
| Google Agent | 多模态Agent | 全流程引入RL评估 | 内部产品集成 | 2026.Q1 |
| NVIDIA ProRL Agent | 通用Agent RL框架 | Rollout-as-a-Service架构 | SWE-bench 9.6→18.0% | 2026.3 |

**关键观察**：
1. 头部厂商**全部**在Agent训练中引入RL——区别仅在于RL在整个流水线中的权重
2. 趋势方向一致：SFT做基础能力注入，RL做闭环优化——**混合范式是主流**，纯SFT已不够
3. 开源社区（DeepSWE）验证了即使不依赖顶级base model，RL也能显著提升Agent能力

**措辞校准**：RL不是"唯一路径"，而是Agent能力提升中**不可或缺的关键环节**。
> **修正说明**：v2将RL定位为"唯一路径"，L2指出过于绝对。修正后更准确反映SFT+RL混合的主流范式。

**DeepSWE数据说明**：DeepSWE在SWE-bench Verified上达40%+，此数据来自论文报告，具体口径为"resolved rate"。
> **修正说明**：L2在Phase 2提醒数据口径需严谨，此处明确标注评测指标定义。

**P1→P2承接**：P1说"Agent产品在爆发"，P2回答"这些成功的Agent产品背后，RL训练正在成为标配方法论"。

---

## P3 | 动因层：为什么必须是Agentic RL

**标题**：从Harness到模型：Agent能力的迁移方向锁定了RL后训练

**与v2的区别**：叙事结构基本保持，但措辞从"唯一路径"全面修正为"不可或缺的关键环节"。P2已经用多案例证明RL是主流，P3进一步解释**为什么RL不可或缺**。

### 叙事结构

**上半页：双引擎演进——Agent = Model + Harness**

```
2024（胖Harness+瘦模型）          2026（瘦Harness+胖模型）
┌──────────────────┐              ┌──────────────────┐
│ Harness承担：      │              │ 模型内化：        │
│ · 上下文管理       │    ──→       │ · self-summarize  │
│ · 工具选择逻辑     │              │ · tool use规划    │
│ · 错误恢复         │              │ · 自主纠错        │
│ · 任务分解         │              │ · 多步规划        │
│                   │              │                   │
│ 模型只做：生成文本  │              │ Harness只做：      │
│                   │              │ 安全边界+API代理   │
└──────────────────┘              └──────────────────┘
```

**关键实证**：Cursor Composer 2通过RL训练self-summarization，模型学会从200K token压缩到~1K token，compaction error降50%。

**背景补充**（回应L8 Phase 3质疑）：Composer 2的base model是Kimi K2.5（MoE），在B300上做continued pretraining再做大规模RL。这说明RL路线的前提是"足够强的base + 算力集群"——不是任何base model上做RL都能有效。

**下半页：这条迁移路径RL不可或缺**

- SFT能教模仿，不能教从错误中学习——Agent状态空间太大，演示覆盖不了
- DPO能教对齐，不能教与环境交互——Agent奖励来自环境验证，不来自人类偏好
- **RL是不可或缺的关键环节**：环境反馈+闭环探索+从错误中学习 → 系统性地将harness能力迁移到模型参数中
- **但不是唯一方法**：业界主流是SFT做基础能力注入 + RL做闭环优化的混合范式

**P2→P3承接**：P2说"Agent RL正在成为主流"，P3回答"为什么RL不可或缺？——因为harness→模型能力迁移只有通过环境闭环才能实现"。

---

## P4 | 核心推导层：三大结构性变化 → 三大挑战

**标题**：范式跃迁：Agent RL的三大结构性变化，直接产生三大未解挑战

**关键修正**（回应L2 Phase 4质疑）：

> **框架标注**：以下三分法是**我们的工程分析框架**（按RL循环的评估→归因→采集三环节切分），不是学术界的标准分类。学术界有其他切法，如GiGPO将归因和采集合并处理，微软按"稳定性/泛化性/可扩展性"切分。我们选择此切法的原因是：它直接对应RL循环的三个可独立优化的工程环节，便于技术落地时分而治之。

### 推导逻辑

**左半页：三个结构性变化**

| # | LLM RL | Agent RL | 变化本质 |
|---|--------|----------|---------|
| ① | 轨迹=几十到几百token | 轨迹=几千到几万token，跨多轮工具调用 | **轨迹长度量级跃升** |
| ② | 每条轨迹秒级生成 | 每条轨迹分钟级（环境执行、API调用） | **单条经验成本爆炸** |
| ③ | 正确性可验证（数学/代码有标准答案） | 正确性模糊多标准（开放任务无唯一正解） | **评估从确定变为模糊** |

**右半页：三大变化直接对应三大挑战**

```
变化① 轨迹长度跃升 ──→ 挑战① 归因（Credit Assignment）
                        关键步骤被淹没在长轨迹中

变化② 单条经验成本爆炸 ──→ 挑战② 采集（Experience Collection）  
                            每条rollout代价高，必须让每条都"值"

变化③ 评估从确定到模糊 ──→ 挑战③ 评估（Reward Evaluation）
                            奖励信号本身可能不忠实、可被游戏
```

**挑战间耦合说明**（回应L2 Phase 4"内部矛盾"质疑）：
> 三大挑战并非完全正交。例如CSO解决归因需要额外PRM和expert model推理，本身加剧采集成本；CISPO提升采集效率但引入entropy崩塌风险影响评估稳定性。技术地图（P5）标注了这些cross-cutting trade-off。

**实战优先级说明**（回应L8 Phase 4质疑）：
> 上述排序是因果链顺序（变化→挑战的映射顺序），不是落地优先级。L8从实战经验指出：评估应排第一——"没有可靠的reward，一切都是空中楼阁"。在实际技术路线中，我们将评估可靠性（含横切约束的精度一致性）作为第一保障。

---

## P5 | 技术地图

**标题**：技术地图：三大挑战 × 业界前沿解法 × 成熟度与NPU状态

**核心修正**（回应L2/L3/L8 Phase 4联合质疑）：
1. 每项技术标注**成熟度**（论文级/框架级/工程实践级）
2. 每项技术标注**已知局限**
3. 每项技术标注**NPU适配状态**（已适配/原型中/未适配）
4. 区分**已有能力**（实线）和**待建能力**（虚线）
5. 标注挑战间的**cross-cutting trade-off**

| 挑战 | 前沿解法 | 成熟度 | 已知局限 | NPU适配 | 对应页 |
|------|---------|--------|---------|---------|--------|
| ①归因 | CSO（关键步骤检测+验证） | 论文级（arXiv 2602.03412） | 需PRM（Agent任务PRM不存在）；额外计算开销高，加剧采集成本 | 未适配 | P6 |
| ①归因 | SALT（轨迹图step-level归因） | 论文级（Amazon 2025-2026） | 依赖轨迹间状态共享，高随机环境退化；可扩展性未验证 | 未适配 | P6 |
| ②采集 | ProRL Agent（Rollout-as-a-Service） | 框架级（NVIDIA开源，2026.3） | 纯CUDA实现，依赖NeMo Gym；HTTP通信延迟；Singularity容器 | **未适配（第一优先级）** | P7 |
| ②采集 | Forge+CISPO（异步off-policy RL） | 框架级（MiniMax，2026.3） | CISPO后期entropy崩塌（DISPO论文证实）；长轨迹importance weight累积衰减 | 未适配 | P7 |
| ③评估 | Composer 2（高保真仿真harness中RL训练） | 产品级（Cursor内部） | 依赖强base model+B300集群；sandbox≠production；嵌套归因难 | 不适用（产品逻辑参考） |  P8 |
| ③评估 | Harness Patterns（5种Agent模式） | 工程实践级（Anthropic博客） | 无代码/无benchmark/无ablation，不可直接复现 | 不适用（设计参考） | P8 |
| 横切 | 训推一致性/确定性计算/算子验证 | **原型中（我们的投入方向）** | batch-invariant算子原型阶段；精度基线仅覆盖5个算子 | **原型中** | P9-P10 |

**已有能力基线**（实线）：
- MindSpeed RL：基础RL训练（GRPO）在384卡NPU集群上跑通，支持DeepSeek-R1-671B规模
- vllm-ascend：推理引擎NPU适配，sleep mode已初步解决，hybrid engine刚打通
- 5个RL关键路径算子CANN vs CUDA精度比对档案

**待建能力**（虚线）：以上六项前沿技术的NPU适配均为待建。

---

## P6 | 归因挑战：CSO + SALT（压缩为一页）

**标题**：挑战一：长轨迹中谁是关键决策者？——两条技术路线及其适用条件

> **v2→v3变化**：从两页压缩为一页，删除纯学术细节，增加适用条件和前提链条成本分析。

### CSO（Critical Step Optimization）

- 来源：arXiv 2602.03412（2026.2）
- 核心：用PRM识别关键步骤候选（仅16%步骤是关键的），用expert model提出替代方案，只在验证通过的关键步骤上做DPO
- 效果：8B模型在**GAIA-Text-103**上追平GPT-4.1（注：GPT-4.1在GAIA上为中游水平，GPT-4.5和Claude表现更强）
  - **修正说明**：v2表述暗示"8B追平顶级模型"，L2指出选择性呈现。现补充GPT-4.1在该任务上的定位
- **前提链条成本**（回应L2+L8 Phase 5追问）：step-level标注数据 → 训PRM → CSO执行。Agent任务PRM基本不存在，冷启动成本可能超过直接GRPO全量训练
- **适用条件**：已有PRM或可低成本获得step-level信号的场景（如数学推理有Qwen Math PRM）
- NPU适配：未适配

### SALT（Step-level Advantage via Trajectory Graph）

- 来源：Amazon（2025-2026）
- 核心：将同一prompt的多条轨迹建模为有向图，通过共享状态节点推算每步advantage。plug-and-play，不需要额外PRM或expert model
- **优势**：计算开销几乎可忽略，可直接叠加在GRPO上
- **局限**：依赖轨迹间有足够状态共享。高随机环境（同一操作不同结果）下图结构退化。原始实验在WebShop/ALFWorld（规则化环境），SWE-bench等真实SE任务的可扩展性未验证
- NPU适配：未适配

**归因小结**：CSO精度高但前提重（需PRM），SALT轻量但场景受限（需状态共享），两者互补。当前NPU上均未验证。归因方向的投入优先级低于采集和横切约束。

---

## P7 | 采集挑战：ProRL Agent为核心（第一优先级）

**标题**：挑战二：每条轨迹都很贵——Rollout-as-a-Service是公共底座

> **v2→v3变化**：ProRL Agent从半页升级为整页（第一优先级），CISPO/Forge压缩为辅助参考。

### ProRL Agent — Rollout-as-a-Service（NVIDIA，2026.3，**第一优先级**）

- 来源：arXiv 2603.18815
- 核心创新：
  - 将rollout生命周期（环境初始化、工具执行、奖励评分）作为独立HTTP服务从训练循环解耦
  - **Token-in/token-out一致性保证**：rollout产生的token ID直接传给trainer，全流程不做re-tokenization，避免隐性off-policy偏差
- 效果：Qwen3-8B在SWE-Bench Verified上从9.6%提升到18.0%
- **为什么是第一优先级**（演讲者在Phase 5回答L2"只选一个"时的三条理由）：
  1. 训推解耦+token一致性是**所有Agent RL方法的公共基础设施**——不管上层用CSO还是CISPO还是GRPO，都需要这个底座
  2. NVIDIA刚开源一个月，快速跟进可在NPU生态中建立先发优势
  3. Rollout-as-a-Service的核心是推理引擎+HTTP服务，与vllm-ascend技术栈复用度最高
- **已知挑战**（回应L6/L7 Phase 5-6追问）：
  - 纯CUDA实现，NeMo Gym集成，昇腾零移植
  - Singularity容器运行时 vs 我们的Docker环境：需要sandbox层适配
  - HTTP round-trip延迟：Agent rollout涉及几十次工具调用，每次一次round-trip，部署拓扑需co-located
  - vllm-ascend对rollout动态batch和长上下文管理的支持程度待验证
- **L7背书**（Phase 6）：ProRL Agent适配同时推进"矛"（Agent推理优化）和"盾"（RL训练底座），一个方向解决两个问题
- NPU适配：**未适配，列为第一优先级工作项**

### 辅助参考：Forge + CISPO（MiniMax，2026.3）

- CISPO（Clipped IS Policy Optimization）：比DAPO快2x达到同等性能（Qwen2.5-32B，50%训练步数）
- **已知重大风险**：
  - 后期entropy急剧下降导致性能崩溃（DISPO论文证实）
  - 100步以上Agent长轨迹中importance weight累积衰减（单步ratio 0.95，100步后仅剩0.006）——长轨迹训练信号几乎被weight到零
  - DISPO的修复方案（动态entropy coefficient）在Agent场景下未验证
- Forge框架解耦训练框架与Agent scaffold，但需要Firecracker级sandbox调度能力（我们不具备）
- NPU适配：未适配

---

## P8 | 评估挑战：Harness即奖励（压缩为一页）

**标题**：挑战三：你优化的目标本身可信吗？——Harness is the New Dataset

> **v2→v3变化**：从三页（P11-P13）压缩为一页，聚焦核心洞察而非技术细节。

### 核心洞察

开放式Agent任务的正确性判定不像数学题有标准答案。三层风险叠加：
1. **奖励稀疏**：长轨迹中大部分步骤无信号
2. **Reward hacking**：并行探索越多越容易发现奖励漏洞
3. **Harness偏差**：harness定义的"成功"与真实用户需求之间的系统性偏差

**关键判断**：Harness的奖励定义决定了Agent学到什么。竞争优势不在数据量，在harness捕获的轨迹质量。

### 业界实践参考

**Composer 2（Cursor）**：
- 在**高保真仿真harness**（Firecracker VM隔离sandbox）中做RL训练——same tools, same prompts, same system message
  - **措辞修正**：v2表述为"production harness"，L2指出sandbox≠production，修正为"高保真仿真harness"
- RL-trained self-summarization：模型学会上下文压缩，compaction error降50%（还有50%未解决）
- **嵌套归因挑战**（L8 Phase 6指出）：compaction-in-the-loop RL的reward信号要同时奖励agent response和中间的self-summary，credit assignment纠缠
- 参考价值：验证了"在接近真实环境中训练"的路线有效性；但其前提（强base model + B300集群）我们不具备

**Anthropic Harness Patterns**：
- 5种生产级Agent模式：Single Agent / Prompt Chaining / Router / Orchestrator-Worker / Swarm
- **成熟度说明**：工程博客总结，无代码/无benchmark/无ablation，不可直接复现——与CSO等论文级方案成熟度不同
  - **修正说明**：L5在Phase 6指出与CSO并列但成熟度不对等
- 参考价值：理解harness设计模式→理解Agent RL的训练负载特征

---

## P9 | 横切约束：精度与一致性（扩展，融入L8实战数据）

**标题**：横切约束：所有优化手段的"宪法底线"——精度与一致性

> **v2→v3变化**：从概念性列表升级为有L8一手实战数据支撑的深度分析页。融入L8在910B上的GRPO精度分叉经验。

### 为什么精度一致性在RL中比预训练更致命

RL训练中importance ratio的计算对数值差异极度敏感。**L8实战数据**（Phase 6，910B上跑GRPO）：
- CANN的LayerNorm算子和CUDA的数值结果在第5-6位有效数字不同
- Reward曲线前200步与GPU几乎一致，**300步后开始分叉，500步差距达2-3个百分点**
- 根因定位花了一周：LayerNorm的reduction order不同
- 结论：预训练能容忍的微小精度差异，在RL训练中会被importance ratio放大为系统性偏移

### 副作用来源与应对

| 副作用来源 | 具体风险 | 我们的应对 | 当前状态 |
|-----------|---------|-----------|---------|
| 异步引入staleness | off-policy importance ratio偏移 | 异步RL精度保证算法 | 方向性研究 |
| 训推路径不一致 | 同一输入不同logits→ratio有方向性bias | **训推一致性/批不变性** | **原型阶段**（LayerNorm/Softmax两算子验证） |
| 低精度推理(mxfp8/mxfp4) | 策略分布尾部行为改变 | **4bit精度恢复算法** | 方向性研究 |
| 浮点非确定性 | 同checkpoint两次运行不同结果→调参变玄学 | **bf16/低精度确定性计算** | **原型阶段** |
| 跨硬件部署 | NPU vs GPU浮点实现差异累积 | **Ascend精度基线** | **5个算子比对完成**，LayerNorm和AllReduce差异最大 |
| MoE训推一致性 | expert routing不一致导致推理行为偏移 | router replay机制研究（参考Cursor） | 方向性研究 |

### BF16 vs FP16的关键发现

最新研究（arXiv 2510.26788）证明BF16的mantissa位数不够是训推不一致的主要根因，切FP16可消除大部分mismatch。但昇腾训练流水线多处hardcode BF16，FP16的dynamic range小导致大模型训练overflow风险。改造是系统级工作，尚未启动。

### CANN Next的预期管理

950PR的CANN Next提升了编程兼容性（CUDA-like thread blocks/warps），但**L6确认BF16/FP16算子行为与当前CANN差异不大**。编程兼容性提升不等于数值精度对齐。**不能依赖下一代硬件自动解决精度问题**，910C上的精度对齐工作必须独立推进。

---

## P10 | 【新增】我们的差异化：确定性RL训练

**标题**：差异化方向：如果昇腾能做到Bitwise Reproducibility

> **v3新增原因**：Phase 6中L8从最严厉质疑者转为差异化方向的最有力背书者，L1在此基础上给出原则性支持。确定性RL训练从"横切约束的一个子项"提升为独立的差异化卖点页。

### 为什么这是差异化

**L8背书**（Phase 6）：
> "做RL训练的人——包括我自己——极度痛恨不可复现性。你调一个超参，reward上去了，你不知道是超参的功劳还是随机性的功劳。如果昇腾能做到bitwise reproducibility——同样的代码、同样的数据、跑两次结果完全一样——这在RL领域是一个非常有吸引力的卖点。"

**竞争态势**：
- GPU阵营：SGLang+vLLM联合的batch-invariant-ops库在**Qwen3-1.7B**上演示了bitwise consistent RL训练——但仅在小模型上，大规模模型确定性训练仍是开放问题
- 昇腾机会：如果能先于CUDA生态实现**大规模确定性RL训练**，这是GPU阵营目前做不到的差异化

### 技术路径

1. **batch-invariant算子昇腾适配**（与vLLM/SGLang社区合作）
   - 社区有CUDA实现的batch-invariant-ops库
   - 我们贡献昇腾backend，换取社区支持和生态存在感（**L8建议**）
   - 挑战：CANN AllReduce reduction order与CUDA不同，非简单backend替换
   - 状态：原型阶段，LayerNorm和Softmax两个算子验证方案完成

2. **RL关键路径算子精度基线**
   - 目标：全部关键算子（LayerNorm、Softmax、AllReduce、FlashAttention、Embedding等）CANN vs CUDA数值比对档案
   - 当前：5个算子完成比对，LayerNorm和AllReduce差异最大
   - 下一步：扩展覆盖面 + 制定修复优先级

3. **BF16→FP16系统级评估**
   - 可消除大部分训推mismatch
   - 但改造工程量大（hardcode BF16的流水线环节多），需评估overflow风险
   - 状态：未启动

### L8认证的差异化逻辑

L8在整场汇报中提供了三个关键贡献：
1. **痛点验证**：910B上LayerNorm精度分叉导致reward曲线偏移的一手数据——证明精度问题是真实的工程痛点
2. **方向背书**：RL领域对可复现性的需求"真实且迫切"——证明差异化卖点有市场需求
3. **路径建议**：与vLLM/SGLang社区合作batch-invariant-ops适配——低成本高收益的切入路径

---

## P11 | 行动路线图（重新设计）

**标题**：分阶段路线图：1月跑通 → 3月Demo → 6月发布

> **v2→v3变化**：v2的P15只有概念性的技术映射图，缺具体时间线和交付物。v3基于Phase 6领导团自发形成的三阶段gate机制重新设计。

### Gate 1：1个月后（~2026年5月初）— ProRL Agent单机跑通

**交付物**（L4 Phase 6确认的check项）：
1. ProRL Agent的rollout服务在**910C单机**上启动并完成**一条完整的SWE-bench rollout**——不求性能，先求跑通
2. Singularity→Docker sandbox适配方案设计文档

**通过条件**：L4来check。通过后支持扩到5-8人核心团队。

**并行启动**：
- 确定性RL训练方向：batch-invariant算子昇腾适配POC（3-5人独立团队）
- 精度基线扩展：从5个算子扩展RL关键路径全覆盖

### Gate 2：3个月后（~2026年7月）— 可展示的客户Demo

**交付物**（L3 Phase 6要求）：
1. 在910C集群上用ProRL Agent架构跑通**SWE-bench Agent RL训练全流程**
2. 展示训推一致性和token-level精度对齐的差异化
3. **必须是可以带客户看的东西**，不是内部benchmark

**Demo定义建议**：
- 可视化的RL训练过程（reward曲线、解题成功率随训练步数提升）
- SWE-bench解题成功率的昇腾 vs GPU对比
- 训推一致性的A/B对比展示（同一checkpoint两次训练结果一致性）

### Gate 3：6个月后（~2026年10月）— 可发布Benchmark

**交付物**：
1. 可复现的SWE-bench Agent RL benchmark结果，能对外发布
2. batch-invariant推理集成进vllm-ascend推理流水线
3. RL关键路径算子精度基线完整覆盖
4. （可选）与vLLM/SGLang社区联合发布昇腾确定性RL训练结果

### 资源需求

| 方向 | 人力 | 时间 | 技能栈 |
|------|------|------|--------|
| ProRL Agent Rollout-as-a-Service昇腾适配 | 5-8人 | 3个月PoC，6个月可用 | 推理引擎、容器编排、HTTP服务 |
| 确定性RL训练（batch-invariant + 精度基线） | 3-5人 | 3-4个月算子集成，6个月全路径覆盖 | 算子开发、CANN内核、精度调优 |
| 总计 | **8-13人** | 并行推进，技能栈不完全重叠 | |

> **说明**：两个方向可并行——前者偏推理引擎侧，后者偏算子精度侧。

### 技术映射总图（已有能力 vs 待建能力）

```
━━━ 已有能力（实线）━━━
挑战②采集 ←── MindSpeed RL基础RL训练（GRPO，384卡，671B规模）
              vllm-ascend推理引擎（sleep mode已解决，hybrid engine刚打通）
横切约束  ←── 5个RL关键路径算子CANN/CUDA精度比对档案
              L8的910B RL实战经验和精度数据

┈┈┈ 待建能力（虚线，按优先级排序）┈┈┈
【优先级1】ProRL Agent Rollout-as-a-Service昇腾适配
          → 公共底座，所有上层RL算法都受益
【优先级2】确定性RL训练（batch-invariant算子 + 精度基线扩展）
          → 差异化卖点，GPU阵营大规模场景未解决
【优先级3】CISPO/Forge异步RL框架评估
          → 待CISPO稳定性问题有方案后再投入
【优先级4】CSO/SALT归因技术NPU验证
          → 待Agent PRM生态成熟后再投入
```

---

## P12 | 【新增】竞争定位与客户价值

**标题**：竞争定位重构：不是对标NVIDIA，而是让国内客户能做起来

> **v3新增原因**：演讲者在Phase 3将竞争对标从NVIDIA转向"让国内客户能做起来"，获L1认可。这个reframing需要在PPT中显式呈现。

### 竞争定位

**传统定位**（已过时）：910C vs H100单卡性能对标
- 910C推理性能约为H100的60%
- 纯FLOPS比拼我们没有优势

**新定位**：让国内客户的Agent RL从"不能做"变为"能做"
- **供应链现实**：国内大量客户受约束，H100不是想买就能买
- **真正的对标不是NVIDIA，而是"不做"**：客户的替代选项不是H100，而是放弃Agent RL
- **差异化不在单卡FLOPS，在全栈保障**：
  - 训推一致性/确定性计算（GPU阵营大规模场景未解决）
  - RL训练可复现性（RL从业者极度痛恨不可复现性——L8认证）
  - 全栈优化弥补裸性能差距（通信优化、算子精度、精度一致性）

### 目标客户画像

- **第一目标**：国内做Agent产品的公司——有模型、有Agent场景、受H100供应约束
- **第一场景**：Software Engineering Agent的RL训练
  - 选择理由：评估相对标准化（SWE-bench）、开源benchmark丰富、学术界关注度高
- **差异化卖点验证**（L1+L5 Phase 6建议）：需与2-3个目标客户做用户调研，确认"可复现性"卖点的付费意愿

### 收束语

Agent时代的竞争，不在谁的模型更大，而在谁能让Agent**更快地从做事中学会做事**。我们的价值主张不是"和NVIDIA比谁的卡更快"，而是：

**在国产算力平台上，让客户能做Agent RL训练，并且训练结果可信赖、可复现。**

ProRL Agent Rollout-as-a-Service是第一步——搭建公共底座；确定性RL训练是差异化——做GPU阵营没做到的事。

---

## 附：v2→v3修改对照表

| 维度 | v2 | v3 | 修改原因 |
|------|----|----|---------|
| 页数 | 15页 | 12页 | 压缩业界综述，扩大"我们做什么" |
| P1→P2逻辑 | 直接跳到"为什么RL" | 新增P2"Agent RL成为主流"过渡页 | L5指出逻辑缺环 |
| RL定位 | "唯一路径" | "不可或缺的关键环节" | L2指出过于绝对 |
| 数据点 | 多处不严谨 | 全部修正并标注修正说明 | L2/L4/L5多次指出 |
| 三分法 | 暗示学术共识 | 显式标注"我们的分析框架" | L2指出不诚实 |
| 技术地图 | 只列优势 | 增加成熟度/已知局限/NPU适配状态/挑战间trade-off | L2/L3/L8联合质疑 |
| P5-P13 | 9页业界综述 | 压缩为3页（P6-P8） | 主持人总评：前10页讲别人太多 |
| P14-P15 | 2页我们的技术 | 扩展为4页（P9-P12） | 主持人总评：最后5页该扩一倍 |
| 确定性RL训练 | P14的一行 | 独立页P10 | L8背书+L1支持 |
| 行动路线图 | 概念性映射图 | 三阶段gate+具体交付物+资源需求 | L1/L4/L3分阶段gate机制 |
| 竞争定位 | 缺失 | 新增P12 | L1认可的reframing |
| L8贡献 | 未体现 | 融入P9-P10（实战数据+方向背书+路径建议） | L8从质疑者变为盟友 |
| 已有vs待建 | 未区分 | 全面区分（实线/虚线） | L3"实线vs虚线"要求 |
