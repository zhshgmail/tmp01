# 9项技术工业分量评估与替换建议

> 基于2026年4月实际搜索验证，对标OpenAI/Google/Meta/NVIDIA/MiniMax/Cursor/ByteDance最新实践。

## 一、无需替换（工业级确认）

| # | 技术 | 验证结论 |
|---|------|---------|
| 1 | **Off-Policy PPO** | 各家基座算法。Cursor Composer 2用异步RL on Blackwell GPU；OpenRLHF已支持PPO/DAPO/GRPO全家桶 |
| 4 | **Dynamo (NVIDIA)** | NVIDIA生产级推理框架，ProRL Agent直接集成NeMo Gym生态，2026.03刚发布 |
| 5 | **LMCache** | vLLM官方集成的KV Cache复用方案，Cursor/OpenRLHF等框架均依赖vLLM推理后端 |
| 6 | **Mooncake (月之暗面)** | 生产级Prefill-Decode分离架构，已被多家公司参考部署 |

## 二、需替换的5项技术

### 替换1: PRIME-RL --> DAPO (ByteDance) + CISPO (MiniMax)

| 维度 | 旧: PRIME-RL | 新: DAPO + CISPO |
|------|-------------|-----------------|
| **来源** | PrimeIntellect小团队 | ByteDance Seed + MiniMax |
| **工业验证** | 仅学术实验(7B) | DAPO: AIME 50分(32B); CISPO: MiniMax M2.5达SWE-Bench 80.2% |
| **核心优势** | 隐式PRM无需过程标签 | DAPO四大技术(Clip-Higher/动态采样/Token级梯度/超长奖励整形); CISPO对IS权重裁剪，比DAPO快2x |
| **生态** | 独立仓库 | DAPO已集成OpenRLHF; CISPO驱动MiniMax Forge框架 |

**替换理由**: PRIME-RL的隐式PRM思想有价值但缺乏工业验证。DAPO已被OpenRLHF等主流框架集成，是2026年开源RL的事实标准；CISPO在200K上下文、百万级日吞吐下验证过。

### 替换2: SkyRL --> ProRL Agent (NVIDIA)

| 维度 | 旧: SkyRL | 新: ProRL Agent |
|------|----------|----------------|
| **来源** | UC Berkeley学术 | NVIDIA，2026.03发布 |
| **架构** | Rollout耦合在Trainer内，I/O阻塞GPU | Rollout-as-a-Service，三阶段异步流水线(INIT/RUN/EVAL) |
| **性能** | 基线 | Qwen3-8B在SWE-Bench: 9.6%-->18.0%(近2x); Qwen3-14B: 15.4%-->23.6% |
| **生态** | 独立开源 | 集成NeMo Gym，支持Singularity沙箱，Token ID全链路透传 |

**替换理由**: NVIDIA论文直接点名SkyRL/VeRL-Tool的耦合缺陷。ProRL Agent的解耦架构是工业级Agent RL的正确方向，且已开源。

### 替换3: SALT (Amazon) --> Cursor Real-Time RL + Meta Self-Play SWE-RL

| 维度 | 旧: SALT | 新: Cursor实时RL + Meta SSR |
|------|---------|---------------------------|
| **来源** | Amazon学术论文 | Cursor(产品级) + Meta FAIR |
| **解决问题** | 长horizon稀疏奖励的步级优势分配 | Cursor: 5小时端到端RL闭环，日内多次部署; Meta SSR: 自博弈生成无限训练数据 |
| **工业验证** | 论文阶段 | Cursor Composer 2已上线数百万用户; SSR在SWE-Bench Verified提升+10.4分 |
| **基础设施** | 无 | Cursor: Anyrun平台(10万+沙箱环境); Meta: 仅需沙箱仓库，零人工标注 |

**替换理由**: SALT解决的"长horizon信用分配"问题，工业界用两种更务实的方式解决——Cursor用极快的实时反馈闭环缩短horizon，Meta用自博弈生成自带奖励的训练数据。

### 替换4: SWE-PRM --> AgentPRM / 隐式步级奖励 (iStar)

| 维度 | 旧: SWE-PRM | 新: AgentPRM + iStar |
|------|-----------|---------------------|
| **来源** | 小众学术 | AgentPRM(多机构); iStar(隐式步级奖励) |
| **方法** | SWE专用过程奖励模型 | Actor-Critic范式+蒙特卡洛rollout计算奖励目标，最小修改即可集成现有RLHF流水线 |
| **趋势** | 领域窄 | 2026主流趋势：不单独训练PRM，而是用隐式方法从outcome reward提取过程信号 |

**替换理由**: 独立训练SWE-PRM成本高且泛化差。工业界趋势是"免费获得过程奖励"——通过隐式方法从结果奖励中提取步级信号，无需额外标注。

### 替换5: HiPER --> MiniMax Forge框架级分层奖励

| 维度 | 旧: HiPER | 新: Forge分层设计 |
|------|----------|-----------------|
| **来源** | 小众学术 | MiniMax Forge，2026.02发布 |
| **方法** | 学术分层奖励 | Agent与训练引擎完全解耦，支持任意Agent脚手架+工具，树状样本合并策略 |
| **验证** | 论文实验 | M2.5在20万+真实环境中训练，日吞吐百万级，40x训练加速 |

**替换理由**: HiPER的分层奖励思想在Forge的框架级设计中已被涵盖——Forge天然支持多层Agent的奖励传递，且经过超大规模验证。

## 三、替换总表

| 旧技术 | 新技术 | 来源巨头 | 搜索验证状态 |
|--------|--------|---------|------------|
| PRIME-RL | **DAPO + CISPO** | ByteDance + MiniMax | DAPO: OpenRLHF集成; CISPO: M2.5验证 |
| SkyRL | **ProRL Agent** | NVIDIA | 2026.03发布，NeMo Gym开源 |
| SALT | **Cursor实时RL + Meta Self-Play SWE-RL** | Cursor + Meta | Composer 2上线; SSR ICLR 2026 |
| SWE-PRM | **AgentPRM / iStar隐式步级奖励** | 多机构+社区共识 | 2026主流趋势，OpenRLHF支持 |
| HiPER | **Forge框架级分层设计** | MiniMax | M2.5生产验证，40x加速 |

## 四、关键信号

1. **Cursor Composer 2** (2026.03): 自研MoE模型+MXFP8低精度训练+全异步RL+Anyrun沙箱，5小时端到端闭环，是Agent RL工程化的标杆。
2. **NVIDIA ProRL Agent** (2026.03): Rollout-as-a-Service架构终结了SkyRL式耦合设计，已成为多回合Agent RL的基础设施标准。
3. **MiniMax Forge + CISPO** (2026.02): 200K上下文+百万日吞吐+40x加速，证明CISPO在超长horizon Agent场景比DAPO/PPO更高效。
4. **Meta Self-Play SWE-RL**: 零人工标注的自博弈范式，ICLR 2026接收，SWE-Bench +10.4分提升。
5. **DAPO**: 已成为开源Agent RL的事实标准算法，OpenRLHF全面支持。
