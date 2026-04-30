# Agent Evaluator

> **AI Agent 项目架构评价工具 - 对标 2026 年 SOTA 标准**

[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/yourusername/agent-evaluator.svg)](https://github.com/yourusername/agent-evaluator)

## 为什么需要这个工具?

AI Agent 领域发展极快,2022年的 ReAct 正则解析已是旧范式。本工具帮助你:

- **识别架构差距**: 对比 Anthropic、OpenAI、Manus 等最新方案
- **学习最佳实践**: 通过七大里程碑理解 Agent 设计原理
- **避免常见陷阱**: 指出正则解析、无沙箱、无记忆等旧范式问题

## 核心特性

- ✅ **SOTA 对标**: 强制搜索最新信息,拒绝过时知识
- ✅ **七大里程碑**: 感知层、大脑层、协议层、行动层、记忆层、工程化、安全性
- ✅ **生产级验证**: 参考 Anthropic MCP、OpenAI Function Calling、LangGraph 等经过大规模验证的方案
- ✅ **第一性原理**: 每个评价都有设计原理支撑,拒绝"魔法 Prompt"

## 快速开始

### 在任何 Agent 中使用

本 Skill 是一个通用的 Agent 评价工具,可以在任何支持 Skill 系统的 Agent 环境中使用:

1. **安装 Skill**: 将本 skill 复制到你的 Agent 的 skills 目录中
   - Trae: `.trae/skills/agent-evaluator/`
   - Claude Code: `.claude/skills/agent-evaluator/`
   - 其他 Agent: 根据你的 Agent 的 Skill 目录结构放置

2. **调用 Skill**: 在 Agent 中输入评价请求
   ```
   评价我的 Agent 项目架构
   分析这个 Agent 项目的 SOTA 对标差距
   对比这个 Agent 与 Anthropic/OpenAI 的架构差异
   ```

3. **获取报告**: Skill 会自动执行 SOTA 搜索并生成详细的评估报告

### 评估报告示例

```markdown
## 项目整体概况
- 项目类型: ReAct + 正则解析的本地 Agent
- 设计思路: 通过 Thought-Action-Observation 循环实现自主执行

## SOTA 对标差距
- ❌ 协议层: 使用正则解析(2022范式),应升级到 MCP 或 Function Calling
- ❌ 安全性: 无沙箱隔离,存在误删文件风险
- ✅ 大脑层: 实现了 ReAct 循环,但缺少 Plan-and-Execute
```

## 七大评估里程碑

| 里程碑 | 通过标准 | 为什么这是标准 |
|:---|:---|:---|
| **感知层** | 多模态反馈闭环 | Agent 的行动能力受限于感知能力 |
| **大脑层** | Plan-and-Execute 或自反思循环 | LLM 推理是概率性的,需要规划+反思 |
| **协议层** | MCP 或结构化函数调用 | LLM 输出是概率性的,正则解析不兼容 |
| **行动层** | 异步/并行执行与错误容错 | 生产环境有多点故障,需要容错机制 |
| **记忆层** | 分层管理(向量长时记忆 vs 上下文短时记忆) | 上下文窗口有限,需要分层管理 |
| **工程化** | 上下文滑动窗口、Token 成本控制与自动化评测 | Token 成本昂贵,需要成本控制 |
| **安全性** | 执行沙箱、路径白名单及敏感操作的人工确认机制 | Agent 会执行代码,需要沙箱隔离 |

## SOTA 对标选择逻辑

### 为什么选择这些公司?

```
选择优先级:

1. 【行业标准制定者】(最高优先级)
   - Anthropic (MCP): MCP 是开放标准,Anthropic 是标准制定者
   - OpenAI (Function Calling): Function Calling 是实践标准
   
2. 【生产级验证】(高优先级)
   - LangGraph: 25k stars,被 Klarna、Replit、Elastic 采用
   
3. 【最新范式】(高优先级)
   - Manus: 完全自主执行 + Sandbox 隔离
   - Hermes: 自进化记忆 + $1B 估值
```

### 为什么不参考其他?

| 不参考的项目 | 原因 |
|:---|:---|
| AutoGPT | 2022 年 ReAct 正则解析范式,不是 2026 SOTA |
| CrewAI | 对话式编排不可控,不适合生产环境 |
| 小众项目 | stars < 10k,无生产验证 |

## 文档

- [评估标准详解](reference/evaluation-standards.md) - 七大里程碑的深度解释
- [Anthropic 参考](reference/anthropic.md) - MCP 架构详解
- [OpenAI 参考](reference/openai.md) - Function Calling 架构详解
- [示例报告](examples/sample-reports/) - 真实评估案例

## 贡献

欢迎贡献新的 SOTA 参考文档或改进评估标准! 请阅读 [CONTRIBUTING.md](CONTRIBUTING.md)

## 许可证

MIT License - 详见 [LICENSE](LICENSE)

## 致谢

本工具的设计灵感来源于:
- Anthropic 的 MCP 协议
- OpenAI 的 Function Calling 最佳实践
- LangGraph 的 StateGraph 架构
- Manus 的自主执行范式