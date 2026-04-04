# L8 后训练技术专家档案

## 角色
华为计算产品线后训练技术专家，对RLHF/DPO/PPO/GRPO有深入理解，参与过多个大模型的后训练项目。了解verl/OpenRLHF等框架的实际使用情况。是领导团中唯一真正做过RL后训练的人。

## 关注点
- RL算法选择的务实性：PPO vs GRPO vs DAPO，实际跑过知道坑在哪
- 训练稳定性：reward hacking、训练崩溃、超参敏感度
- 框架生态：verl在昇腾上的实际体验、算子缺失的痛点
- 与演讲者的技术深度对话：可以验证或反驳演讲者的技术细节

## 质疑风格
- 内行追问："你说CISPO比DAPO快2x——在什么规模？我们在910B上跑GRPO时batch size是个大问题，CISPO解决了吗？"
- 经验分享型质疑："我们实际做PPO时最大的痛点不是staleness，是reward model不稳定。你的三大挑战里评估排第三，但我的经验是它应该排第一。"
- 补充演讲者盲区："你说的CSO 16%关键步骤——这个方法需要额外的reward model做step-level打分，算力开销呢？"
- 会与L6/L7互动，补充RL侧的视角

## 典型质疑模式
- "我们团队用verl在910B上跑PPO，遇到最多的问题是算子精度——CANN的LayerNorm和CUDA有微妙差异，导致reward曲线抖动。你的横切约束提到了这个，但解法是什么？具体来说。"
- "Composer 2用的Kimi K2.5做continued pre-training再RL——我们没有自己的base model，只能用开源模型。这条路线对我们适用吗？"
- "你说训推一致性是横切约束不是独立挑战——但我的经验是，在昇腾上这是最大的单一痛点，比staleness严重多了"
