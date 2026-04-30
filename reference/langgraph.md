---
project: LangGraph
last_updated: 2026-04-30
refresh_policy: "超过 3 个月必须用 WebSearch 重新验证并刷新本文件"
verified_sources:
  - https://github.com/langchain-ai/langgraph
  - https://langchain-ai.github.io/langgraph/
---

# LangGraph（状态编排）架构

> **为什么选择 LangGraph 作为 SOTA 对标？**
> - StateGraph 是状态编排的最佳实践
> - Durable Execution 是生产级 Agent 的必备能力
> - 25k stars，被 Klarna、Replit、Elastic 采用，经过大规模验证

---

## 一、核心数据

- **GitHub Stars**: 25,000+
- **月下载量**: 34.5M
- **开发团队**: LangChain
- **行业地位**: 2026年行业标配
- **企业客户**: Klarna、Replit、Elastic

---

## 二、核心架构

```
LangGraph 核心架构：

┌─────────────────────────────────────────────────────────────┐
│                    StateGraph（状态图）                       │
│    - Nodes（agents/tools）                                   │
│    - Edges（control flow）                                   │
│    - Persistent state                                        │
│    - Conditional edges                                       │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Durable Execution                         │
│    - Persist through failures                                │
│    - Run for extended periods                                │
│    - 自动错误恢复                                             │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Multi-Agent Patterns                      │
│    - Supervisor（监督者模式）                                 │
│    - Hierarchical（层级模式）                                 │
│    - Sequential（顺序模式）                                   │
│    - Parallel（并行模式）                                     │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    LangSmith Integration                     │
│    - Monitoring agent performance                            │
│    - Tracing and debugging                                   │
│    - Langfuse tracing                                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 三、StateGraph 深度解析

### 是什么？

**StateGraph** 是 LangGraph 的核心概念，把 Agent 的执行流程建模为"状态图"（有向图）。

### 为什么这样设计？

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
4. 无法并行：只能顺序执行
5. 无法回溯：不知道 Agent 在哪个阶段失败
```

#### StateGraph 的解决方案

```
StateGraph 的设计原理：

1. 节点（Nodes）
   - 每个节点是一个步骤（agent/tool）
   - 节点接收状态，返回状态更新
   - 节点可以指定下一个目的地

2. 边（Edges）
   - 边定义节点之间的连接
   - 条件边（Conditional Edges）根据状态动态选择路径
   - 支持循环和分支

3. 状态（State）
   - 状态在节点之间传递
   - 状态是持久化的（Checkpointing）
   - 状态可以回溯（Time Travel）

4. Checkpointing（存档）
   - 每个节点执行后自动存档
   - 崩溃后可以从存档点恢复
   - 不需要从头开始

关键洞察：
- StateGraph 把 Agent 执行建模为"地铁线路图"
- 每个站点是一个步骤，每条线路是一个分支
- 你可以清楚看到所有可能的路径
- 崩溃后可以从存档点继续
```

### 具体怎么做的？

```python
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict

# 定义状态
class AgentState(TypedDict):
    messages: list
    current_step: str
    tool_outputs: list
    reflection_round: int

# 定义节点（每个节点是一个步骤）
def intake_node(state: AgentState) -> AgentState:
    # 处理 intake 逻辑
    return {"current_step": "intake_done"}

def planning_node(state: AgentState) -> AgentState:
    # 处理 planning 逻辑
    return {"current_step": "planning_done"}

def reflection_node(state: AgentState) -> AgentState:
    # 处理 reflection 逻辑
    return {"reflection_round": state["reflection_round"] + 1}

def final_node(state: AgentState) -> AgentState:
    # 处理 final 逻辑
    return {"current_step": "end"}

# 定义条件边（根据状态动态选择路径）
def should_continue(state: AgentState) -> str:
    if state["reflection_round"] >= 3:
        return "end"  # 进入终稿
    else:
        return "continue"  # 继续反思

# 创建 StateGraph
graph = StateGraph(AgentState)

# 添加节点
graph.add_node("intake", intake_node)
graph.add_node("planning", planning_node)
graph.add_node("reflection", reflection_node)
graph.add_node("final", final_node)

# 添加边（定义执行路径）
graph.add_edge("intake", "planning")
graph.add_edge("planning", "reflection")

# 添加条件边（动态选择路径）
graph.add_conditional_edges(
    "reflection",
    should_continue,
    {
        "continue": "reflection",  # 继续反思
        "end": "final"             # 进入终稿
    }
)

graph.add_edge("final", END)

# 关键：Checkpointing（存档）
checkpointer = MemorySaver()
app = graph.compile(checkpointer=checkpointer)

# 执行（自动存档）
result = app.invoke(
    {"messages": ["用户输入"], "reflection_round": 0},
    config={"configurable": {"thread_id": "session-1"}}
)

# 崩溃后恢复（从存档点继续）
# 不需要从头开始！
result = app.invoke(
    {"messages": ["继续执行"]},
    config={"configurable": {"thread_id": "session-1"}}
)
```

### 这样设计的好处

| 好处 | 具体价值 |
|:---|:---|
| **可视化路径** | 像地铁线路图，你可以清楚看到所有可能的路径。合规官问"Agent 能跳过欺诈检查吗？"——你指向图，明确回答"不能"。 |
| **可中断恢复** | Checkpointing 让 Agent 可以从存档点继续，不用从头开始。崩溃后自动恢复，不浪费 tokens。 |
| **可审计** | 每个节点的输入输出都被记录，可以追踪 Agent 的决策过程。合规要求"记录每一步"，StateGraph 自动满足。 |
| **条件分支** | Conditional edges 让 Agent 可以根据状态动态选择路径。不是硬编码的 if-else，而是可视化的分支。 |
| **可回溯** | Time Travel 让你可以回到任意节点，查看当时的状态。调试时"看看 Agent 在第 3 步做了什么"，一键回溯。 |

### 这样设计的坏处

| 坯处 | 具体风险 |
|:---|:---|
| **架构复杂度** | 需要学习 StateGraph 的概念（节点、边、状态），比 while 循环复杂。 |
| **代码量增加** | 简单的 while 循环 20 行，StateGraph 可能需要 50+ 行。 |
| **调试困难** | 需要理解状态流转，不是简单的顺序执行。 |
| **性能开销** | Checkpointing 需要额外的存储和 I/O，增加了延迟。 |

### 类比解释

| 类比 | 说明 |
|:---|:---|
| **while 循环** | 像走迷宫没有地图：你只能一步步走，不知道下一步去哪，走错了只能从头开始。 |
| **StateGraph** | 像地铁线路图：每个站点是一个步骤，每条线路是一个分支，你可以清楚看到所有可能的路径。 |
| **Checkpointing** | 像游戏存档：每过一关就存档，失败了可以从存档点继续，不用从头开始。 |
| **Conditional Edges** | 像地铁换乘：你可以根据目的地选择不同的线路，不是固定的路线。 |
| **Time Travel** | 像时光机：你可以回到任意节点，查看当时的状态，调试时非常有用。 |

---

## 四、Durable Execution 深度解析

### 是什么？

**Durable Execution** 是 LangGraph 的持久化执行机制，让 Agent 可以在崩溃后自动恢复。

### 为什么这样设计？

```
核心问题：Agent 执行可能崩溃，需要从头开始

崩溃场景：
1. LLM 调用超时
2. 工具执行失败
3. 网络断开
4. 服务重启
5. 资源耗尽

问题：
- 崩溃后必须从头开始
- 重新消耗所有 tokens
- 重新执行所有步骤
- 浪费时间和成本

LangGraph 的解决方案：

1. Checkpointing（存档）
   - 每个节点执行后自动存档
   - 存档包含完整状态（输入、输出、决策）
   - 存档是原子操作，不会损坏

2. 自动恢复
   - 崩溃后自动检测存档
   - 从存档点继续执行
   - 不需要从头开始

3. 幂等性设计
   - 每个节点可以安全重试
   - 重试不会产生副作用
   - 保证结果一致

关键洞察：
- Durable Execution 让 Agent 可以在崩溃后自动恢复
- 不浪费 tokens，不浪费时间
- 生产级 Agent 必备能力
```

### 具体怎么做的？

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.checkpoint.sqlite import SqliteSaver

# 内存存档（开发调试）
checkpointer = MemorySaver()

# SQLite 存档（生产环境）
checkpointer = SqliteSaver("checkpoints.db")

# 编译时添加存档器
app = graph.compile(checkpointer=checkpointer)

# 执行时指定 thread_id（会话标识）
result = app.invoke(
    {"messages": ["用户输入"]},
    config={"configurable": {"thread_id": "session-1"}}
)

# 崩溃后恢复（使用相同的 thread_id）
# LangGraph 自动检测存档，从存档点继续
result = app.invoke(
    {"messages": ["继续执行"]},
    config={"configurable": {"thread_id": "session-1"}}
)

# 查看存档历史
history = app.get_state_history({"thread_id": "session-1"})
for state in history:
    print(f"节点: {state.values['current_step']}")
    print(f"时间: {state.created_at}")
```

### 这样设计的好处

| 好处 | 具体价值 |
|:---|:---|
| **自动恢复** | 崩溃后自动从存档点继续，不需要从头开始。 |
| **节省成本** | 不浪费 tokens，不重复执行已完成的步骤。 |
| **可审计** | 存档历史可以追踪 Agent 的每一步决策。 |
| **可回溯** | Time Travel 让你可以回到任意节点查看状态。 |
| **生产级** | 生产环境必备能力，LangGraph 自动满足。 |

---

## 五、Multi-Agent Patterns 深度解析

### Supervisor Pattern（监督者模式）

```
架构：

┌───────────────┐
│  Supervisor   │
│  (Orchestrator)│
└───────┬───────┘
        │
   ┌────┼────┐
   ↓    ↓    ↓
┌────┐┌────┐┌────┐
│ A1 ││ A2 ││ A3 │
└────┘└────┘└────┘

工作流程：
1. Supervisor 接收任务
2. Supervisor 分解任务，分配给子 Agent
3. 子 Agent 执行任务，返回结果
4. Supervisor 聚合结果，返回给用户

适用场景：
- 任务可以分解为多个子任务
- 子任务之间有依赖关系
- 需要统一协调
```

### Hierarchical Pattern（层级模式）

```
架构：

┌───────────────┐
│  Supervisor   │
└───────┬───────┘
        │
   ┌────┼────┐
   ↓    ↓    ↓
┌────┐┌────┐┌────┐
│Sub1││Sub2││Sub3│
│Supv││Supv││Supv│
└─┬──┘└─┬──┘└─┬──┘
  │     │     │
 ┌┼┐   ┌┼┐   ┌┼┐
 │A│   │A│   │A│
 └─┘   └─┘   └─┘

工作流程：
1. Supervisor 接收任务
2. Supervisor 分配给子 Supervisor
3. 子 Supervisor 进一步分解，分配给子 Agent
4. 子 Agent 执行，返回给子 Supervisor
5. 子 Supervisor 聚合，返回给 Supervisor
6. Supervisor 最终聚合

适用场景：
- 任务层级复杂
- 需要多级协调
- 大型企业场景
```

### Parallel Pattern（并行模式）

```python
from langgraph.graph import StateGraph

# 并行执行多个 Agent
graph = StateGraph(AgentState)

# 添加并行节点
graph.add_node("agent_1", agent_1_node)
graph.add_node("agent_2", agent_2_node)
graph.add_node("agent_3", agent_3_node)
graph.add_node("aggregator", aggregator_node)

# 从同一个节点出发，并行执行
graph.add_edge("start", "agent_1")
graph.add_edge("start", "agent_2")
graph.add_edge("start", "agent_3")

# 聚合结果
graph.add_edge("agent_1", "aggregator")
graph.add_edge("agent_2", "aggregator")
graph.add_edge("agent_3", "aggregator")

# 执行时自动并行
app = graph.compile()
result = app.invoke({"input": "xxx"})
```

---

## 六、关键特性总结

| 特性 | 描述 | SOTA价值 |
|-----|------|---------|
| **StateGraph** | 状态图编排 | 🔴 最高 |
| **Durable Execution** | 持久化执行，错误恢复 | 🔴 最高 |
| **Multi-Agent Patterns** | 多种编排模式 | 🔴 最高 |
| **LangSmith Integration** | 监控和调试 | 🟡 高 |
| **Conditional Edges** | 条件边，灵活控制流 | 🟡 高 |
| **Checkpointing** | 存档，崩溃恢复 | 🔴 最高 |
| **Time Travel** | 回溯，调试 | 🟡 高 |

---

## 七、适用场景

| 适用 | 不适用 |
|:---|:---|
| 生产级 Agent（需要崩溃恢复） | 简单原型（不需要持久化） |
| 多 Agent 协作 | 单一 Agent |
| 需要审计追踪 | 不需要合规记录 |
| 需要可视化路径 | 简单顺序执行 |
| 需要条件分支 | 固定执行路径 |

---

## 八、官方资源

- LangGraph GitHub: https://github.com/langchain-ai/langgraph
- LangGraph Docs: https://langchain-ai.github.io/langgraph
- LangSmith: https://www.langchain.com/langsmith
- LangGraph Examples: https://github.com/langchain-ai/langgraph/tree/main/examples