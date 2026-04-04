# 演讲稿 v2：Agentic RL 后训练洞察（配合PPT v4）

> 基于PPT v4 + Round 1/Round 2两轮共16个Phase的质疑经验
> 每页目标用时：60-90秒（全程12页，总时长12-18分钟）
> 标注：[可对外] = 可在行业会议/客户沟通中使用；[仅内部] = 仅限内部对齐
> 标注：[预判质疑] = 基于两轮经验预设的质疑应对

---

## P1 | Agent时代已来（75秒）

### 演讲话术

[可对外]

各位好。今天汇报的主题是Agent RL后训练。开门见山，先看三个2026年Q1发生的事。

第一，产品端。Cursor发布了Composer 2，用RL后训练让Coding Agent在部分benchmark上超越了Opus 4.6。同一时期，OpenAI的GPT-5.4在Terminal-Bench上拿到75.1分，远超Composer 2的61.7。两家都在Agent能力上投入RL训练，竞争格局已经白热化。

第二，开源端。OpenAI将Codex CLI以Apache 2.0协议开源，两周内拿到6.7万星。Agent工具链正在从闭源走向开放。

第三，资本端。据The Information报道，Anthropic领导层在讨论每年超过10亿美元级别的RL环境投入。注意是"讨论中的规模"，不是已投入金额。

一个背景数据：广义训练算力——包括预训练和后训练——占整体AI算力的约40%，其中后训练的份额在持续攀升。Agent产品在爆发，但下一个问题是：哪个领域的Agent最先从RL中真正获益？

[预判质疑]
- "40%的口径是什么？"——广义训练含预训练+后训练，后训练份额在攀升但不单独占40%。两轮质疑后此数据无争议
- "Composer 2只是cherry-picking一个benchmark"——已加GPT-5.4数据对比，叙事是"后训练战场已开"不是"谁最强"

---

## P2 | Coding/SE Agent RL是当前Sweet Spot（90秒）

### 演讲话术

[可对外]

回答刚才的问题：Coding和Software Engineering领域的Agent，是RL训练当前最先落地的sweet spot。

为什么是coding？有一个结构性原因：代码能编译、测试能通过——这是天然的可验证reward信号。后面P4会讲到Agent RL的三大挑战，其中评估挑战在coding领域是最轻的，因为你有确定的通过/不通过信号。相比之下，企业级Agent——比如CRM、ITSM领域——奖励信号模糊，RL闭环难得多。

看数据。我整理了coding/SE领域五个代表性案例。

Cursor的Composer 2，用Kimi K2.5做大规模RL训练，部分benchmark超越Opus 4.6。OpenAI的Codex，SFT加RL混合范式，Terminal-Bench 75.1%。Together.ai的DeepSWE，用改进版GRPO变体——注意不是vanilla GRPO，集成了DAPO的clip high机制、去掉了KL loss和entropy loss——在SWE-bench Verified上达到42.2%。Google在Agent评估中全流程引入RL，但公开技术数据有限。NVIDIA开源了ProRL Agent框架，SWE-bench成绩从9.6%提升到18.0%。

关键观察：在coding/SE领域，主要头部厂商的主流方向已转向引入RL。区别仅在于RL在整个流水线中的权重。混合范式是主流——SFT做基础能力注入，RL做闭环优化。

需要诚实说明的scope限定：这个判断在coding/SE领域成立。在企业级Agent生态中——微软的Agent Lightning、Salesforce的AgentForce——主要路线仍是SFT加工具编排，RL渗透率还比较低。本次汇报聚焦在coding/SE Agent领域。

[预判质疑]
- "你说'主要'不是'全部'了？"——对，Round 2被微软反例击穿后修正。coding/SE领域成立，企业级领域RL还不是主流
- "DeepSWE不就是随便拿个GRPO就行吗？"——不是。Qwen3-32B是一线base model，GRPO做了多项稳定性改造，200步高效训练背后有精细工程
- "Google那行没数据怎么凑五个？"——已标注"公开信息有限"，四个有数据的案例足够支撑论点

---

## P3 | 为什么必须是Agentic RL（75秒）

### 演讲话术

[可对外]

P2回答了"谁在做"，P3回答"为什么不可或缺"。

Agent的演进方向可以概括为一句话：从胖Harness瘦模型，走向瘦Harness胖模型。2024年，Agent的核心能力——上下文管理、工具选择、错误恢复——都靠harness代码实现，模型只负责生成文本。到2026年，这些能力正在被迁移到模型参数里。

最好的例子是Cursor Composer 2。它通过RL训练让模型学会self-summarization——200K token的上下文压缩到约1K token，compaction error降了50%。这个能力之前全靠harness代码，现在模型自己就能做。

这条迁移路径上，RL是不可或缺的关键环节。SFT能教模仿，但Agent状态空间太大，演示覆盖不了所有情况。DPO能教对齐，但Agent的奖励来自环境验证，不来自人类偏好。只有RL能实现环境反馈加闭环探索——系统性地把harness能力迁移到模型参数中。

但RL不是唯一方法。业界主流是SFT做基础注入加RL做闭环优化的混合范式。

[预判质疑]
- "Cursor做RL的前提是强base+顶级GPU集群，我们两个都没有"——对，RL路线有前提。我们的定位不是训模型，是提供训练平台
- "self-summarization的compaction error还有50%没解决"——对，说明方向对但技术未完全成熟

---

## P4 | 三大结构性变化 → 三大挑战（75秒）

### 演讲话术

[可对外]

从LLM的RL训练到Agent的RL训练，发生了三个结构性变化，直接产生三个未解挑战。

说明一下：下面这个三分法是我们自己的工程分析框架，按RL循环的归因、采集、评估三个环节切分。学术界有其他切法——比如GiGPO把归因和采集合并，微软按稳定性、泛化性、可扩展性切分。我们选这个切法是因为它直接对应三个可独立优化的工程环节，便于落地时分而治之。

三个变化。第一，轨迹长度跃升——从几百token变成几千到几万token。第二，单条经验成本爆炸——从秒级变成分钟级。第三，评估从确定变为模糊——代码有标准答案，但开放式Agent任务没有唯一正解。

三个变化对应三个挑战。长轨迹导致归因困难——关键步骤被淹没在长轨迹中。高成本导致采集瓶颈——每条rollout都很贵，必须让每条都"值"。评估模糊导致奖励信号不可信——可能被游戏。

需要说明的是，这三个挑战不是完全正交的。比如CSO解决归因需要额外PRM推理，本身加剧采集成本。CISPO提升采集效率但引入entropy崩塌风险影响评估稳定性。后面技术地图会标注这些交叉影响。

一个实战优先级的说明：上面的排序是因果链顺序，不是落地优先级。实战中评估应排第一——没有可靠的reward，一切都是空中楼阁。

[预判质疑]
- "你自创框架，凭什么让人信？"——标注清楚了是我们的框架不是学术共识，选择理由也给了。Round 2对此无新质疑
- "三个挑战有耦合你还分开讲？"——分开讲是为了工程层面分而治之，耦合关系在技术地图里显式标注

---

## P5 | 技术地图（90秒）

### 演讲话术

[可对外]

这页是导航页，三大挑战对应的业界前沿解法一览。

请注意几个关键信息。第一列是成熟度——从论文级到框架级到产品级，成熟度差异很大。第二列是已知局限——每项技术都有明确的已知限制。第三列是NPU适配状态——六项前沿技术在NPU上目前零落地。

[仅内部] 第四列是这次新增的——跨挑战影响。上一轮汇报我们把技术之间的trade-off藏在注释里，这次直接放在表格中。比如CSO，已知局限写的是"需PRM但Agent PRM不存在"，跨挑战影响写的是"加剧采集瓶颈"——因为CSO额外的推理开销本身就是一种采集成本。CISPO那行，跨挑战影响写的是"长轨迹weight衰减影响归因质量"。

页面下半部分分成两框。左边是基础设施层已有能力——注意标题，是基础设施层，不是前沿技术层。我们有MindSpeed RL基础训练、vllm-ascend推理引擎、5个算子精度基线。但需要说明：MindSpeed RL目前跑的是reasoning任务的GRPO训练，不是Agent RL。vllm-ascend支持的是标准LLM serving，Agent rollout的推理特征还没优化。

右边是前沿技术层待建能力。六项前沿技术的NPU适配均为待建。在前沿技术层面，我们的已有能力是零。

[预判质疑]
- "已有能力怎么看着像三项很多的样子？"——明确标注了是基础设施层，前沿技术层是零。不做稀释
- "MindSpeed RL不就是已经有Agent RL了吗？"——不是。MindSpeed RL是reasoning RL，Agent RL需要外部环境交互和sandbox，这是ProRL Agent要补的空白

---

## P6 | 归因挑战：CSO + SALT（60秒）

### 演讲话术

[可对外]

归因挑战——长轨迹中谁是关键决策者？两条技术路线。

CSO，Critical Step Optimization。核心思路是用过程奖励模型PRM识别关键步骤，只在这些步骤上做训练。效果：8B模型在GAIA-Text-103这个特定benchmark上追平GPT-4.1——注意是特定benchmark，GPT-4.1在GAIA上是中游水平。

CSO的最大问题：它需要PRM，但Agent任务的PRM基本不存在。数学推理领域有Qwen Math PRM可用，Agent领域空白。冷启动成本可能超过直接做全量GRPO训练。而且CSO的额外推理开销——PRM加expert model加验证rollout——本身加剧采集瓶颈。

SALT是另一条路——轻量级，用轨迹间的状态共享推算每步的advantage。计算开销几乎可忽略。但依赖轨迹间有足够状态共享，高随机环境下效果退化。

归因方向投入优先级低于采集和横切约束。两项技术在NPU上均未适配。

[预判质疑]
- "CSO那个8B追平GPT-4.1听起来很强啊"——仅GAIA-Text-103一个benchmark，GPT-4.1在该任务非SOTA。两轮质疑后不会再被追问

---

## P7 | 采集挑战：ProRL Agent（90秒）

### 演讲话术

[可对外]

采集挑战的核心：每条Agent rollout都很贵。NVIDIA在2026年3月开源了ProRL Agent，提出了Rollout-as-a-Service架构——把rollout的整个生命周期作为独立HTTP服务，从训练循环中解耦出来。

关键技术点：Token-in/token-out一致性保证。rollout产生的token ID直接传给trainer，全流程不做re-tokenization。这从架构层面避免了隐性off-policy偏差。效果：Qwen3-8B在SWE-bench Verified上从9.6%提升到18.0%。

为什么它是我们的第一优先级？四个理由。一，训推解耦加token一致性是所有Agent RL方法的公共基础设施——不管上层用CSO还是CISPO，都需要这个底座。二，NVIDIA刚开源一个月，快速跟进可在NPU生态建立先发优势。三，和vllm-ascend技术栈复用度最高。四，MindSpeed RL只覆盖训练侧，ProRL Agent正好补rollout和环境交互侧的空白。

[仅内部] Gate 1的量化标准已经和L4、L7一起定义清楚了：一个月后，单条SWE-bench rollout在910C单机上30分钟内完成、4条并发稳定运行、Singularity到Docker适配方案设计文档提交、并且记录rollout过程中环境操作的I/O延迟分布。存储I/O是L6提出的新观测维度——Agent rollout的小文件随机I/O和训练集群的大块数据吞吐特征相反，可能成为被忽视的瓶颈。

[预判质疑]
- "30分钟一条rollout太慢了"——30分钟是Gate 1的上限标准，给了3倍余量。H100基线5-8分钟，910C折算8-13分钟。Gate 2阶段会优化
- "vllm-ascend能支持Agent rollout推理吗？"——需要三项改造：动态batch、prefix caching跨轮复用、context挂起恢复。第一个月先用同步推理跑通，优化推到后续
- "存储I/O你之前没想到？"——确实是盲区，感谢L6提出。Gate 1增加观测项

---

## P8 | 评估挑战：Harness即奖励（75秒）

### 演讲话术

[可对外]

评估挑战的核心洞察：你优化的目标本身可信吗？

Agent任务的评估面临三层风险叠加。奖励稀疏——长轨迹中大部分步骤没有信号。Reward hacking——并行探索越多越容易发现奖励漏洞。Harness偏差——harness定义的"成功"和真实用户需求之间有系统性偏差。

一个具体例子。我们选SWE-bench作为第一个落地场景，它的reward信号是"测试全通过"。但真实软件工程不只是通过测试——代码可读性、设计合理性、性能影响都很重要。学术界已有论文指出Agent可能学会monkey-patching——写一个hack让测试通过但代码质量极差。

[仅内部] 这个风险不是被忽视的，是有计划缓解的。Gate 1到Gate 2阶段先用SWE-bench的二值reward做概念验证——先证明Agent RL在NPU上能跑能收敛。Gate 2到Gate 3阶段引入多维辅助评估信号：代码质量分、diff最小化、编译警告数等。先求有，再求好。

一个更深层的洞察：Harness质量决定训练方向。如果harness本身不可信，精度再高也没用——确定性地训出走捷径的Agent，可复现地产出垃圾。这意味着确定性训练和harness质量必须是双轮驱动。后面P10和P11会展开。

[预判质疑]
- "SWE-bench reward hacking你怎么办？"——分阶段。Gate 1-2用二值reward做概念验证，Gate 2-3引入多维评估
- "你说确定性训练+harness双轮驱动，harness工程谁做？"——Gate 2交付物已包含SWE-bench参考harness实现

---

## P9 | 横切约束：精度与一致性（90秒）

### 演讲话术

[仅内部]

横切约束——所有优化手段的宪法底线。这页对内部技术对齐最重要。

先看一手实战数据。在910B上跑GRPO，CANN的LayerNorm算子和CUDA在第5-6位有效数字不同。Reward曲线前200步和GPU几乎一致，300步后开始分叉，500步差距达到2到3个百分点。定位根因花了一周——LayerNorm的reduction order不同。结论：预训练能容忍的微小精度差异，在RL训练中会被importance ratio放大为系统性偏移。

目前的进展。LayerNorm的确定性实现验证完成——固定reduction order，性能开销约15%，可接受。Softmax类似，开销约12%。

关键瓶颈是AllReduce。CANN的AllReduce和NCCL在reduction order上差异大，不是简单参数配置问题。我们有两条方案路径：固定reduction tree拓扑，预估开销20%到30%；确定性ring-reduce替代tree-reduce，预估开销40%到50%但确定性更强。目前倾向第一条路。

性能阈值已经和L8对齐：30%以内方向成立继续推进，50%以上要重新评估方案。参考数据：CUDA上SGLang社区的batch-invariant inference开启后延迟增加约34%。如果我们端到端做到30%到40%的水平，和CUDA相当就不是劣势。

AllReduce的实测数据是Gate 1并行启动的第一个milestone。

[预判质疑]
- "15%和12%是单算子，端到端累积多少？"——好问题，端到端profile还没有做。但参考CUDA上SGLang的34%（工程优化后），我们的目标是相当水平
- "AllReduce如果超50%怎么办？"——重新评估方案。L8做技术评审

---

## P10 | 我们的差异化：确定性RL训练（75秒）

### 演讲话术

[可对外——部分内容]

差异化方向。如果昇腾能做到bitwise reproducibility——同样的代码、同样的数据、跑两次结果完全一样——这在RL领域是一个非常有吸引力的卖点。做RL训练的人极度痛恨不可复现性。你调了一个超参，reward上去了，不知道是超参的功劳还是随机性的功劳。

竞争态势：GPU阵营的batch-invariant-ops目前只在小模型上演示了bitwise consistent训练。大规模确定性RL训练仍是开放问题。如果我们先于CUDA生态实现大规模确定性RL训练，这是差异化。

[仅内部] 但我们在这一轮汇报中学到了一个更深的洞察：确定性训练本身不够，必须和harness质量形成双轮驱动。

确定性训练解决"训练结果可信赖"——同样配置跑两次结果一致，客户知道效果好坏归因于什么。Harness质量解决"训练方向正确"——确保Agent学到的是真正有用的能力，不是exploit奖励漏洞。

两者缺一不可。有确定性但没好harness——确定性地训出垃圾。有好harness但不确定性——好结果不可归因不可复现。两者兼具——可信赖、可复现、方向正确的Agent RL训练。

技术路径：第一，batch-invariant算子昇腾适配，和vLLM/SGLang社区合作。我们贡献昇腾backend换取社区支持。关键瓶颈是AllReduce，上一页已详细分析。第二，RL关键路径算子精度基线扩展。第三，BF16到FP16系统级评估——尚未启动。

[预判质疑]
- "客户真的愿意为可复现性付费吗？"——需要市场验证。路线图中建议和2-3个目标客户做用户调研
- "确定性训练+harness双轮驱动，你能同时做好吗？"——确定性训练需要专门团队，harness参考实现复用ProRL Agent已有的SWE-bench集成，不需大量额外人力

---

## P11 | 行动路线图（90秒）

### 演讲话术

[仅内部]

路线图。三阶段gate机制：1月跑通，3月Demo，6月发布。

Gate 1，一个月后，ProRL Agent单机跑通。四个量化check项：第一，单条SWE-bench rollout在910C上30分钟内完成。H100基线5到8分钟，折算到910C约8到13分钟，30分钟给了3倍余量。第二，4条并发rollout稳定运行，证明架构可横向扩展。第三，Singularity到Docker适配方案设计文档。第四，rollout过程中环境操作的I/O延迟分布数据——这是识别存储是否成为瓶颈的观测项。L4来check。通过后扩到5到8人团队。

并行启动的两件事。确定性RL方向：AllReduce实测数据是第一个milestone，L8做技术评审，30%以内方向成立。精度基线扩展：从5个算子扩展到RL关键路径全覆盖。

Gate 2，三个月后，可展示的客户Demo。五项交付物。SWE-bench Agent RL训练全流程在910C集群跑通。训推一致性和精度对齐展示。SWE-bench参考harness实现——文档加代码，让客户能照着跑。NPU-hour消耗数据——让客户对训练成本有量级感知。最后必须是可以带客户看的可视化Demo。L3来check。

Gate 3，六个月后，可发布的benchmark。包括可复现的SWE-bench Agent RL benchmark结果、batch-invariant推理集成、精度基线全覆盖，以及多维评估信号集成来缓解SWE-bench的reward hacking风险。

资源需求：ProRL Agent适配5到8人，确定性RL训练3到5人，总计8到13人。Harness参考实现不需要额外大量人力，复用ProRL Agent已有的SWE-bench集成做昇腾适配。两个方向并行推进，技能栈不完全重叠。

[预判质疑]
- "8-13人够吗？"——两个方向并行，技能栈不同。ProRL Agent偏推理引擎和容器编排，确定性RL偏算子开发和CANN内核
- "Gate 2的harness参考实现会不会分散精力？"——复用ProRL Agent已有SWE-bench集成，做的是适配+整理，不是从零开发
- "如果Gate 1存储I/O是瓶颈怎么办？"——适配方案复杂度增加，可能需要存储侧优化或调整rollout的环境操作方式

---

## P12 | 收束：竞争定位与客户价值（75秒）

### 演讲话术

**对外版本**（当前阶段使用）：

[可对外]

最后做一个收束。我们的竞争定位不是对标NVIDIA比谁的卡更快。910C单卡推理性能约为H100的60%，纯FLOPS比拼我们没有优势。

我们的定位是：让国内客户的Agent RL从"不能做"变为"能做"。

现实是国内大量客户受供应链约束。他们的替代选项不是H100，而是放弃Agent RL。我们正在投入让国产算力平台支持Agent RL训练。

第一目标客户是国内做Agent产品的公司——有模型、有Agent场景、受供应约束。第一场景是Software Engineering Agent的RL训练——coding/SE领域是RL的sweet spot，可验证reward信号是结构性优势。

ProRL Agent Rollout-as-a-Service是第一步——搭建公共底座。确定性RL训练是差异化——做GPU阵营没做到的事。参考harness实现是客户体验——确保客户用得起来。

---

**内部版本**（Gate 2通过后可对外升级）：

[仅内部]

Agent时代的竞争，不在谁的模型更大，而在谁能让Agent更快地从做事中学会做事。

我们的价值主张是：在国产算力平台上，让客户能做Agent RL训练，并且训练结果可信赖、可复现。

这句话在Gate 2 Demo出来之前不对外使用。对外先用刚才的务实版本——"我们正在投入让国产算力平台支持Agent RL训练"。等有了benchmark结果再升级叙事。

[预判质疑]
- "你们连Agent RL都没跑通就来定义竞争维度？"——这就是为什么内部版暂不对外。Gate 2 Demo是对外叙事的前提
- "客户真的买不到H100吗？需要量化验证"——确实需要和2-3个目标客户做调研，确认供应约束的实际程度
- "确定性训练客户真的在乎吗？"——RL从业者极度痛恨不可复现性是技术共识，但付费意愿需市场验证

---

## 附录：预判质疑速查表

> 基于两轮共16个Phase的质疑经验整理。按"被问到的概率"排序。

| 排序 | 预判质疑 | 预设回应要点 | 两轮经验来源 |
|------|---------|-------------|-------------|
| 1 | "我们今天有什么能拿出手的？" | 基础设施层有MindSpeed RL+vllm-ascend+精度基线，前沿技术层零落地。这是投入诉求的原因 | Round 1 L3/L4 |
| 2 | "为什么不直接用H100？" | 供应链约束+我们的差异化在确定性训练不在单卡FLOPS | Round 1 L3 |
| 3 | "AllReduce开销太大怎么办？" | 30%以内方向成立。超50%重新评估。L8做技术评审 | Round 2 L8+L7 |
| 4 | "SWE-bench训出来的Agent会走捷径" | 分阶段：Gate 1-2二值reward概念验证，Gate 2-3多维评估信号 | Round 2 L8 |
| 5 | "Harness工程谁负责？" | Gate 2交付物含参考harness实现，复用ProRL Agent的SWE-bench集成 | Round 2 L2+L1 |
| 6 | "这个scope是不是太窄了？只做coding？" | Coding/SE是RL sweet spot有结构性原因。验证通过后可扩展到其他可验证领域 | Round 2 L8解释 |
| 7 | "DeepSWE说明GRPO就够了，为什么要做ProRL Agent？" | DeepSWE验证RL方法有效，但它的rollout还是在GPU上跑。ProRL Agent解决的是rollout基础设施 | Round 2 L8+L6 |
| 8 | "8-13人做两个方向够吗？" | 并行推进、技能栈不重叠。harness复用已有集成不需大量额外人力 | Round 2讨论 |
| 9 | "3个月后Demo能给pricing吗？" | 不能给pricing但能给NPU-hour消耗数据，客户自己算成本 | Round 2 L3 |
| 10 | "你这个叙事对外讲会不会被打脸？" | 内部版暂不对外。对外用务实版"我们正在投入"。Gate 2后升级 | Round 2 L1+L8+L3 |

---

## 附录：全程时间分配

| 页码 | 内容 | 目标用时 | 内/外 |
|------|------|---------|-------|
| P1 | Agent时代已来 | 75秒 | 可对外 |
| P2 | Coding Agent RL是sweet spot | 90秒 | 可对外 |
| P3 | 为什么必须是Agentic RL | 75秒 | 可对外 |
| P4 | 三大变化→三大挑战 | 75秒 | 可对外 |
| P5 | 技术地图 | 90秒 | 部分仅内部 |
| P6 | 归因：CSO+SALT | 60秒 | 可对外 |
| P7 | 采集：ProRL Agent | 90秒 | 部分仅内部 |
| P8 | 评估：Harness即奖励 | 75秒 | 部分仅内部 |
| P9 | 横切约束 | 90秒 | 仅内部 |
| P10 | 差异化：确定性RL训练 | 75秒 | 部分可对外 |
| P11 | 行动路线图 | 90秒 | 仅内部 |
| P12 | 收束 | 75秒 | 分版本 |
| **总计** | | **约16分钟** | |