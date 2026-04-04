# Agentic RL 后训练洞察 — PPT结构设计 v2

> 基于三位专家（RL算法、系统架构、Agent系统）多轮批判性讨论 + 2026前沿搜索
> 范围：Agent System（电脑端Agent，如coding/browsing/工具使用）
> 团队定位：帮助客户在NPU上开箱即用地完成Agent后训练任务

---

## 页码总览与逻辑流

```
P1  Agent时代已来              ← 定调：产品/开源/资本三维爆发
P2  为什么是Agentic RL         ← 动因：harness→模型能力迁移 + RL不可替代性
P3  三大结构性变化 → 三大挑战   ← 核心推导页（重点）
P4  技术地图                   ← 导航：3挑战 × 业界解法 × 我们的技术
P5  挑战①归因 展开             ← 稀疏信号下的信用分配
P6  归因-技术1：CSO            ← Critical Step Optimization
P7  归因-技术2：SALT           ← Step-level Advantage via Trajectory Graph
P8  挑战②采集 展开             ← 探索成本与训练效率
P9  采集-技术1：Forge+CISPO    ← MiniMax异步RL框架
P10 采集-技术2：ProRL Agent    ← NVIDIA Rollout-as-a-Service
P11 挑战③评估 展开             ← 奖励信号的忠实性
P12 评估-技术1：Composer 2     ← Cursor的harness内训练+自判
P13 评估-技术2：Harness Patterns ← Anthropic 5种生产级模式
P14 横切约束：精度与一致性      ← 训推一致性/确定性计算/算子验证
P15 全栈协同·我们的技术布局    ← 收束：我们的技术如何映射到三大挑战
```

**逻辑链**：现象(P1) → 为什么RL(P2) → 变化→挑战(P3) → 导航(P4) → 逐个击破(P5-P13) → 横切约束(P14) → 我们的定位(P15)

---

## P1 | 定调层：Agent时代已来

**标题**：智能涌现：Agent从概念验证走向规模落地

**核心信息**：产品、开源、资本三维同步爆发。与原PPT基本一致，但更新数据点到2026.Q1。

**关键数据点（2026最新）**：
- Cursor Composer 2（2026.3）：基于Kimi K2.5做RL后训练，超越Opus 4.6
- OpenAI Codex CLI开源（Apache 2.0，7.3万星）
- Anthropic讨论年投$1B+采购RL环境
- 后训练算力占总算力40%+

---

## P2 | 动因层：为什么必须是Agentic RL（重点重构）

**标题**：从Harness到模型：Agent能力的迁移方向锁定了RL后训练

**与原PPT P2的区别**：原PPT用"SFT→DPO→RL三阶段排除法"论证。新版增加了**harness→模型能力迁移**作为第二条论证线，更有前瞻性。

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

**关键实证**：Cursor Composer 2通过RL训练self-summarization，模型学会从200K token压缩到~1K token，错误率降低50%。这就是harness能力内化到模型的活案例。

**下半页：这条迁移路径只有RL能走**

- SFT能教模仿，不能教决策——Agent状态空间太大，演示覆盖不了
- DPO能教对齐，不能教交互——Agent奖励来自环境验证，不来自人类偏好
- **只有Agentic RL**：环境反馈+闭环探索+从错误中学习 → 能系统性地将harness能力迁移到模型参数中

**飞轮**：模型更强→能做更多→产生更好轨迹→更好的RL信号→模型更强。三巨头各押一环：Anthropic投环境、OpenAI开放grader、Google全流程RL。

**P1→P2承接**：P1说"Agent时代已来"，P2回答"Agent能力提升的关键路径是什么？——是RL驱动的harness→模型能力迁移"。

---

## P3 | 核心推导层：三大结构性变化 → 三大挑战（重点重构）

**标题**：范式跃迁：Agent RL的三大结构性变化，直接产生三大未解挑战

**与原PPT P3的关键区别**：
1. 原PPT：六维变化→异步+训推分离→三大挑战（手段嵌入因果链，有逻辑跳跃）
2. 新版：**三个结构性变化直接对应三大挑战**（不经过"手段"中间环节，因果链更短更直接）
3. 异步+训推分离作为"业界应对手段"在P4技术地图中出现，不在P3

### 推导逻辑（一页之内完成）

**左半页：三个结构性变化（从LLM RL到Agent RL）**

| # | LLM RL | Agent RL | 变化本质 |
|---|--------|----------|---------|
| ① | 轨迹=几十到几百token | 轨迹=几千到几万token，跨多轮工具调用 | **轨迹长度量级跃升** |
| ② | 每条轨迹秒级生成 | 每条轨迹分钟级（环境执行、API调用） | **单条经验成本爆炸** |
| ③ | 正确性可验证（数学/代码有标准答案） | 正确性模糊多标准（开放任务无唯一正解） | **评估从确定变为模糊** |

上下文管理作为贯穿维度标注：Agent RL的三个变化都在"上下文持续累积"的背景下发生——轨迹越长上下文越大，成本越高上下文管理越关键，评估越模糊上下文中的线索越重要。

**右半页：三大变化直接对应三大挑战**

```
变化① 轨迹长度跃升 ──→ 挑战① 归因（Credit Assignment）
                        关键步骤被淹没在长轨迹中，CSO发现仅16%步骤是关键的

变化② 单条经验成本爆炸 ──→ 挑战② 采集（Experience Collection）  
                            每条rollout代价高，必须让每条都"值"
                            [子节]训练-部署harness一致性是采集效率的前提

变化③ 评估从确定到模糊 ──→ 挑战③ 评估（Reward Evaluation）
                            奖励信号本身可能不忠实、可被游戏
```

**底部过渡句**：三大挑战对应RL循环的三个环节——评估提供信号，归因分配信号，采集生成数据。业界用异步RL+训推分离+并行环境来应对，但手段本身引入副作用（staleness、精度偏移、状态管理）→P4展开。

**P2→P3承接**：P2说"必须走Agentic RL"，P3回答"Agentic RL和传统RL有什么结构性不同？这些不同引发了什么挑战？"

---

## P4 | 技术地图

**标题**：技术地图：三大挑战 × 业界前沿解法

**核心信息**：从P3的架构示意图出发，每个挑战拉出一条线对接2026前沿技术。标注"业界手段"（异步/解耦/并行环境）引入的副作用 → 横切约束（P14展开）。

| 挑战 | 2026前沿解法 | 对应页 |
|------|-------------|--------|
| ①归因 | CSO（关键步骤检测+验证）、SALT（轨迹图step-level归因） | P6-P7 |
| ②采集 | MiniMax Forge+CISPO（异步off-policy鲁棒）、ProRL Agent（Rollout-as-a-Service） | P9-P10 |
| ③评估 | Composer 2（harness内训练+模型自判）、Anthropic Harness Patterns | P12-P13 |
| 横切 | 训推一致性、确定性计算、算子验证 | P14 |

---

## P5 | 挑战①展开：稀疏信号下的信用分配

**标题**：挑战一：长轨迹中，谁是关键决策者？

**问题**：Agent一条轨迹几十到上百步，最终成功或失败。trajectory级奖励太粗——哪一步是真正的关键决策？CSO发现仅16%步骤是关键的，但识别这16%本身是难题。

**张力**：归因越精细→训练效果越好，但精细归因的计算和标注成本也在爆炸。

## P6 | 技术①-1：CSO（Critical Step Optimization）

- 来源：arxiv 2602.03412（2026.2）
- 用PRM识别关键步骤候选，用expert model提出替代方案，只在验证通过的关键步骤上做DPO
- 仅16%步骤上做监督，8B模型追平GPT-4.1
- **与我们技术的关联**：step-level信用分配需要高效的轨迹图计算算子 → 算子自动生成

## P7 | 技术①-2：SALT（Step-level Advantage via Trajectory Graph）

- 来源：Amazon，2025-2026
- 将Agent轨迹建模为有向图，推算每步的advantage
- **与我们技术的关联**：图推算的计算密度和精度要求高 → NPU算子优化 + 确定性计算

---

## P8 | 挑战②展开：探索成本与训练效率

**标题**：挑战二：每条轨迹都很贵——如何让每条都"值"？

**问题**：Agent每条rollout需要真实环境执行（秒到分钟级），比chat RL贵10-100倍。两层子问题：
1. **异步与新鲜度的权衡**：为维持吞吐必须异步，但异步引入策略滞后（staleness）
2. **训练-部署harness一致性**（显式子节）：训练环境与生产环境不同→学到的策略部署时失效。Composer 2验证了"在真实harness中训练"的路线，但涉及不确定性、安全隔离、状态回滚三大系统难题。

## P9 | 技术②-1：MiniMax Forge + CISPO

- 来源：MiniMax（2026.3）
- CISPO算法（Clipped IS Policy Optimization）：比DAPO快2x达到相同性能，对off-policy数据更鲁棒
- Forge框架：解耦训练框架和Agent scaffold，支持跨scaffold泛化
- **与我们技术的关联**：异步长程MoE RL范式 + staleness控制 + 训推一致性保证

## P10 | 技术②-2：NVIDIA ProRL Agent

- 来源：NVIDIA（arxiv 2603.18815，2026.3）
- Rollout-as-a-Service：将环境交互与策略更新解耦为独立服务
- Token-in/token-out一致性保证：解决re-tokenization drift
- **与我们技术的关联**：训推一致性/批不变性 + Ascend精度基线

---

## P11 | 挑战③展开：奖励信号的忠实性

**标题**：挑战三：你优化的目标本身可信吗？

**问题**：开放式Agent任务的正确性判定不像数学题有标准答案。三层风险叠加：
1. **奖励稀疏**：长轨迹中大部分步骤无信号
2. **Reward hacking**：并行探索越多越容易发现奖励漏洞
3. **Harness偏差**：harness定义的"成功"与真实用户需求之间的系统性偏差

**关键洞察**：Harness的奖励定义决定了Agent学到什么。"Harness is the New Dataset"——竞争优势不在数据量，在harness捕获的轨迹质量。

## P12 | 技术③-1：Composer 2（Cursor）

- 来源：Cursor（2026.3）
- 在完全相同的production harness中做RL训练——same tools, same prompts, same system message
- RL-trained self-summarization：模型学会上下文压缩，从harness能力变为模型能力
- 500+ Firecracker VM pods/sec的训练环境调度
- **与我们技术的关联**：Agent hardness工程（环境设计）+ RL框架易用性

## P13 | 技术③-2：Anthropic Harness Design Patterns

- 来源：Anthropic工程博客（2026）
- 5种生产级harness模式：Single Agent / Prompt Chaining / Router / Orchestrator-Worker / Swarm
- harness中的验证节点直接影响RL训练信号质量——harness设计=奖励工程
- **与我们技术的关联**：理解harness设计 → 理解Agent RL的训练负载特征 → 优化NPU支撑能力

---

## P14 | 横切约束：精度与一致性

**标题**：横切约束：所有优化手段的"宪法底线"——精度与一致性

**核心信息**：异步/量化/解耦等应对手段在提效的同时引入数值偏差。RL对偏差极度敏感（远超预训练），精度可复现性是科学调参的前提。

| 副作用来源 | 具体风险 | 我们的技术应对 |
|-----------|---------|--------------|
| 异步引入staleness | off-policy importance ratio偏移 | 异步RL精度保证算法 |
| 训推路径不一致 | 同一输入不同logits→ratio有方向性bias | **训推一致性/批不变性** |
| 低精度推理(mxfp8/mxfp4) | 策略分布尾部行为改变 | **4bit精度恢复算法** |
| 浮点非确定性 | 同checkpoint两次运行不同结果→调参变玄学 | **bf16/低精度确定性计算** |
| 跨硬件部署 | NPU vs GPU浮点实现差异累积 | **Ascend精度基线 + 算子自动扫描验证** |

---

## P15 | 收束层：全栈协同——我们的技术布局

**标题**：全栈协同：我们的技术如何精准对接三大挑战

**核心信息**：三大挑战 + 横切约束 → 需要全栈协同 → 我们已有的技术布局

### 技术映射图

```
挑战①归因 ←── 算子自动生成（step-level图计算算子）
              确定性计算（归因精度保证）

挑战②采集 ←── 异步长程MoE RL范式（Decoupled PPO/staleness控制）
              4bit低精RL（降低单条rollout的推理成本）
              Eagle3投机推理（加速rollout生成）
              Agent hardness工程（环境难度匹配提升采样价值）
              训推一致性/批不变性（采集质量保证）

挑战③评估 ←── RL框架易用性（标准化评估流水线）
              Ascend精度基线（跨平台评估对齐）

横切约束  ←── bf16/低精度确定性计算
              算子自动扫描验证
              Ascend精度基线
```

### 收束语

Agent时代的竞争，不在谁的模型更大，而在谁能让Agent**更快地从做事中学会做事**。我们的技术布局精准对接Agentic RL三大未解挑战和横切约束——让NPU成为客户做Agent后训练的最优选择。

---

## 附：与原PPT v3.1的对比

| 维度 | 原PPT v3.1 | 新PPT v2 |
|------|-----------|----------|
| P2论证 | 三阶段排除法(SFT→DPO→RL) | **+Harness→模型能力迁移线** |
| P3推导 | 六维变化→异步+训推分离→三大挑战 | **三个变化直接→三大挑战**（不经过手段中间环节） |
| 三大挑战 | 新鲜度/有状态/奖励失控 | **归因/采集/评估**（按RL循环切，正交性更强） |
| 技术选择 | PRIME-RL/SkyRL/DAPO/Dynamo等(2025) | **CSO/SALT/Forge+CISPO/ProRL/Composer 2等(2026)** |
| Harness视角 | 完全缺失 | **P2核心论证线 + P12-P13技术页** |
| 上下文管理 | 缺失 | **贯穿维度 + Composer 2实证** |
| 横切约束 | 无 | **P14专页：精度/一致性/确定性** |
| 我们的技术 | 未融入 | **P14-P15自然融入** |
| 前沿度 | 偏2025 | **2026.Q1前沿** |
