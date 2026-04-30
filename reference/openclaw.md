---
project: OpenClaw
last_updated: 2026-04-30
refresh_policy: "超过 3 个月必须用 WebSearch 重新验证并刷新本文件"
verified_sources:
  - https://github.com/
---

# OpenClaw（开源之王）

## 核心数据

- **GitHub Stars**: 250,000+
- **企业渗透率**: 22%（shadow-IT使用率）
- **核心理念**: Local-first，数据安全

## 核心架构

```
OpenClaw 架构全景：

┌─────────────────────────────────────────────────────────────┐
│                    SOUL.md（Agent 定义）                      │
│    - 身份、规则、工具、行为                                    │
│    - Markdown 配置文件                                        │
│    - Local-first 数据安全                                     │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Gateway（控制平面）                        │
│    - 管理 LLM 连接                                            │
│    - 管理消息渠道（Discord/Slack/Telegram）                   │
│    - 管理会话                                                  │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Model Layer（模型层）                      │
│    - Model-agnostic                                          │
│    - Claude Opus 4.6 / Sonnet 4.6                            │
│    - GPT-5 / Gemini 3.1 Pro                                  │
│    - Ollama（本地模型）                                        │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Skills System（技能系统）                  │
│    - community-contributed modules                           │
│    - agentskills.io Skills Hub                               │
│    - 可扩展能力模块                                            │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    MCP Integration（MCP 集成）                │
│    - Native MCP Server support（2026-03）                    │
│    - 能力边界扩大 3-4 倍                                       │
│    - HTTP/SSE Transport                                      │
│    - Dynamic Tool Discovery                                  │
└─────────────────────────────────────────────────────────────┘
```

## 关键特性

| 特性 | 描述 | SOTA价值 |
|-----|------|---------|
| **SOUL.md** | Markdown配置文件定义Agent | 🔴 最高 |
| **Gateway** | 多平台统一接入 | 🟡 高 |
| **Model-agnostic** | 支持多种LLM | 🔴 最高 |
| **Skills System** | 社区贡献模块 | 🟡 高 |
| **MCP Native** | 原生MCP支持，能力扩大3-4倍 | 🔴 最高 |
| **Local-first** | 本地优先，数据安全 | 🔴 最高 |

## 参考价值

- **开源最佳实践**：250k stars，社区验证
- **MCP集成**：能力扩展标准
- **Local-first**：数据安全架构
- **Skills System**：可扩展能力模块

## 官方资源

- OpenClaw GitHub: https://github.com/openclaw/openclaw
- Skills Hub: https://agentskills.io