---
project: Hermes Agent
last_updated: 2026-04-30
refresh_policy: "超过 3 个月必须用 WebSearch 重新验证并刷新本文件"
verified_sources:
  - https://preview.aclanthology.org/master-new-author-system-ui/2025.emnlp-main.1318.pdf
---

# Hermes Agent（自进化框架）

## 核心数据

- **开发团队**: Nous Research（2023年成立，Saratoga CA）
- **融资**: Paradigm领投$50M A轮，估值$1B
- **发布时间**: 2026-02-25
- **开源协议**: MIT
- **部署成本**: $5/mo VPS即可运行

## 核心架构

```
Hermes Agent 架构全景：

┌─────────────────────────────────────────────────────────────┐
│                    自进化记忆系统                              │
│    - Honcho memory                                           │
│    - 持久化 + 自我改进                                        │
│    - 跨会话记忆                                               │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    40+ 内置工具                               │
│    - Terminal、Browser、File                                  │
│    - Voice Mode、Persistent Shell                            │
│    - Chrome CDP browser connect                              │
│    - ACP IDE integration                                     │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    多平台 Gateway                             │
│    - CLI、Discord、Slack、Telegram                            │
│    - Unified Streaming Infrastructure                        │
│    - Real-time token-by-token delivery                       │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Plugin Architecture                        │
│    - First-class plugin system                               │
│    - Vercel AI Gateway integration                           │
│    - Smart approvals                                         │
└─────────────────────────────────────────────────────────────┘
```

## 关键特性

| 特性 | 描述 | SOTA价值 |
|-----|------|---------|
| **自进化记忆** | Honcho memory，跨会话自我改进 | 🔴 最高 |
| **40+ 内置工具** | 丰富的工具集 | 🟡 高 |
| **多平台Gateway** | CLI/Discord/Slack/Telegram | 🟡 高 |
| **Plugin Architecture** | 一流插件系统 | 🟡 高 |
| **低成本部署** | $5/mo VPS | 🔴 最高 |

## 参考价值

- **自进化记忆**：跨会话学习能力
- **插件架构**：可扩展性设计
- **低成本部署**：生产级方案
- **多平台支持**：统一接入架构

## 官方资源

- Hermes GitHub: https://github.com/nousresearch/hermes
- Nous Research: https://nousresearch.com