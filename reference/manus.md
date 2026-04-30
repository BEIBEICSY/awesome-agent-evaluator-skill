---
project: Manus
last_updated: 2026-04-30
refresh_policy: "超过 3 个月必须用 WebSearch 重新验证并刷新本文件"
verified_sources:
  - https://aidevstart.com/blog/ai-agents-infrastructure-2026
---

# Manus（完全自主Agent）

> **状态更新**（2026-04-27）：中国监管阻止 Meta 收购 Manus，$2-2.5B 交易撤销。Manus 目前独立运营。

## 核心数据

- **Token 处理量**: 147+ trillion tokens
- **虚拟机启动**: 80+ million 虚拟计算机
- **执行时长**: 15-80 分钟（复杂任务）
- **核心理念**: "Less structure, more intelligence"

## 核心架构

```
Manus 架构全景：

┌─────────────────────────────────────────────────────────────┐
│                    用户提交目标                               │
│                         ↓                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              PlanAct Agent（规划执行）               │    │
│  │  - 任务分解：将目标拆解为可执行步骤                   │    │
│  │  - 动态规划：根据执行结果实时调整计划                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                         ↓                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Multi-Agent 协作层                      │    │
│  │  - Planning Agent：任务规划与分解                    │    │
│  │  - Browser Agent：网页浏览与信息提取                 │    │
│  │  - Code Agent：代码编写与执行                        │    │
│  │  - Research Agent：深度研究与信息整合                │    │
│  │  - Quality Agent：结果验证与质量检查                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                         ↓                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Sandbox 隔离环境                        │    │
│  │  - 每个 Task 分配独立 Docker 容器                    │    │
│  │  - Ubuntu 环境 + Chrome + API 服务                   │    │
│  │  - VNC 实时查看 + Websockify 转换                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                         ↓                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              工具调用层                              │    │
│  │  - Terminal：Shell 命令执行                          │    │
│  │  - Browser：网页浏览与操作                           │    │
│  │  - File：文件读写与管理                              │    │
│  │  - Web Search：网络搜索                              │    │
│  │  - MCP：外部工具集成                                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                         ↓                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              结果汇总与交付                          │    │
│  │  - SSE 实时回传执行进度                              │    │
│  │  - 整合多 Agent 执行结果                             │    │
│  │  - 生成最终交付物（报告/代码/数据集）                │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## LangGraph 编排

```
Manus Agent Graph Flow（LangGraph 编排）：

      flowchart TD
    Start([🚀 Start]) --> Planner

    subgraph Core["🧠 Agent Core"]
        Planner[📋 Planner Node]
        Router{🔀 Router}
    end

    Planner --> Router

    subgraph Executors["⚡ Executor Nodes"]
        Bash[🖥️ Bash Executor]
        Search[🔍 Search]
        Playwright[🌐 Playwright]
        DeepResearch[🔬 Deep Research]
    end

    Router -->|bash| Bash
    Router -->|search| Search
    Router -->|playwright| Playwright
    Router -->|deep_research| DeepResearch
    Router -->|complete| Consolidator[📦 Consolidator]
    Router -->|end| End([✅ End])

    Bash --> Planner
    Search --> Planner
    Playwright --> Planner
    DeepResearch --> Planner
    Consolidator --> Planner
```

## 关键特性

| 特性 | 描述 | SOTA价值 |
|-----|------|---------|
| **Multi-Agent Architecture** | 多个专业Agent协作 | 🔴 最高 |
| **PlanAct Agent** | 规划与执行一体化 | 🔴 最高 |
| **Sandbox Isolation** | 每个Task独立Docker容器 | 🔴 最高 |
| **LangGraph Orchestration** | 模块化图架构 | 🔴 最高 |
| **SSE Real-time** | Server-Sent Events实时反馈 | 🟡 高 |
| **MCP Integration** | 外部MCP工具集成 | 🟡 高 |
| **Deep Research Mode** | 多源综合分析 | 🟡 高 |
| **Real-time Viewing** | VNC + NoVNC实时查看 | 🟡 高 |

## 与其他框架对比

| 维度 | Manus | Anthropic Claude | OpenAI Agents | 阿里 Qoder |
|-----|-------|-----------------|---------------|-----------|
| **自主性级别** | 完全自主（用户可离开） | 半自主（需监控） | 半自主 | 半自主 |
| **执行时长** | 15-80 分钟 | 数分钟 | 数分钟 | 数分钟 |
| **Sandbox** | 每个Task独立容器 | Session级隔离 | Computer Use Layer | VSCode原生 |
| **交付物** | 完整制品（报告/网站/代码） | 对话结果 | 对话结果 | 代码文件 |
| **实时查看** | VNC + NoVNC | 无 | 无 | IDE内置 |
| **LangGraph** | ✅ 原生支持 | ❌ | ❌ | ❌ |

## 参考价值

- **完全自主范式**：技术前沿，值得研究
- **PlanAct模式**：规划执行一体化
- **Sandbox架构**：每个Task独立容器
- **LangGraph编排**：模块化图架构最佳实践
- **SSE实时反馈**：长任务进度可视化

## 注意事项

- **技术前沿 ≠ 工业最佳实践**：完全自主目前不是工业共识
- **成本控制**：长任务成本不可控
- **错误恢复**：长链幻觉风险
- **企业落地**：需要Human-in-the-loop机制

## 官方资源

- Manus Official: https://manus.ai
- Manus GitHub: https://github.com/manus-ai/manus