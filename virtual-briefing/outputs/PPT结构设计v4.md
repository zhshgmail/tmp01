# Agentic RL 后训练洞察 — PPT结构设计 v4

> 基于v3 + Round 2两轮领导再质疑（Phase 1: P1-P6 + Phase 2: P7-P12）
> 范围：**Coding/SE Agent**（诚实界定scope，不泛化到企业级Agent生态）
> 团队定位：帮助客户在NPU上开箱即用地完成Agent后训练任务
> **核心修改原则**：修正over-generalization、cross-cutting trade-off显式化、harness工程纳入路线图、AllReduce瓶颈量化、内外叙事分层

---

## 页码总览与逻辑流

```
P1   Agent时代已来              ← 定调：产品/开源/资本三维爆发（数据点已校验）
P2   Coding Agent RL是当前sweet spot ← 【v4修正】scope收窄到coding/SE领域，不再声称"全部"
P3   为什么必须是Agentic RL     ← 动因：harness→模型能力迁移 + RL不可替代性
P4   三大结构性变化 → 三大挑战  ← 核心推导页（标注"我们的分析框架"）
P5   技术地图                   ← 导航：3挑战 × 解法 × 成熟度/局限/NPU状态/cross-cutting trade-off
P6   挑战①归因：CSO + SALT      ← 压缩一页，标注前提条件和适用范围
P7   挑战②采集：ProRL Agent     ← 第一优先级，Gate 1量化标准融入
P8   挑战③评估：Harness即奖励   ← 增加reward hacking风险显式讨论
P9   横切约束：精度与一致性      ← 融入AllReduce瓶颈数据和性能阈值
P10  我们的差异化：确定性RL训练  ← 确定性训练+harness质量=双轮驱动
P11  行动路线图                 ← Gate 1量化check项+Gate 2增harness+存储I/O评估
P12  收束：竞争定位与客户价值   ← 区分内部版和外部版叙事
```

**逻辑链**：现象(P1) → Coding Agent RL是sweet spot(P2) → 为什么RL(P3) → 变化→挑战(P4) → 导航(P5) → 三挑战精炼(P6-P8) → 横切约束(P9) → 差异化卖点(P10) → 行动路线图(P11) → 竞争定位(P12)

**与v3的关键变化**：
- P2从"Agent RL正在成为主流"收窄为"Coding/SE Agent RL是当前sweet spot"，不再用"全部"
- P5技术地图增加cross-cutting trade-off列（从注释提升到表格）
- P7增加Gate 1量化标准（<30min + 4并发）
- P8增加reward hacking风险显式讨论
- P9增加AllReduce瓶颈数据和L8阈值
- P10增加"确定性训练+harness质量=双轮驱动"
- P11路线图全面升级：Gate 1量化、Gate 2增harness+NPU-hour、新增存储I/O评估
- P12区分内部版和外部版叙事
- 总页数保持12页

---

## P1 | 定调层：Agent时代已来

**标题**：智能涌现：Agent从概念验证走向规模落地

**核心信息**：产品、开源、资本三维同步爆发。全部数据点经过校验修正。

**关键数据点（2026.Q1，已修正，两轮质疑后无新争议）**：
- Cursor Composer 2（2026.3）：基于Kimi K2.5做RL后训练，在部分benchmark上超越Opus 4.6；GPT-5.4在Terminal-Bench达75.1分，竞争格局激烈
  - **修正说明**：v2单提Composer 2超Opus被L1指出cherry-picking。v3加GPT-5.4数据，v4无变化，Round 2确认修复
- OpenAI Codex CLI开源（Apache 2.0，约6.7万星）
- 据The Information报道，Anthropic领导层**讨论**在RL环境上的年度投入超$1B
- 广义训练算力（预训练+后训练）占总算力约40%，其中后训练占比持续攀升

**底部过渡句**：Agent产品在爆发，但哪个领域的Agent最先从RL中获益？——下一页回答。

---

## P2 | 【v4修正】过渡层：Coding/SE Agent RL是当前Sweet Spot

**标题**：RL驱动的Agent能力提升——Coding/SE领域率先验证

**核心信息**：v3声称"头部厂商全部在Agent训练中引入RL"，Round 2被L1用微软Agent Lightning/AutoGen反例击穿，L5指出五案例全是coding/SE领域不能泛化。v4诚实收窄scope。

> **v3→v4关键修正**：
> 1. "全部"→"在coding/SE Agent领域，主要头部厂商的主流方向已转向引入RL"
> 2. 明确标注scope限定和技术理由
> 3. Google案例标注"公开信息有限"
> 4. DeepSWE的"纯RL"表述修正

### Scope限定说明

**为什么是coding/SE Agent率先落地RL**（L8 Round 2 Phase 1解释，全场认可）：
- 代码能编译、测试能通过——天然的可验证reward信号
- 企业级Agent（CRM/ITSM/ERP）的奖励信号模糊（客户满意度？流程完成率？），RL落地更难
- P4挑战③"评估从确定到模糊"恰好解释了领域差异

**不在本PPT scope内**：
- 企业级Agent（微软Agent Lightning/AutoGen、Salesforce AgentForce等）——主要路线是SFT+工具编排，RL权重较低
- 多模态Agent——评估标准化程度不够，RL闭环尚早

### 案例论证矩阵（Coding/SE领域）

| 公司/项目 | Agent类型 | RL角色 | 关键结果 | 时间 |
|-----------|----------|--------|---------|------|
| Cursor Composer 2 | Coding Agent | 核心训练方法（基于K2.5做大规模RL） | 部分benchmark超Opus 4.6 | 2026.3 |
| OpenAI Codex | SE Agent | SFT+RL混合，RL做闭环优化 | Terminal-Bench 75.1% | 2026.3 |
| DeepSWE | 开源SE Agent | **改进版GRPO**（DAPO clip high、去KL/entropy loss），无SFT数据 | SWE-bench Verified 42.2% | 2026.2 |
| Google Agent | 多模态Agent | 全流程引入RL评估 | 内部产品集成（**公开信息有限**） | 2026.Q1 |
| NVIDIA ProRL Agent | 通用Agent RL框架 | Rollout-as-a-Service架构 | SWE-bench 9.6→18.0% | 2026.3 |

> **v4修正说明**：
> - DeepSWE行：v3写"纯RL（GRPO）"，L8指出是高度定制化的GRPO变体。v4标注"改进版GRPO"并列出具体改造。Qwen3-32B是一线base model，200步高效训练背后有精细工程
> - Google行：v3写"内部集成"，L5+L2指出数据不充分。v4显式标注"公开信息有限"。四个案例已足够支撑论点

**关键观察**：
1. 在coding/SE Agent领域，**主要头部厂商**在Agent训练中引入RL——区别仅在于RL在整个流水线中的权重
2. 趋势方向一致：SFT做基础能力注入，RL做闭环优化——**混合范式是主流**
3. ~~开源社区验证了不依赖顶级base model也行~~ → **修正**：DeepSWE用一线base model（Qwen3-32B）+ 精细工程化GRPO变体，验证了**RL方法论的有效性**，但高效训练有明确前提

**措辞校准**：RL不是"唯一路径"，而是Agent能力提升中**不可或缺的关键环节**。在coding/SE领域这一判断尤为成立。

**P1→P2承接**：P1说"Agent产品在爆发"，P2回答"在coding/SE领域——可验证reward信号最充分的领域——RL正在成为标配方法论"。

---

## P3 | 动因层：为什么必须是Agentic RL

**标题**：从Harness到模型：Agent能力的迁移方向锁定了RL后训练

**与v3完全一致，Round 2无新质疑**。

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

**背景补充**（回应L8 Phase 3质疑）：Composer 2的base model是Kimi K2.5（MoE），在B300上做continued pretraining再做大规模RL。RL路线的前提是"足够强的base + 算力集群"。

**下半页：这条迁移路径RL不可或缺**

- SFT能教模仿，不能教从错误中学习——Agent状态空间太大，演示覆盖不了
- DPO能教对齐，不能教与环境交互——Agent奖励来自环境验证，不来自人类偏好
- **RL是不可或缺的关键环节**：环境反馈+闭环探索+从错误中学习
- **但不是唯一方法**：业界主流是SFT做基础能力注入 + RL做闭环优化的混合范式

---

## P4 | 核心推导层：三大结构性变化 → 三大挑战

**标题**：范式跃迁：Agent RL的三大结构性变化，直接产生三大未解挑战

**与v3结构一致，Round 2无新质疑**。

> **框架标注**：以下三分法是**我们的工程分析框架**（按RL循环的评估→归因→采集三环节切分），不是学术界的标准分类。

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
变化② 单条经验成本爆炸 ──→ 挑战② 采集（Experience Collection）  
变化③ 评估从确定到模糊 ──→ 挑战③ 评估（Reward Evaluation）
```

**挑战间耦合说明**（v3已有，v4保留）：
> 三大挑战并非完全正交。例如CSO解决归因需要额外PRM和expert model推理，本身加剧采集成本；CISPO提升采集效率但引入entropy崩塌风险影响评估稳定性。技术地图（P5）在表格中显式标注了这些cross-cutting trade-off。

**实战优先级说明**：
> 上述排序是因果链顺序，不是落地优先级。实战中评估应排第一——"没有可靠的reward，一切都是空中楼阁"（L8）。

---

## P5 | 技术地图

**标题**：技术地图：三大挑战 × 业界前沿解法 × 成熟度/局限/NPU状态/跨挑战影响

**v3→v4核心修改**：
1. **cross-cutting trade-off从P4注释提升为表格独立列**（L2 Round 2要求：表格应自洽不依赖外部注释）
2. **"已有能力"框标题精确化**（L3 Round 2要求：区分基础设施层和前沿技术层）

| 挑战 | 前沿解法 | 成熟度 | 已知局限 | **跨挑战影响** | NPU适配 | 对应页 |
|------|---------|--------|---------|--------------|---------|--------|
| ①归因 | CSO（关键步骤检测+验证） | 论文级 | 需PRM（Agent任务PRM不存在）；额外计算开销高 | **加剧采集瓶颈**（额外PRM推理+expert model+验证rollout三步成本） | 未适配 | P6 |
| ①归因 | SALT（轨迹图step-level归因） | 论文级 | 依赖轨迹间状态共享；高随机环境退化 | 影响较小（轻量级plug-and-play） | 未适配 | P6 |
| ②采集 | ProRL Agent（Rollout-as-a-Service） | 框架级 | 纯CUDA；HTTP通信延迟；Singularity容器 | **公共底座**——所有上层RL算法受益 | **未适配（第一优先级）** | P7 |
| ②采集 | Forge+CISPO（异步off-policy RL） | 框架级 | entropy崩塌（DISPO证实）；长轨迹importance weight累积衰减 | **长轨迹weight衰减影响归因质量**；entropy崩塌影响评估稳定性 | 未适配 | P7 |
| ③评估 | Composer 2（高保真仿真harness中RL） | 产品级 | 依赖强base+B300；sandbox不等于production；嵌套归因难 | harness质量决定归因和采集的方向（**垃圾进垃圾出**） | 不适用 | P8 |
| ③评估 | Harness Patterns（5种Agent模式） | 工程实践级 | 无代码/无benchmark/无ablation | 设计参考，无直接技术trade-off | 不适用 | P8 |
| 横切 | 训推一致性/确定性计算/算子验证 | **原型中** | batch-invariant原型阶段；**AllReduce是关键瓶颈**（开销20-50%） | 精度不一致放大所有挑战的误差 | **原型中** | P9-P10 |

**基础设施层已有能力**（实线）：
> **v4标题修正**：v3写"已有能力"被L3指出与前沿技术层混淆，稀释了"零落地"事实。v4明确标注为基础设施层。

- MindSpeed RL：基础RL训练（GRPO）在384卡NPU集群上跑通，支持DeepSeek-R1-671B规模
  - **v4补充说明**（L6+L8 Round 2 Phase 1指出）：MindSpeed RL目前验证的是**reasoning任务**（数学推理）的RL训练，不是Agent RL（多轮工具调用+环境交互）。从reasoning RL到Agent RL需要增加rollout管理、sandbox交互、多轮状态管理等能力——这正是ProRL Agent补的空白
- vllm-ascend：推理引擎NPU适配，sleep mode已初步解决，hybrid engine刚打通
  - **v4补充说明**（L7 Round 2 Phase 1指出）：支持标准LLM serving，Agent RL rollout的推理特征（动态batch、prefix caching跨轮复用、工具调用context挂起恢复）尚未专门优化
- 5个RL关键路径算子CANN vs CUDA精度比对档案

**前沿技术层待建能力**（虚线）：以上六项前沿技术的NPU适配均为待建。前沿技术层面已有能力为零。

---

## P6 | 归因挑战：CSO + SALT（压缩为一页）

**标题**：挑战一：长轨迹中谁是关键决策者？——两条技术路线及其适用条件

**与v3内容一致，Round 2无新质疑。**

### CSO（Critical Step Optimization）

- 来源：arXiv 2602.03412（2026.2）
- 核心：用PRM识别关键步骤候选（仅16%步骤是关键的），用expert model提出替代方案，只在验证通过的关键步骤上做DPO
- 效果：8B模型在**GAIA-Text-103**上追平GPT-4.1（注：GPT-4.1在GAIA上为中游水平）
- **前提链条成本**：step-level标注数据 → 训PRM → CSO执行。Agent任务PRM基本不存在，冷启动成本可能超过直接GRPO全量训练
- **跨挑战影响**：CSO的额外推理（PRM+expert model+验证rollout）**直接加剧采集瓶颈**
- NPU适配：未适配

### SALT（Step-level Advantage via Trajectory Graph）

- 来源：Amazon（2025-2026）
- 核心：同一prompt的多条轨迹建模为有向图，通过共享状态节点推算每步advantage。plug-and-play，不需要额外PRM
- **优势**：计算开销几乎可忽略，可直接叠加在GRPO上
- **局限**：依赖轨迹间有足够状态共享。高随机环境退化
- NPU适配：未适配

**归因小结**：CSO精度高但前提重（需PRM，加剧采集），SALT轻量但场景受限。归因方向投入优先级低于采集和横切约束。

---

## P7 | 采集挑战：ProRL Agent为核心（第一优先级）

**标题**：挑战二：每条轨迹都很贵——Rollout-as-a-Service是公共底座

**v3→v4核心修改**：增加Gate 1量化标准。

### ProRL Agent — Rollout-as-a-Service（NVIDIA，2026.3，**第一优先级**）

- 来源：arXiv 2603.18815
- 核心创新：
  - 将rollout生命周期（环境初始化、工具执行、奖励评分）作为独立HTTP服务从训练循环解耦
  - **Token-in/token-out一致性保证**：rollout产生的token ID直接传给trainer，全流程不做re-tokenization
- 效果：Qwen3-8B在SWE-Bench Verified上从9.6%提升到18.0%
- **为什么是第一优先级**（Round 1确立，Round 2进一步巩固）：
  1. 训推解耦+token一致性是**所有Agent RL方法的公共基础设施**
  2. NVIDIA刚开源一个月，快速跟进可在NPU生态中建立先发优势
  3. Rollout-as-a-Service核心是推理引擎+HTTP服务，与vllm-ascend技术栈复用度最高
  4. **v4补充**：MindSpeed RL只覆盖训练侧，ProRL Agent正好补rollout和环境交互侧的空白（L6 Round 2确认）

### 【v4新增】Gate 1量化标准

> Round 2 Phase 2中L4+L7联合推动，将模糊的"跑通"升级为可度量的check项。

**Gate 1通过标准**（~2026年5月初，L4来check）：

| Check项 | 标准 | 依据 |
|---------|------|------|
| 单条rollout完成时间 | **<30分钟** | L7数据：H100上单条约5-8min，折算910C约8-13min，30min给3倍余量 |
| 并发稳定性 | **4条rollout同时稳定运行** | L7提出：确保架构可横向扩展，不只是单条能跑 |
| Sandbox适配 | Singularity→Docker适配方案设计文档 | L4要求可check的工程交付物 |
| 存储I/O观测 | rollout环境操作I/O延迟分布数据 | **L6 Round 2新增**：Agent rollout的小文件随机I/O可能成为被忽视的瓶颈 |

### 已知挑战

- 纯CUDA实现，NeMo Gym集成，昇腾零移植
- Singularity容器运行时 vs Docker环境：需sandbox层适配
- HTTP round-trip延迟：Agent rollout几十次工具调用，部署拓扑需co-located
- vllm-ascend对rollout动态batch和长上下文管理的支持程度待验证
  - **v4补充**：三项改造——动态batch、prefix caching跨轮复用、工具调用context挂起恢复。第一个月scope只含最基本跑通（同步推理），优化推到第二、三个月
- **【v4新增】存储I/O特征不匹配风险**（L6 Round 2提出）：
  - Agent rollout环境操作（git clone/编译/测试执行）是小文件随机I/O密集型
  - 训练集群通常优化大块数据吞吐，小文件I/O可能成为性能瓶颈
  - Gate 1阶段增加I/O延迟观测项

### 辅助参考：Forge + CISPO（MiniMax，2026.3）

- CISPO：比DAPO快2x达同等性能（Qwen2.5-32B，50%训练步数）
- **已知重大风险**（Round 1已识别，Round 2无新质疑）：后期entropy崩塌（DISPO证实）；100步以上长轨迹importance weight衰减
- **跨挑战影响**：长轨迹weight衰减影响归因质量，entropy崩塌影响评估稳定性
- NPU适配：未适配

---

## P8 | 评估挑战：Harness即奖励（压缩为一页）

**标题**：挑战三：你优化的目标本身可信吗？——Harness is the New Dataset

**v3→v4核心修改**：增加reward hacking风险显式讨论（L8 Round 2 Phase 2提出）。

### 核心洞察

开放式Agent任务的正确性判定不像数学题有标准答案。三层风险叠加：
1. **奖励稀疏**：长轨迹中大部分步骤无信号
2. **Reward hacking**：并行探索越多越容易发现奖励漏洞
3. **Harness偏差**：harness定义的"成功"与真实用户需求之间的系统性偏差

**关键判断**：Harness的奖励定义决定了Agent学到什么。竞争优势不在数据量，在harness捕获的轨迹质量。

### 【v4新增】SWE-bench Reward Hacking风险显式讨论

> L8 Round 2 Phase 2提出："SWE-bench的'测试通过'reward信号不能评估代码质量。Agent可能学会monkey-patching让测试通过但代码质量极差。"

**风险分析**：
- SWE-bench定义"成功"=测试全通过
- 但真实软件工程不只是通过测试——代码可读性、设计合理性、性能影响、编译警告等都重要
- 学术界已有论文指出部分SWE-bench任务测试覆盖率不够，Agent可走捷径（monkey-patching）
- **如果我们用SWE-bench做第一场景的reward信号，训出的Agent可能在benchamrk上好看但实际不可用**

**缓解策略**：
- Gate 1-2：用SWE-bench二值reward做概念验证——先求有（"Agent RL在NPU上能跑能收敛"）
- Gate 2-3：引入多维辅助评估信号
  - 代码质量分（lint/complexity metrics）
  - Diff最小化（鼓励精准修复而非大面积重写）
  - 编译警告数
  - 测试覆盖率提升（不只是通过原有测试，还要验证新增测试）
- **路线图标注**：reward hacking风险是已知的、有计划缓解的，不是被忽视的

### 业界实践参考

**Composer 2（Cursor）**：
- 在高保真仿真harness（Firecracker VM隔离sandbox）中做RL训练
- RL-trained self-summarization：compaction error降50%（还有50%未解决）
- **嵌套归因挑战**：reward信号要同时奖励agent response和中间的self-summary

**Anthropic Harness Patterns**：
- 5种生产级Agent模式
- 成熟度说明：工程博客总结，无代码/无benchmark/无ablation

### 【v4新增】关键洞察：确定性训练 + Harness质量 = 双轮驱动

> L2 Round 2 Phase 2的核心论点："如果harness本身不可信，精度再高也没用。确定性地训出一个走捷径的Agent，可复现地产出垃圾。"

这个洞察贯穿P8→P10→P11：
- **P8（本页）**：识别harness质量是评估挑战的核心
- **P10（差异化）**：确定性RL训练只有在harness质量有保障时才有意义
- **P11（路线图）**：Gate 2必须包含参考harness实现，不能只有基础设施

---

## P9 | 横切约束：精度与一致性

**标题**：横切约束：所有优化手段的"宪法底线"——精度与一致性

**v3→v4核心修改**：增加AllReduce瓶颈的具体数据和L8性能阈值。

### 为什么精度一致性在RL中比预训练更致命

**L8实战数据**（910B上跑GRPO，Round 1提供）：
- CANN的LayerNorm算子和CUDA的数值结果在第5-6位有效数字不同
- Reward曲线前200步与GPU几乎一致，**300步后开始分叉，500步差距达2-3个百分点**
- 根因定位花了一周：LayerNorm的reduction order不同

### 副作用来源与应对

| 副作用来源 | 具体风险 | 我们的应对 | 当前状态 |
|-----------|---------|-----------|---------|
| 异步引入staleness | off-policy importance ratio偏移 | 异步RL精度保证算法 | 方向性研究 |
| 训推路径不一致 | 同一输入不同logits→ratio有方向性bias | 训推一致性/批不变性 | **原型阶段** |
| 低精度推理 | 策略分布尾部行为改变 | 4bit精度恢复算法 | 方向性研究 |
| 浮点非确定性 | 同checkpoint两次运行不同结果 | bf16/低精度确定性计算 | **原型阶段** |
| 跨硬件部署 | NPU vs GPU浮点实现差异累积 | Ascend精度基线 | **5个算子比对完成** |
| MoE训推一致性 | expert routing不一致 | router replay机制研究 | 方向性研究 |

### 【v4新增】AllReduce：确定性RL方向的关键瓶颈

> L8+L7 Round 2 Phase 2联合追问，明确了这是差异化方向的生死线。

**已完成验证**：
- LayerNorm确定性实现：固定reduction order，性能开销约**15%**——可接受
- Softmax确定性实现：固定split策略，性能开销约**12%**——可接受

**待攻克瓶颈——AllReduce**：
- CANN的AllReduce实现和NCCL在reduction order上差异大，不是简单参数配置
- 两条方案路径：

| 方案 | 原理 | 预估开销 | 确定性程度 |
|------|------|---------|-----------|
| 固定reduction tree拓扑 | 锁定通信拓扑消除非确定性 | **20-30%** | 高 |
| 确定性ring-reduce | 替代tree-reduce为确定性ring | **40-50%** | 极高 |

**L8性能阈值**（Round 2 Phase 2设定）：
- **30%以内**：方向成立，继续推进
- **30-50%**：需要工程优化压缩开销
- **50%以上**：重新评估方案

**CUDA基线参考**（L7提供）：SGLang社区batch-invariant inference开启后延迟增加约34%（优化后），和Thinking Machines Lab原始实现的61.5%相比大幅降低。我们昇腾端到端30-40%水平和CUDA相当则不是劣势。

**AllReduce实测数据是Gate 1并行启动的第一个milestone。L8担任技术评审。**

### BF16 vs FP16的关键发现

最新研究证明BF16 mantissa位数不够是训推不一致主要根因，切FP16可消除大部分mismatch。但昇腾训练流水线多处hardcode BF16，改造是系统级工作。尚未启动。

### CANN Next的预期管理

950PR CANN Next提升编程兼容性，但BF16/FP16算子行为与当前CANN差异不大。**不能依赖下一代硬件自动解决精度问题。**

---

## P10 | 【更新】我们的差异化：确定性RL训练

**标题**：差异化方向：确定性训练 + Harness质量 = 双轮驱动

**v3→v4核心修改**：从单纯的"确定性RL训练"扩展为"确定性训练+harness质量"双轮驱动，融入L2 Round 2的关键洞察。

### 为什么确定性训练是差异化

**L8背书**（Round 1 Phase 6，Round 2继续背书）：
> "做RL训练的人极度痛恨不可复现性。如果昇腾能做到bitwise reproducibility，这在RL领域是一个非常有吸引力的卖点。"

**竞争态势**：
- GPU阵营：SGLang+vLLM的batch-invariant-ops在Qwen3-1.7B上演示了bitwise consistent RL——仅小模型，大规模仍是开放问题
- 昇腾机会：如果先于CUDA生态实现大规模确定性RL训练，这是GPU阵营做不到的差异化
- **关键瓶颈**：AllReduce（见P9），30%以内方向成立

### 【v4新增】L2的洞察：确定性训练 + Harness质量 = 双轮驱动

> L2 Round 2 Phase 2："如果harness本身不可信，精度再高也没用。确定性地训出一个走捷径的Agent，可复现地产出垃圾。"

**双轮驱动逻辑**：
1. **确定性训练**解决"训练结果可信赖"——同样代码同样数据跑两次结果一致，客户知道效果好坏归因于什么
2. **Harness质量**解决"训练方向正确"——高质量harness确保Agent学到的是真正有用的能力，不是exploit奖励漏洞
3. **两者缺一不可**：
   - 有确定性但没好harness → 确定性地训出垃圾
   - 有好harness但不确定性 → 好结果不可归因、不可复现
   - **两者兼具 → 可信赖、可复现、方向正确的Agent RL训练**

**对差异化叙事的影响**：
- v3的差异化叙事只有"确定性RL训练"一条线
- v4的差异化叙事是"确定性训练+harness质量"双线——后者通过Gate 2的参考harness实现来交付

### 技术路径

1. **batch-invariant算子昇腾适配**（与vLLM/SGLang社区合作）
   - 社区有CUDA实现的batch-invariant-ops库
   - 我们贡献昇腾backend，换取社区支持和生态存在感（L8建议）
   - 挑战：**AllReduce是关键瓶颈**（见P9详细分析）
   - 状态：LayerNorm/Softmax验证完成（15%/12%开销），AllReduce方案设计中

2. **RL关键路径算子精度基线**
   - 当前：5个算子完成比对
   - 下一步：扩展覆盖面 + 制定修复优先级

3. **BF16→FP16系统级评估**
   - 可消除大部分训推mismatch，但改造工程量大
   - 状态：未启动

### L8在两轮汇报中的三个关键贡献

1. **痛点验证**：910B上LayerNorm精度分叉导致reward曲线偏移的一手数据
2. **方向背书**：RL领域对可复现性的需求"真实且迫切"
3. **路径建议**：社区合作batch-invariant-ops适配 + AllReduce性能阈值设定

---

## P11 | 行动路线图（v4升级版）

**标题**：分阶段路线图：1月跑通 → 3月Demo → 6月发布

**v3→v4核心修改**：
1. Gate 1增加量化check项（30min+4并发+I/O观测）
2. Gate 2增加harness参考实现+NPU-hour消耗数据
3. 新增存储I/O评估为并行工作项
4. AllReduce实测数据作为确定性方向第一milestone

### Gate 1：1个月后（~2026年5月初）— ProRL Agent单机跑通

**交付物**（L4+L7 Round 2联合定义的**量化check项**）：

| # | 交付物 | 量化标准 | 来源 |
|---|--------|---------|------|
| 1 | ProRL Agent rollout在910C单机运行 | **单条SWE-bench rollout <30分钟** | L7：H100基线5-8min，910C折算8-13min，30min给3倍余量 |
| 2 | 并发能力验证 | **4条rollout同时稳定运行** | L7：确保架构可横向扩展 |
| 3 | Sandbox适配方案 | Singularity→Docker适配方案设计文档 | L4：可check的工程交付物 |
| 4 | **【v4新增】存储I/O观测** | rollout环境操作I/O延迟分布数据 | **L6 Round 2**：小文件随机I/O可能成被忽视的瓶颈 |

**通过条件**：L4来check。通过后支持扩到5-8人核心团队。

**并行启动**：
- 确定性RL训练方向：batch-invariant算子昇腾适配POC（3-5人独立团队）
  - **v4明确**：AllReduce实测数据是第一milestone，L8担任技术评审
  - 阈值：AllReduce确定性开销30%以内方向成立，50%以上重新评估
- 精度基线扩展：从5个算子扩展RL关键路径全覆盖
- **【v4新增】存储I/O评估**：如果Gate 1观测发现瓶颈在存储而非推理，适配方案需包含存储优化

### Gate 2：3个月后（~2026年7月）— 可展示的客户Demo

**交付物**（L3 Round 1要求 + L2/L1 Round 2升级）：

| # | 交付物 | 说明 | 来源 |
|---|--------|------|------|
| 1 | SWE-bench Agent RL训练全流程在910C集群跑通 | 端到端完整训练循环 | v3已有 |
| 2 | 训推一致性和token-level精度对齐展示 | 差异化可视化 | v3已有 |
| 3 | **【v4新增】SWE-bench参考harness实现** | 文档+代码，让客户能照着跑 | **L2+L1 Round 2**："造了高速公路但没人教客户开车" |
| 4 | **【v4新增】NPU-hour消耗数据** | 单次SWE-bench RL训练成本量级 | **L3 Round 2**：客户看Demo还要看经济可行性 |
| 5 | 必须是可以带客户看的东西 | 可视化的RL训练过程 | L3 Round 1 |

**Demo定义**：
- 可视化的RL训练过程（reward曲线、解题成功率随训练步数提升）
- SWE-bench解题成功率的昇腾 vs GPU对比
- 训推一致性的A/B对比展示（同一checkpoint两次训练结果一致性）
- **【v4新增】参考harness实现文档和代码**——客户看了Demo后知道自己怎么用
- **【v4新增】NPU-hour消耗数据**——客户对训练成本有量级感知

### Gate 3：6个月后（~2026年10月）— 可发布Benchmark

**交付物**：
1. 可复现的SWE-bench Agent RL benchmark结果，能对外发布
2. batch-invariant推理集成进vllm-ascend推理流水线
3. RL关键路径算子精度基线完整覆盖
4. **【v4新增】多维评估信号集成**：SWE-bench基础reward + 代码质量分 + diff最小化等辅助reward，缓解monkey-patching风险
5. （可选）与vLLM/SGLang社区联合发布昇腾确定性RL训练结果

### 资源需求

| 方向 | 人力 | 时间 | 技能栈 |
|------|------|------|--------|
| ProRL Agent Rollout-as-a-Service昇腾适配 | 5-8人 | 3个月PoC，6个月可用 | 推理引擎、容器编排、HTTP服务 |
| 确定性RL训练（batch-invariant + 精度基线） | 3-5人 | 3-4个月算子集成，6个月全路径覆盖 | 算子开发、CANN内核、精度调优 |
| **【v4说明】harness参考实现** | **不需额外大量人力** | Gate 2阶段 | ProRL Agent已含SWE-bench harness集成，做昇腾适配+整理为参考 |
| 总计 | **8-13人** | 并行推进 | |

### 技术映射总图

```
━━━ 基础设施层已有能力（实线）━━━
训练侧  ←── MindSpeed RL基础RL训练（GRPO，384卡，671B规模）
              ⚠ 目前是reasoning RL，不是Agent RL
推理侧  ←── vllm-ascend推理引擎
              ⚠ 标准LLM serving，Agent rollout推理特征未优化
精度侧  ←── 5个RL关键路径算子CANN/CUDA精度比对档案
              L8的910B RL实战经验和精度数据

┈┈┈ 前沿技术层待建能力（虚线，按优先级排序）┈┈┈
【优先级1】ProRL Agent Rollout-as-a-Service昇腾适配
          → 公共底座，补MindSpeed RL缺的rollout和环境交互
【优先级2】确定性RL训练（batch-invariant算子 + 精度基线）
          → 差异化卖点，关键瓶颈AllReduce（阈值30%）
【优先级2b】SWE-bench参考harness实现
          → 确保客户用得起来，与确定性训练形成双轮驱动
【优先级3】CISPO/Forge异步RL框架评估
          → 待CISPO稳定性问题有方案后再投入
【优先级4】CSO/SALT归因技术NPU验证
          → 待Agent PRM生态成熟后再投入
```

---

## P12 | 收束：竞争定位与客户价值

**标题**：竞争定位重构：不是对标NVIDIA，而是让国内客户能做起来

**v3→v4核心修改**：区分内部版和外部版叙事（L1+L8+L3 Round 2 Phase 2联合推动）。

### 竞争定位

**传统定位**（已过时）：910C vs H100单卡性能对标
- 910C推理性能约为H100的60%
- 纯FLOPS比拼没有优势

**新定位**：让国内客户的Agent RL从"不能做"变为"能做"
- **供应链现实**：国内大量客户受约束，H100不是想买就能买
- **真正的对标不是NVIDIA，而是"不做"**
- **差异化不在单卡FLOPS，在双轮驱动**：
  - 确定性RL训练（GPU阵营大规模场景未解决）
  - 高质量harness参考实现（客户用得起来的第一步）

### 目标客户画像

- **第一目标**：国内做Agent产品的公司——有模型、有Agent场景、受H100供应约束
- **第一场景**：Software Engineering Agent的RL训练
  - **v4强调**：Coding/SE Agent是RL的sweet spot（L8 Round 2技术解释，全场认可）——可验证reward信号是结构性优势
  - 选择理由：评估相对标准化（SWE-bench）、开源benchmark丰富、学术界关注度高

### 【v4新增】内部版 vs 外部版叙事

> L1+L8+L3 Round 2 Phase 2联合讨论结论。

| 维度 | 内部版（当前使用） | 外部版（Gate 2后使用） |
|------|-------------------|---------------------|
| **核心叙事** | "Agent时代的竞争，不在谁的模型更大，而在谁能让Agent更快地从做事中学会做事" | "我们正在投入让国产算力平台支持Agent RL训练" |
| **定位** | 定义竞争维度，内部愿景对齐 | 说我们在做什么，不过度承诺 |
| **前提条件** | 无——内部对齐不需要外部证据 | **Gate 2 Demo背书**——有可展示的成果 |
| **风险** | 无 | 没有Demo就对外讲会被反驳"你们连Agent RL都没跑通就来定义竞争维度"（L1原话） |
| **升级路径** | — | Gate 2通过后，从"我们正在投入"升级为"我们能做到"；Gate 3通过后，可使用内部版叙事对外 |

**软化过渡话术**（L3 Round 2建议）：
> "我们正在投入让国产算力平台支持Agent RL训练。我们的目标是：在昇腾上，让客户能做Agent RL训练，并且训练结果可信赖、可复现。"

### 收束语（内部版）

Agent时代的竞争，不在谁的模型更大，而在谁能让Agent**更快地从做事中学会做事**。

我们的价值主张不是"和NVIDIA比谁的卡更快"，而是：

**在国产算力平台上，让客户能做Agent RL训练，并且训练结果可信赖、可复现。**

- ProRL Agent Rollout-as-a-Service是第一步——搭建公共底座
- 确定性RL训练是差异化——做GPU阵营没做到的事
- **harness参考实现是客户体验——确保客户用得起来**

---

## 附：v3→v4修改对照表

| 维度 | v3 | v4 | 修改原因 |
|------|----|----|---------|
| P2 scope | "头部厂商全部引入RL" | "Coding/SE Agent领域主流方向已转向RL" | L1微软反例+L5企业级Agent反例击穿"全部" |
| P2 Google案例 | 写"内部集成"无注释 | 显式标注"公开信息有限" | L5+L2指出数据不充分 |
| P2 DeepSWE | "纯RL（GRPO）" | "改进版GRPO（DAPO clip high、去KL/entropy loss）" | L8指出简化过度 |
| P2 观察3 | 暗示"不依赖顶级base model" | 修正为"验证RL方法论有效性，但有前提" | L8指出Qwen3-32B是一线模型 |
| P5框标题 | "已有能力"/"待建能力" | "基础设施层已有能力"/"前沿技术层待建能力" | L3指出稀释"零落地"事实 |
| P5表格 | cross-cutting trade-off藏在P4注释 | **独立列"跨挑战影响"** | L2："报喜不报忧的更隐蔽形式" |
| P5已有能力说明 | 未区分reasoning RL和Agent RL | 明确MindSpeed RL是reasoning RL不是Agent RL | L6+L8 Round 2指出 |
| P7 Gate 1 | "单条rollout跑通" | **<30min+4并发+sandbox文档+I/O观测** | L4+L7量化+L6存储I/O盲区 |
| P8 | 无reward hacking讨论 | **SWE-bench monkey-patching风险+缓解策略** | L8 Round 2提出 |
| P9 | 未提AllReduce具体数据 | **AllReduce两方案（20-30%/40-50%）+L8阈值30%** | L8+L7 Round 2联合追问 |
| P10 | 仅"确定性RL训练" | **"确定性训练+harness质量=双轮驱动"** | L2："确定性地训出垃圾"论点 |
| P11 Gate 2 | 技术Demo | **+参考harness实现+NPU-hour消耗数据** | L2+L1 harness盲区+L3商务维度 |
| P11并行 | 无存储I/O评估 | **新增存储I/O评估工作项** | L6 Round 2提出 |
| P11优先级 | 无harness排序 | **增加优先级2b：harness参考实现** | L2+L1 Round 2 |
| P12叙事 | 单一版本 | **内部版vs外部版分层+软化过渡话术** | L1+L8+L3叙事时机管控 |

---

## 附：两轮质疑后的34项停车场议题状态

> 34项停车场议题中：
> - 已在PPT v4中直接解决：0项（停车场议题均为超出PPT范围的深入专题）
> - 在PPT v4中部分回应：8项（#28 Gate 1量化、#29存储I/O、#30 reward hacking、#31 AllReduce、#32 harness工程、#33 Gate 2商务、#34对外叙事、#24 scope界定——在PPT层面做了修改，但深入分析仍需专题讨论）
> - 待后续专题讨论：26项