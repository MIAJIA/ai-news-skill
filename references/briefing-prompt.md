# Briefing Prompt

请整理过去 3 天内的重要动态，并输出一份高信噪比的技术简报。

## 要求

1. 每条信息必须注明来源账号
2. 区分【事实】与【观点】
3. 避免简单转述，要提炼核心判断
4. 重点关注技术信号，而不是营销信息

## 输出结构

### ① 今日速览（Top Signals）

只挑选 5 条最重要的动态，包括：
- 重大的 AI 模型或产品发布
- 技术路线变化
- 重要行业判断

每条包含：
- 【事实】发生了什么
- 【观点】发布者或行业的判断
- 【工程含义】这对 AI 系统设计或工程实践意味着什么

### ② AI 公司与开发者发布（Builder's Changelog）

追踪 AI 企业与开发者的重要发布，例如：
- 新模型
- API 变化
- 推理性能变化
- 定价变化
- 新模态能力
- 开源模型或权重

忽略：
- UI 更新
- marketing

重点分析：
这些发布对 **开发者生态或系统架构** 的实际影响。

### ③ 工程师 / 研究者观点

总结研究者和工程师发布的技术讨论，例如：
- 新架构
- agent 系统
- reasoning 模型
- evaluation 方法
- 长上下文处理
- inference 优化

要求：
提炼其核心技术判断，而不是简单转述。

### ④ 创业者 / 企业家观点

关注 AI 创业者对产品方向和应用层的判断，例如：
- AI agent
- workflow automation
- vertical AI
- 新应用模式

重点提炼：
这些判断反映了 **哪些新的产品机会或趋势**。

### ⑤ VC / 投资信号

总结投资机构或 VC 的重要观点，例如：
- AI 创业方向
- 投资热点
- 技术路径判断

不要只记录融资新闻，而要分析：
这些投资叙事反映了市场对哪些技术路径的认可，例如：
- agent 平台
- vertical AI
- infra 层
- data 层

### ⑥ Architect Decision Lens（架构决策视角）

从 AI 系统架构师的角度总结：
1. 今天最值得 reconsider 的技术假设是什么？
2. 哪个技术趋势可能在 6-12 个月内改变 production architecture？
3. 哪个 hype 最可能被证明是 false signal？
4. 如果设计新的 AI 系统，今天的最佳实践会发生什么变化？

### ⑦ One Pattern to Remember

最后提炼一个最值得记住的 "AI Engineering Pattern"。

例如：
- **Pattern**: Reasoning models are shifting evaluation from accuracy → process supervision.
- **解释**: 这个模式对未来 AI 系统设计意味着什么。

## 输出要求

- 中文
- Bullet points
- 高信噪比
- 信息密度高
- 避免冗长描述
- 语言像技术 briefing
- 每条信息注明来源账号或出处
