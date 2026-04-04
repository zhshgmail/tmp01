# 停车场：发散议题待深入讨论

> 演讲过程中出现的有价值但超出本次范围的议题，记录在此用于后续专题讨论。

## Phase 2 产生的待深入议题

1. **Agent RL算力市场规模测算**（L1-战略VP提出）：演讲者无法回答"不做会怎样"的量化商业后果，需要补充TAM/SAM测算
2. **910C全栈优化 vs H100单卡性能的量化对比框架**（L3-产品总裁提出）：需要明确在Agent RL特定负载下，全栈优化能将60%的裸性能差距缩小到多少
3. **第一个可交付产品的定义和时间线**（L4-工程VP提出）：需要定义MVP——是一个RL训练框架？一个benchmark套件？还是一个端到端demo？给出6个月里程碑

## Phase 3 产生的待深入议题

4. **Agent RL rollout推理负载特征 vs LLM serving负载特征的量化对比**（L7-推理优化专家提出）：Agent推理是多轮、动态上下文、工具调用中断的，和continuous batching优化目标不同。需要量化现有推理优化对Agent RL rollout的适配程度，以及对标Dynamo KV-aware routing的方案
5. **910C训练可靠性数据：长时间RL训练的稳定性实测**（L6+L8联合提出）：DeepSeek报告指出910C在长时间训练可靠性上有差距。需要实测数据——连续多天RL训练的崩溃率、算子精度漂移、reward曲线抖动等
6. **Composer 2 router replay机制对昇腾MoE训练的启示**（L8-后训练专家提出）：Cursor为解决MoE训推一致性开发了router replay。昇腾上跑MoE RL是否需要类似机制？CANN算子精度差异是否会放大MoE routing的不确定性？
7. **竞争定位重构：从"对标NVIDIA"到"让国内客户能做"**（L1+L3联合提出）：演讲者的reframing有说服力——国内客户的替代选项不是H100而是"不做"。但需要量化验证：国内头部公司（字节、腾讯等）在Agent RL上的实际算力采购约束是什么？他们真的买不到H100吗？

## Phase 4 产生的待深入议题

8. **CSO全流程计算开销量化**（L2-首席架构师提出）：CSO需要PRM训练、expert model替代动作生成、policy model验证rollout三步。总计算成本与基线GRPO全量训练相比是多少？CSO解决归因却加剧采集——这个trade-off的量化数据缺失
9. **CISPO训练崩溃缓解方案**（L8-后训练专家提出）：DISPO论文明确指出CISPO后期entropy急剧下降导致性能崩溃。替代方案（DISPO、CE-GPPO等）在NPU上的可行性评估。如果推荐CISPO给客户，必须同时给出稳定性保障方案
10. **六项前沿技术的基础设施需求兼容性矩阵**（L6-AI基础设施提出）：CSO需要多模型交替推理、CISPO需要大batch异步训练、ProRL Agent需要训推完全分离架构——这些需求是否互相兼容？能否在统一的基础设施上支持？工程量评估
11. **三挑战之间的耦合与trade-off分析**（L2-首席架构师提出）：演讲者声称三挑战"正交、互不重叠"，但L2用CSO案例证明归因解法会加剧采集问题。需要系统分析六项技术方案之间的交互效应和trade-off
12. **技术地图的"实线vs虚线"标注体系**（L3-产品总裁提出）：六项前沿技术在NPU上零项可用，但技术地图未区分"已有能力"和"待建能力"。正式PPT需要增加成熟度、已知风险、NPU适配状态三列

## Phase 5 产生的待深入议题

13. **Agent任务PRM的训练路径与冷启动成本**（L8-后训练专家提出）：CSO的前提是有一个靠谱的process reward model，但Agent任务的PRM基本不存在（数学推理有Qwen Math PRM，Agent领域空白）。需要评估：从零训练Agent PRM的数据标注成本、算力成本，以及是否有bootstrapping方案（如用outcome reward反向蒸馏step-level信号）
14. **CISPO/DISPO在100步以上Agent轨迹中的importance weight衰减实验**（L7+L8联合提出）：CISPO的2x加速数据来自数学推理（轨迹几十步内），对Agent任务100步以上长轨迹，单步importance ratio 0.95的累积效应导致轨迹级weight趋近于零。需要设计实验量化这个衰减对训练信号的实际影响
15. **Eagle3投机推理在结构化输出场景下的准确率**（L8-后训练专家提出）：Agent推理包含大量JSON/API调用等结构化token序列，Eagle3的draft model对这类非自然语言输出的投机准确率没有数据。如果准确率低，投机推理反而增加开销。需要在典型Agent工具调用场景下实测
16. **ProRL Agent昇腾适配详细工程评估**（L4+L6+L7联合提出）：演讲者初估5-8人3个月PoC，但需要细化：vllm-ascend推理引擎对rollout动态batch和长上下文管理的支持程度、Singularity→Docker sandbox层改造工程量、HTTP通信延迟在昇腾集群内网拓扑下的实测数据、与现有MindSpeed RL训练流水线的集成方案
17. **昇腾集群sandbox编排能力建设**（L6-AI基础设施提出）：Forge需要Firecracker级micro-VM调度（Cursor每秒500+ pods），ProRL Agent需要Singularity容器运行时。当前昇腾集群为batch训练优化，缺乏大规模sandbox编排能力。需要评估在NPU集群上引入轻量级VM/容器编排的可行性和工程量

## Phase 6 产生的待深入议题

18. **batch-invariant-ops的昇腾适配路径**（L8-后训练专家提出）：vLLM/SGLang社区已有CUDA上的开源batch-invariant算子库（batch-invariant-ops），实现了bitwise consistent RL训练（Qwen3-1.7B演示）。需要评估：昇腾适配的工程量（CANN AllReduce reduction order与CUDA不同）、与社区合作的模式（贡献昇腾backend换取社区支持）、从小模型到大模型的扩展性
19. **CANN vs CUDA精度基线完整覆盖计划**（L8+L6联合提出）：目前仅完成5个RL关键路径算子的CANN vs CUDA数值比对，发现LayerNorm和AllReduce差异最大。需要扩展到全部关键算子（Softmax、FlashAttention、Embedding等），建立量化精度档案，并制定修复优先级排序。L8的实战数据：LayerNorm精度差异导致reward曲线300步后与GPU偏移2-3个百分点
20. **BF16→FP16切换的系统级impact评估**（L8-后训练专家提出）：最新研究（arXiv 2510.26788）证明BF16的mantissa位数不够是训推不一致的主要根因，切FP16可消除大部分mismatch。但昇腾训练流水线多处hardcode BF16，FP16的dynamic range小导致大模型训练overflow风险。需要评估改造工程量和精度/稳定性trade-off
21. **CANN Next精度行为内部验证**（L6+L7联合提出）：950PR的CANN Next提升了编程兼容性（CUDA-like thread blocks/warps），但L6确认BF16/FP16算子行为与当前CANN差异不大。需要内部验证：CANN Next的精度对齐到底做到什么程度，是否能在不改应用代码的情况下缩小和CUDA的数值差异
22. **3个月PoC客户demo定义**（L3-产品总裁提出）：3个月后的ProRL Agent PoC需要是"可以带客户看的东西"，不是内部benchmark。需要定义demo的形态：是否包含可视化的训练过程、SWE-bench解题成功率对比、训推一致性的A/B对比展示等
23. **确定性RL训练作为差异化卖点的市场验证**（L1+L5联合提出）：L8确认RL从业者极度痛恨不可复现性，GPU阵营大规模确定性训练仍是开放问题。但"客户是否愿意为可复现性付费选择昇腾"需要市场验证——与2-3个目标客户做用户调研，确认这个卖点的付费意愿
