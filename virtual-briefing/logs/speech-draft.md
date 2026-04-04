# 演讲稿：Agentic RL 后训练洞察

> 演讲者：RL后训练算法专家
> 听众：华为计算部门5位高级领导
> 日期：2026年4月
> 数据验证状态：全部10个关键数据点已搜索验证（见附录）

---

## 数据验证总结（演讲前必读）

| # | 数据点 | 验证结果 | 风险等级 |
|---|--------|---------|---------|
| 1 | DeepSWE Pass@1 42.2% | 准确。但best-of-16应为71%（Pass@16），59%是hybrid TTS结果，不是best-of-16。**PPT需修正** | 中 |
| 2 | Agent RL 90%+运行时在rollout | 方向正确。业界共识为80-90%，OpenRLHF等多个框架论文引用"80%时间在sample generation"。"90%+"更适用于Agent RL（因环境交互额外开销） | 低 |
| 3 | CSO 16%关键步骤, 37%提升, 8B追平GPT-4.1 | 准确。arXiv 2602.03412原文：37%和26%相对提升（分别在GAIA-Text-103和XBench-DeepSearch），仅16%步骤需要监督，8B模型match GPT-4.1 | 低 |
| 4 | MiniMax Forge CISPO比DAPO快2x | 准确。MiniMax公开报告：CISPO在Qwen2.5-32B上达到DAPO 2x加速 | 低 |
| 5 | Composer 2 self-summarization 200K→1K | 准确。Cursor官方技术报告：trained self-summaries平均约1K token，对比prompted方式5K+ token，compaction error降低50% | 低 |
| 6 | Anthropic年投$1B+采购RL环境 | 方向正确但需谨慎。来源为The Information报道，Anthropic领导层"讨论"花费超10亿美元在RL环境上。非官方确认数字，是"讨论中的规模" | 高 |
| 7 | 后训练算力占总算力40%+ | 方向正确。Epoch AI等多个分析报告确认后训练占比持续攀升至约40%，但不同口径定义"后训练"范围不一 | 中 |
| 8 | 昇腾910C约H100 60%推理性能 | 准确。多个来源确认（Tom's Hardware, TrendForce），原始数据来源为DeepSeek研究报告 | 低 |
| 9 | NVIDIA Dynamo 30x吞吐提升 | 准确但有条件。30x是在GB200 NVL72上跑DeepSeek-R1的特定配置下disaggregated serving的数据，不是通用场景。Llama 70B在Hopper上是2x+ | 中 |
| 10 | ProRL Agent token-in/token-out一致性 | 准确。arXiv 2603.18815原文：用token ID作为canonical representation，消除re-tokenization drift | 低 |

---

## P1 | 定调层：Agent时代已来

### 核心话术（30秒内亮观点）

"各位领导，今天不讲趋势综述，直接说结论：Agent已经从概念验证进入规模落地，产品、开源、资本三条线在2026年Q1同时爆发。这个窗口期对我们的NPU平台意味着——Agent后训练负载将成为下一个主力算力需求。"

### 支撑数据

- **产品端**：Cursor Composer 2（2026.3发布）基于Kimi K2.5做RL后训练，在coding benchmark上超越Claude Opus 4.6——一家IDE公司用RL训练自研模型打败头部大模型厂商，说明Agent RL已经是产品竞争力的核心。（来源：Cursor官方博客、技术报告）
- **开源端**：OpenAI Codex CLI开源（Apache 2.0），7.3万GitHub星，DeepSWE用RL训练开源32B模型达到Pass@1 42.2%——开源模型做Agent的门槛已经降到可触达。（来源：Together.ai DeepSWE技术报告）
- **资本端**：Anthropic领导层讨论年投超10亿美元采购RL训练环境。后训练算力占总算力份额持续攀升，多个分析机构估计达到约40%。（来源：The Information报道；Epoch AI分析）

### 预判质疑

**Q："Agent火了两年了，之前也有人说过爆发，数据可信吗？"**
A："之前的Agent是harness驱动的——靠外部工具链堆能力，模型本身没变。2026的区别是模型自身通过RL学会了Agent能力——Cursor自研模型反超Opus就是最直接的证据。这不是hype cycle，是产品已经在用RL赚钱。"

**Q："10亿美元数字可靠吗？"**
A："这来自The Information的消息源，属于'领导层讨论的规模'而非官方确认的预算数字。但即便打个折，量级足以说明头部公司对RL环境的战略投入级别。如需更保守的表述，我们可以说'业界头部公司在RL环境上的年度投入已达到数亿到十亿美元量级'。"

### 对我们的含义

"这三条线同时爆发意味着：Agent后训练不再是实验室项目，而是真实的算力负载——我们的NPU平台必须能承接这类负载，否则客户会转向GPU。"

---

## P2 | 动因层：为什么必须是Agentic RL

### 核心话术（30秒内亮观点）

"Agent能力提升有且只有一条系统性路径：把harness（外部工具链）的能力通过RL迁移到模型参数中。SFT教不了决策，DPO教不了交互，只有RL能闭环驱动这个迁移。Cursor已经证明了这条路线。"

### 支撑数据

- **迁移实证**：Cursor Composer 2通过RL训练self-summarization——模型学会将200K token的上下文压缩到约1K token，对比传统prompted方式（5K+ token），compaction error降低50%。这是一个harness能力（上下文管理）被RL内化到模型参数的实例。（来源：Cursor官方博客 cursor.com/blog/self-summarization，Composer 2技术报告）
- **三条路径排除法**：
  - SFT：状态空间太大，演示覆盖不了所有Agent场景
  - DPO：Agent奖励来自环境验证而非人类偏好，DPO的偏好对范式不适用
  - **RL**：环境反馈 + 闭环探索 + 从错误中学习 = 唯一能系统性迁移能力的方法
- **三巨头各押一环**：Anthropic投环境（年度投入十亿级）、OpenAI开放grader、Google全流程RL

### 预判质疑

**Q："SFT+DPO够用了，为什么一定要RL？成本不是高很多吗？"**
A："对单轮对话对齐，SFT+DPO确实够。但Agent是多轮、有状态、与环境交互的。Cursor用RL训self-summarization，错误率降50%——这个结果SFT做不到，因为SFT无法从'压缩失败导致任务失败'的反馈中学习。RL的成本确实高，但这正是我们要讨论的第三页——如何降低成本。"

**Q："harness→模型迁移的终局是什么？模型最终能完全替代harness？"**
A："不会完全替代。harness负责安全边界和API代理这些不适合模型内化的功能。但决策类能力（规划、纠错、上下文管理）正在加速迁移。终局是'瘦harness+胖模型'——harness只做安全网，模型做所有决策。"

### 对我们的含义

"Agentic RL是Agent能力提升的必经之路。我们的NPU平台要成为客户的首选，就必须在Agentic RL这条路径上提供完整支撑——不是做大模型预训练的备选，而是做Agent后训练的首选。"

---

## P3 | 核心推导层：三大结构性变化 → 三大挑战

### 核心话术（30秒内亮观点）

"从传统LLM RL到Agent RL，有三个结构性变化：轨迹长了百倍、单条经验贵了百倍、评估从确定变为模糊。这三个变化直接产生三个未解挑战：归因、采集、评估。这不是学术分类，而是RL循环的三个环节各自遇到的工程瓶颈。"

### 支撑数据

- **变化①→挑战①（轨迹长→归因难）**：Agent轨迹从几百token增长到几千到几万token，跨多轮工具调用。CSO研究发现，一条Agent轨迹中仅16%的步骤是真正影响结果的关键决策步骤——但识别这16%本身就是难题。（来源：arXiv 2602.03412）
- **变化②→挑战②（成本爆炸→采集难）**：Agent RL的80-90%训练时间消耗在rollout推理上（业界共识，OpenRLHF等多个框架论文引用）。Agent的rollout还需要真实环境执行（秒到分钟级），比chat RL贵10-100倍。这就是为什么NVIDIA做ProRL Agent、MiniMax做Forge——都在解决采集效率问题。
- **变化③→挑战③（评估模糊→奖励不可信）**：开放式Agent任务不像数学题有标准答案。奖励稀疏、reward hacking、harness偏差三层风险叠加。

### 预判质疑

**Q："三个挑战听着像学术界的问题，我们客户真的在乎吗？"**
A："客户不会说'我有归因问题'，但客户会说'我用8张卡训了3天Agent效果没提升'。归因问题导致训练信号被噪声淹没，采集问题导致GPU利用率不到20%，评估问题导致模型学歪了。这三个挑战就是客户训练效率低的根因。"

**Q："为什么不用原来的六维分析？三个变化会不会太简化了？"**
A："六维分析是全面的但冗余——比如'多轮交互'和'工具使用'本质都是轨迹变长的表现。我们按RL循环的三个环节切（评估提供信号、归因分配信号、采集生成数据），每个挑战正交、互不重叠，方便后续逐个对接技术方案。"

### 对我们的含义

"这三大挑战中的每一个都有算力含义——归因需要精确计算、采集需要高效推理、评估需要可靠环境。我们的技术布局必须覆盖这三个环节，不能只做其中一个。"

---

## P4 | 技术地图

### 核心话术（30秒内亮观点）

"三大挑战不是我们发明的问题——2026年Q1业界最前沿的团队都在攻坚。这张技术地图对接六项2026前沿技术，每一项都已经发表了论文或产品。我们的技术布局精准对接每个挑战和横切约束。"

### 支撑数据

| 挑战 | 前沿技术 | 出处与验证 | 我们的对接点 |
|------|---------|-----------|-------------|
| ①归因 | CSO：16%步骤监督，8B追平GPT-4.1 | arXiv 2602.03412，2026.2，已验证 | step-level图计算算子、确定性计算 |
| ①归因 | SALT：轨迹图step-level归因 | Amazon，2025-2026 | NPU算子优化 |
| ②采集 | MiniMax Forge+CISPO：比DAPO快2x | MiniMax公开报告，2026.3，已验证 | 异步长程MoE RL + staleness控制 |
| ②采集 | ProRL Agent：Rollout-as-a-Service | arXiv 2603.18815，2026.3，已验证 | 训推一致性/批不变性 |
| ③评估 | Composer 2：RL-trained self-summarization | Cursor技术报告，2026.3，已验证 | Agent harness工程 + RL框架易用性 |
| ③评估 | Anthropic Harness Patterns | Anthropic工程博客，2026 | harness设计理解→负载特征优化 |
| 横切 | 训推一致性 + 精度保证 | 多项，见P14 | 确定性计算 + Ascend精度基线 |

### 预判质疑

**Q："这些技术都是别人的，我们自己有什么？"**
A："我们的价值不是重新发明CSO或CISPO——而是让客户在NPU上能跑通这些方案。P14-P15会展开，我们在训推一致性、确定性计算、4bit精度恢复、算子自动生成等方面有差异化技术。业界解法确定了'做什么'，我们解决'在NPU上怎么做'。"

**Q："NVIDIA Dynamo 30x吞吐提升，910C只有H100的60%，差距怎么追？"**
A："两点：第一，Dynamo的30x是在GB200 NVL72特定配置下跑DeepSeek-R1的disaggregated serving结果，不是通用场景——Llama 70B在Hopper上是2x+提升，数量级完全不同。第二，910C的60%是推理性能裸数据，加上软件栈优化（DeepSeek自己的实测也发现手动优化后效率可进一步提升），实际差距可以缩小。我们的策略不是硬件对标，而是在Agent RL这个特定负载上做全栈优化——推理效率只是一个环节。"

### 对我们的含义

"技术地图说明一件事：Agent RL不是单点问题，是系统问题。谁能提供全栈方案——从算法到框架到算子到硬件——谁就有竞争力。这恰好是我们作为计算平台的优势。"

---

## P5-P7 | 挑战①归因：稀疏信号下的信用分配（简要版）

### P5 核心话术
"一条Agent轨迹几十到上百步，只有末尾给一个成功或失败的信号。问题是：到底哪一步做对了？CSO发现只有16%的步骤是关键的——但如果不识别这16%，训练信号就被83%的噪声步骤稀释了。"

### P6 CSO核心话术
"CSO的做法很巧妙：用PRM识别候选关键步骤，用expert模型提出替代方案，只在验证通过的关键步骤上做DPO。结果是：仅16%步骤的监督让8B模型追平GPT-4.1，在GAIA-Text-103上37%相对提升。这证明归因精度是Agent训练效率的杠杆。"

### P7 SALT核心话术
"SALT把Agent轨迹建模为有向图来推算每步的advantage。和CSO思路互补——CSO用验证，SALT用结构。两者都说明：step-level归因是Agent RL的刚需。"

### 对我们的含义
"归因计算本质上是图计算——需要高效的轨迹图算子和确定性精度保证。这正是我们在NPU算子层面的切入点。"

---

## P8-P10 | 挑战②采集：探索成本与训练效率（简要版）

### P8 核心话术
"Agent RL的rollout推理占训练时间的80-90%。每条轨迹还需要真实环境执行，秒到分钟级。两个核心子问题：一是异步采集带来的策略滞后（staleness），二是训练环境与部署环境不一致导致学到的策略失效。"

### P9 Forge+CISPO核心话术
"MiniMax的CISPO算法通过clipping importance sampling weights而非token updates，让所有token都参与梯度计算——包括低概率但对探索至关重要的token。结果比DAPO快2倍达到相同性能。Forge框架解耦训练框架和Agent scaffold，支持跨scaffold泛化。"

### P10 ProRL Agent核心话术
"NVIDIA的ProRL Agent核心创新是token-in/token-out：用token ID作为canonical representation，全流程不做re-tokenization。解决了一个容易被忽略但影响很大的问题——rollout和training之间的re-tokenization drift会导致隐性off-policy偏差。"

### 对我们的含义
"采集效率直接决定GPU/NPU利用率。我们的异步长程MoE RL范式、4bit低精RL、Eagle3投机推理都是在降低单条rollout的成本——让客户同样的卡数训出更好的Agent。"

---

## P11-P13 | 挑战③评估：奖励信号的忠实性（简要版）

### P11 核心话术
"你优化的目标本身可信吗？Agent任务的正确性判定不像数学题有标准答案。奖励稀疏、reward hacking、harness偏差三层风险叠加。关键洞察：Harness的奖励定义决定了Agent学到什么——'Harness is the New Dataset'，竞争优势不在数据量，在harness捕获的轨迹质量。"

### P12 Composer 2核心话术
"Cursor的做法是在完全相同的production harness中做RL训练——same tools, same prompts, same system message。训练环境用Firecracker VM隔离，每秒能调度500+个pods。self-summarization的成功证明：在真实harness中训练才能得到忠实的奖励信号。"

### P13 Anthropic Harness Patterns核心话术
"Anthropic总结了5种生产级harness模式。关键启示是：harness中的验证节点直接影响RL训练信号质量。Harness设计就是奖励工程——理解harness设计等于理解Agent RL的训练负载特征。"

### 对我们的含义
"评估质量决定训练方向。我们在Agent harness工程和RL框架易用性上的投入，本质上是帮客户设计更好的奖励函数——这比优化训练速度更影响最终效果。"

---

## P14 | 横切约束：精度与一致性（简要版）

### 核心话术
"所有应对手段——异步、量化、解耦——在提效的同时都引入数值偏差。RL对偏差极度敏感，远超预训练。一个具体例子：ProRL Agent发现如果rollout和training之间re-tokenize，token序列可能不同，导致importance ratio有方向性bias，KL散度失控。精度可复现性是科学调参的前提。"

### 对我们的含义
"910C与H100的裸性能差距是60%，但如果我们在精度和一致性上做到业界最好——确定性计算、批不变性、4bit精度恢复——客户会因为训练结果可复现而选择我们，而不仅仅因为FLOPS便宜。"

---

## P15 | 收束层：全栈协同（简要版）

### 核心话术
"三大挑战 + 横切约束 = 需要全栈协同。我们已有的技术布局精准对接：算子自动生成对接归因计算，异步RL+4bit低精+Eagle3推理对接采集效率，harness工程+RL框架易用性对接评估质量，确定性计算+精度基线对接所有横切约束。"

### 收束语
"Agent时代的竞争，不在谁的模型更大，而在谁能让Agent更快地从做事中学会做事。我们的技术布局让NPU成为客户做Agent后训练的最优选择——不是GPU的替代品，而是全栈优化的差异化平台。"

### 对我们的含义
"今天汇报的核心诉求：Agent RL后训练是确定性的算力需求方向，我们的技术布局已经对准了业界三大挑战，需要资源支持加速落地。"

---

## 附录A：数据点详细出处

| # | 数据点 | 出处 | URL |
|---|--------|------|-----|
| 1 | DeepSWE Pass@1 42.2%, Pass@16 71% | Together.ai DeepSWE技术报告 | together.ai/blog/deepswe |
| 2 | RL训练80-90%时间在rollout | OpenRLHF论文、Amplify Partners分析 | arxiv.org/html/2405.11143v6 |
| 3 | CSO 16%关键步骤, 37%提升, 8B≈GPT-4.1 | arXiv 2602.03412 | arxiv.org/abs/2602.03412 |
| 4 | CISPO比DAPO快2x | MiniMax M2.5技术报告 | minimax.io/news/minimax-m25 |
| 5 | Composer 2 self-summarization 200K→~1K token | Cursor官方博客 | cursor.com/blog/self-summarization |
| 6 | Anthropic年度RL环境投入10亿+级别 | The Information报道 | 转引自TechCrunch、LinkedIn |
| 7 | 后训练算力占比约40% | Epoch AI、Deloitte分析 | epoch.ai/trends |
| 8 | 910C约H100 60%推理性能 | Tom's Hardware（原始数据来自DeepSeek） | tomshardware.com |
| 9 | Dynamo 30x吞吐（GB200+DeepSeek-R1特定配置） | NVIDIA官方技术博客 | developer.nvidia.com/blog |
| 10 | ProRL Agent token-in/token-out | arXiv 2603.18815 | arxiv.org/html/2603.18815v1 |

## 附录B：PPT数据修正建议

1. **DeepSWE best-of-16数据**：PPT中写"best-of-16 59%"，实际Pass@16为71%，59%是hybrid test-time scaling结果。建议修正为"Pass@1 42.2%, Pass@16 71%"或明确标注59%为hybrid TTS。
2. **Anthropic $1B+措辞**：建议从"年投$1B+"改为"据The Information报道，Anthropic领导层讨论在RL环境上年投入超10亿美元"，增加来源限定。
3. **Dynamo 30x**：PPT中如引用此数据，需标注是"GB200 NVL72 + DeepSeek-R1 disaggregated serving"的特定配置结果，避免被质疑泛化。
4. **后训练算力40%+**：多个分析来源支持，但"后训练"的定义口径不一（有的包含RLHF+蒸馏+合成数据，有的仅指RLHF）。建议演讲时说"广义后训练"。

## 附录C：高频质疑预案汇总

| 预期质疑 | 出现页 | 推荐回应策略 |
|---------|-------|-------------|
| "Agent之前也说过爆发" | P1 | 区分harness驱动（旧）vs RL驱动（新），Cursor产品验证 |
| "SFT+DPO够了" | P2 | Cursor self-summarization实证，SFT无法从环境反馈学习 |
| "三个挑战太学术" | P3 | 翻译为客户痛点：训练效率低、GPU利用率低、模型学歪 |
| "都是别人的技术" | P4 | 我们解决"在NPU上怎么做"，全栈优化是差异化 |
| "910C性能差距怎么追" | P4/P14 | 不靠FLOPS追，靠精度/一致性/全栈优化差异化 |
| "ROI和时间线？" | P15 | 诚实说正在补充，给出初步框架而非模糊承诺 |
| "团队有什么结果？" | P15 | 给方向性进展，不夸大，不达标的点说"需要补充" |
