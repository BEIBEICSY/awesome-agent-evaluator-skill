---
project: Anthropic Claude
last_updated: 2026-04-30
refresh_policy: "超过 3 个月必须用 WebSearch 重新验证并刷新本文件"
verified_sources:
  - https://www.anthropic.com/research/model-context-protocol
  - https://docs.anthropic.com/
---

# Anthropic Claude Agent 架构

> **为什么选择 Anthropic 作为 SOTA 对标？**
> - MCP 是开放标准，Anthropic 是标准制定者
> - Claude Code 是生产级 Agent，经过大规模验证
> - Constitutional AI 是安全护栏的最佳实践

---

## 一、核心架构

### 六层架构

```
Anthropic Agent 架构全景：

┌─────────────────────────────────────────────────────────────┐
│ 1. Session Layer（会话层）                                    │
│    - 会话管理                                                │
│    - 上下文持久化                                            │
│    - Append-only log                                         │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. Harness Layer（控制层）                                    │
│    - 执行控制                                                │
│    - 错误处理                                                │
│    - 重试机制                                                │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. Sandbox Layer（隔离层）                                    │
│    - 容器隔离                                                │
│    - 资源限制                                                │
│    - 安全边界                                                │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. Hooks Layer（钩子层）                                      │
│    - 生命周期钩子                                            │
│    - 事件监听                                                │
│    - 扩展点                                                  │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. SubAgents Layer（子代理层）                                │
│    - 专业子代理                                              │
│    - 任务委托                                                │
│    - 结果聚合                                                │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. Tools Layer（工具层）                                      │
│    - MCP 协议                                                │
│    - 工具调用                                                │
│    - 结果验证                                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 二、MCP（Model Context Protocol）深度解析

### 是什么？

**MCP** 是 Anthropic 在 2024 年 11 月发布的开放标准，让 AI 连接外部工具和数据源。

### 为什么这样设计？

#### Anthropic 的痛点

```
Anthropic 发现的问题：

1. 工具不可复用
   - Claude Desktop 用户想连接 Google Drive、Slack、Notion
   - 每个工具都需要单独开发，无法复用
   - 用户换模型（从 Claude 到 GPT），工具就废了

2. 工具定义在请求中
   - Function Calling 把工具定义放在 API 请求里
   - 每次请求都要重复定义工具
   - 无法动态发现新工具

3. 跨模型不兼容
   - 不同模型的 Function Calling API 有差异
   - 迁移需要修改代码
```

#### Anthropic 的解决方案

```
MCP 的设计原理：

1. Server-Client 分离
   - 工具逻辑与 Agent 逻辑解耦
   - 工具变成"服务"，Agent 变成"客户端"
   - 同一工具可被多个 Agent 使用

2. 运行时发现工具
   - 工具不是硬编码在 prompt 中
   - 通过 handshake 动态发现
   - Agent 可以"即插即用"新工具

3. JSON Schema 强制校验
   - 模型输出"被迫"符合 schema
   - 从根本上消除格式解析失败的风险
```

### 具体怎么做的？

```typescript
// MCP 架构：Server-Client 分离

// 1. MCP Server：暴露工具/资源
const server = new MCPServer({
  name: "wps-server",
  tools: [
    {
      name: "read_wps_content",
      description: "读取 WPS 云文档内容",
      inputSchema: {  // ← JSON Schema 强制校验
        type: "object",
        properties: {
          file_id: { 
            type: "string",
            description: "WPS 文档 ID"
          }
        },
        required: ["file_id"]
      }
    }
  ]
});

// 2. MCP Client：连接 Server，发现工具
const client = new MCPClient();
await client.connect("wps-server");

// 运行时发现工具（不是硬编码！）
const tools = await client.listTools();
// 返回：[{ name: "read_wps_content", inputSchema: {...} }]

// 3. 调用工具
const result = await client.callTool("read_wps_content", { 
  file_id: "xxx" 
});
// 返回：结构化结果，不是文本
```

### 这样设计的好处

| 好处 | 具体价值 |
|:---|:---|
| **工具可复用** | 一个 MCP Server 可以被 Claude Desktop、Cursor、自定义 Agent 同时使用。 |
| **跨模型兼容** | MCP 是开放标准，任何支持 MCP 的模型都可以使用同一工具。 |
| **动态扩展** | Agent 可以在运行时发现新工具，无需修改代码。 |
| **可靠性高** | JSON Schema 强制校验，从根本上消除格式解析失败。 |
| **生态丰富** | 50+ 官方 MCP Server（PostgreSQL、GitHub、Slack、Notion 等）。 |

### 这样设计的坏处

| 坏处 | 具体风险 |
|:---|:---|
| **架构复杂度** | 需要单独部署 MCP Server，增加了系统复杂度。 |
| **性能开销** | 工具发现和调用需要额外的网络请求，增加了延迟。 |
| **学习曲线** | MCP 是新协议，开发者需要学习新的概念和 API。 |
| **调试困难** | Server-Client 分离，调试需要追踪多个组件。 |

### 适用场景

| 适用 | 不适用 |
|:---|:---|
| 需要跨 Agent 共享工具 | 单一 Agent、单一模型 |
| 需要动态发现工具 | 工具数量固定、不需要动态扩展 |
| 需要跨模型兼容 | 只使用一种模型 |
| 需要工具生态 | 工具数量少、不需要社区贡献 |

---

## 三、StateGraph vs while 循环

### Anthropic 为什么选择 StateGraph？

#### while 循环的问题

```python
# while 循环：无法中断恢复
while phase != "end":
    if phase == "intake": ...
    elif phase == "planning": ...
    elif phase == "reflection": ...
    elif phase == "final": ...

问题：
1. 无法中断恢复：进程崩溃后，必须从头开始
2. 无法条件分支：只能用 if-else，无法可视化路径
3. 无法审计：不知道 Agent 走了哪些路径
```

#### StateGraph 的解决方案

```python
# StateGraph：可视化 + 可中断 + 可审计
from langgraph.graph import StateGraph, END

graph = StateGraph(AgentState)

# 定义节点（每个节点是一个步骤）
graph.add_node("intake", intake_node)
graph.add_node("planning", planning_node)
graph.add_node("reflection", reflection_node)
graph.add_node("final", final_node)

# 定义边（执行路径）
graph.add_edge("intake", "planning")
graph.add_conditional_edges("reflection", should_continue, {
    "continue": "reflection",  # 继续反思
    "end": "final"             # 进入终稿
})

# 关键：Checkpointing（存档）
graph.set_checkpointer(MemorySaver())

# 执行
app = graph.compile()
result = app.invoke({"input": "xxx"})  # ← 自动存档，崩溃后可恢复
```

### 这样设计的好处

| 好处 | 具体价值 |
|:---|:---|
| **可视化路径** | 像地铁线路图，你可以清楚看到所有可能的路径。 |
| **可中断恢复** | Checkpointing 让 Agent 可以从存档点继续，不用从头开始。 |
| **可审计** | 每个节点的输入输出都被记录，可以追踪 Agent 的决策过程。 |
| **条件分支** | Conditional edges 让 Agent 可以根据状态动态选择路径。 |

### 类比解释

| 类比 | 说明 |
|:---|:---|
| **while 循环** | 像走迷宫没有地图：你只能一步步走，不知道下一步去哪，走错了只能从头开始。 |
| **StateGraph** | 像地铁线路图：每个站点是一个步骤，每条线路是一个分支，你可以清楚看到所有可能的路径。 |
| **Checkpointing** | 像游戏存档：每过一关就存档，失败了可以从存档点继续，不用从头开始。 |

---

## 四、Constitutional AI（安全护栏）

### 是什么？

**Constitutional AI** 是 Anthropic 的安全护栏机制，让 AI 遵循一套"宪法"（规则），防止它执行危险操作。

### 为什么这样设计？

```
核心问题：Agent 有能力执行危险操作，需要安全护栏

Anthropic 的解决方案：

1. 定义"宪法"（Constitution）
   - 一套规则，定义 AI 应该做什么、不应该做什么
   - 例如："不要泄露用户隐私"、"不要执行恶意代码"

2. 自动检查（Self-critique）
   - AI 在执行前，先自我检查是否符合宪法
   - 如果不符合，自动修正

3. 快速失败（Fail-fast）
   - 发现危险操作立即终止
   - 不让 Agent 继续执行
```

### 具体怎么做的？

```python
# Constitutional AI 示例

constitution = [
    "不要泄露用户隐私信息",
    "不要执行可能损害系统的代码",
    "不要发送未经用户确认的邮件",
]

def check_action(action, constitution):
    for rule in constitution:
        if violates_rule(action, rule):
            return False, rule
    return True, None

# Agent 执行前检查
action = "delete_file('/etc/passwd')"
is_safe, violated_rule = check_action(action, constitution)

if not is_safe:
    # 快速失败
    raise SafetyError(f"违反宪法规则：{violated_rule}")
```

### 这样设计的好处

| 好处 | 具体价值 |
|:---|:---|
| **自动检查** | AI 自我检查，不需要人工干预。 |
| **快速失败** | 发现危险操作立即终止，防止损害。 |
| **可扩展** | 可以随时添加新规则到宪法。 |
| **透明** | 规则明确，用户知道 AI 的边界。 |

---

## 五、Agent Teams 模式

### 是什么？

**Agent Teams** 是 Anthropic 的多 Agent 协作模式，一个 Orchestrator（编排者）协调多个专业子代理。

### 为什么这样设计？

```
核心问题：单一 Agent 无法处理复杂任务

Anthropic 的解决方案：

1. Orchestrator（编排者）
   - 任务分解：把大任务拆成小任务
   - 子代理调度：把小任务分配给专业子代理
   - 结果聚合：把子代理的结果合并

2. SubAgents（子代理）
   - 专业分工：每个子代理专注一个领域
   - 例如：Researcher（研究）、Coder（编程）、Reviewer（审核）

3. 任务委托
   - Orchestrator 把任务委托给子代理
   - 子代理完成后，把结果返回给 Orchestrator
```

### 具体怎么做的？

```
Agent Teams 架构：

┌─────────────────────────────────────────────────────────────┐
│                    Orchestrator（编排者）                     │
│    - 任务分解                                                │
│    - 子代理调度                                              │
│    - 结果聚合                                                │
└─────────────────────────────────────────────────────────────┘
                         ↓
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│ SubAgent 1    │ │ SubAgent 2    │ │ SubAgent N    │
│ (Researcher)  │ │ (Coder)       │ │ (Reviewer)    │
└───────────────┘ └───────────────┘ └───────────────┘
```

### 这样设计的好处

| 好处 | 具体价值 |
|:---|:---|
| **专业分工** | 每个子代理专注一个领域，提高质量。 |
| **并行执行** | 多个子代理可以同时工作，提高效率。 |
| **可扩展** | 可以随时添加新的子代理。 |
| **容错** | 一个子代理失败，不影响其他子代理。 |

---

## 六、关键特性总结

| 特性 | 描述 | SOTA价值 |
|-----|------|---------|
| **Session Management** | Append-only log，持久化会话 | 🔴 最高 |
| **Sandbox Isolation** | 容器级隔离，安全执行 | 🔴 最高 |
| **Hooks System** | 生命周期钩子，可扩展 | 🟡 高 |
| **SubAgents** | 专业子代理，任务委托 | 🔴 最高 |
| **MCP Protocol** | Model Context Protocol，工具集成 | 🔴 最高 |
| **Guardrails** | 安全护栏，Constitutional AI | 🔴 最高 |
| **StateGraph** | 状态图编排，可中断恢复 | 🔴 最高 |

---

## 七、官方资源

- Anthropic Documentation: https://docs.anthropic.com
- Claude Agent SDK: https://github.com/anthropics/anthropic-sdk-python
- MCP Protocol: https://modelcontextprotocol.io
- MCP Official Servers: https://github.com/anthropics/mcp-servers