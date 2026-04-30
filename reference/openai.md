---
project: OpenAI Agents SDK
last_updated: 2026-04-30
refresh_policy: "超过 3 个月必须用 WebSearch 重新验证并刷新本文件"
verified_sources:
  - https://openai.github.io/openai-agents-python/
  - https://platform.openai.com/docs/guides/function-calling
---

# OpenAI Agents SDK 架构

> **为什么选择 OpenAI 作为 SOTA 对标？**
> - Function Calling 是实践标准，OpenAI 是标准制定者
> - Guardrails 是安全护栏的最佳实践
> - GPT-5 + o3 是模型原生 Agent 的最佳实践

---

## 一、核心架构

```
OpenAI Agent 架构全景：

┌─────────────────────────────────────────────────────────────┐
│                    Agent Loop（代理循环）                     │
│    - 规划 → 执行 → 观察 → 调整                               │
│    - 持续迭代                                                │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Native Sandbox（原生沙箱）                 │
│    - Computer Use Layer                                      │
│    - 安全执行环境                                            │
│    - 资源隔离                                                │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Handoffs（任务移交）                       │
│    - Agent 间任务传递                                        │
│    - 专业分工                                                │
│    - 上下文传递                                              │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Guardrails（护栏）                        │
│    - 输入验证                                                │
│    - 输出过滤                                                │
│    - 安全约束                                                │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Model-Native Harness（模型原生控制）       │
│    - GPT-5 + o3 模型                                         │
│    - 原生推理能力                                            │
│    - 工具调用优化                                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 二、Function Calling 深度解析

### 是什么？

**Function Calling** 是 OpenAI 在 2023 年推出的结构化工具调用机制，让模型输出结构化的 JSON，而不是文本。

### 为什么这样设计？

#### OpenAI 的痛点

```
OpenAI 发现的问题：

1. 正则解析不可靠
   - 模型输出文本，需要用正则解析
   - 模型可能写错格式（比如把"Thought"写成"思考"）
   - 正则解析容易失败

2. 参数提取困难
   - 从自然语言提取参数（如 file_id）
   - 参数可能缺失、错误、格式不对
   - 没有校验机制

3. 工具调用不可控
   - 模型可能调用错误的工具
   - 模型可能传递错误的参数
   - 没有安全护栏
```

#### OpenAI 的解决方案

```
Function Calling 的设计原理：

1. 结构化输出
   - 模型输出 JSON 对象，不是文本
   - JSON 对象包含 tool_name 和 parameters
   - 从根本上消除正则解析的需求

2. JSON Schema 校验
   - 定义工具的参数 schema
   - 模型输出"被迫"符合 schema
   - 参数自动校验，不会缺失或错误

3. strict: true 模式
   - OpenAI 的 Structured Outputs 功能（2024 年 6 月）
   - 强制模型输出符合 schema
   - 即使模型"想"输出错误格式，也会被拒绝
```

### 具体怎么做的？

```python
# Function Calling 架构：单次请求内定义工具

tools = [
    {
        "type": "function",
        "function": {
            "name": "read_wps_content",
            "description": "读取 WPS 云文档内容",
            "parameters": {  # ← JSON Schema 强制校验
                "type": "object",
                "properties": {
                    "file_id": {
                        "type": "string",
                        "description": "WPS 文档 ID"
                    }
                },
                "required": ["file_id"]
            },
            "strict": True  # ← 强制模式，输出必须符合 schema
        }
    }
]

response = client.chat.completions.create(
    model="gpt-5",
    messages=messages,
    tools=tools,  # ← 工具定义在请求中
)

# 模型返回结构化的 tool_call，不是文本
tool_call = response.choices[0].message.tool_calls[0]
# tool_call.function.name = "read_wps_content"
# tool_call.function.arguments = '{"file_id": "xxx"}'  ← 已是 JSON，无需正则解析

# 执行工具
import json
args = json.loads(tool_call.function.arguments)
result = read_wps_content(args["file_id"])
```

### 这样设计的好处

| 好处 | 具体价值 |
|:---|:---|
| **架构简单** | 无需单独部署工具服务器，工具定义在请求中。 |
| **可靠性高** | strict: true 模式保证输出符合 schema，从根本上消除解析失败。 |
| **广泛支持** | 所有主流 LLM 提供商（阿里百炼、Google Gemini、Azure OpenAI）都支持 Function Calling。 |
| **性能好** | 无额外网络请求，工具调用在单次请求内完成。 |
| **易于调试** | 工具定义和调用都在同一请求中，调试简单。 |

### 这样设计的坏处

| 坏处 | 具体风险 |
|:---|:---|
| **工具不可复用** | 工具定义在请求中，无法跨 Agent 共享。 |
| **无法动态发现** | 工具必须硬编码在请求中，Agent 无法在运行时发现新工具。 |
| **跨模型不兼容** | 不同模型的 Function Calling API 有细微差异，迁移需要修改代码。 |
| **请求体积大** | 每次请求都要携带工具定义，增加了请求体积。 |

### 适用场景

| 适用 | 不适用 |
|:---|:---|
| 单一 Agent、单一模型 | 需要跨 Agent 共享工具 |
| 工具数量固定、不需要动态扩展 | 需要动态发现工具 |
| 只使用一种模型 | 需要跨模型兼容 |
| 追求架构简单 | 需要工具生态 |

---

## 三、Guardrails（安全护栏）深度解析

### 是什么？

**Guardrails** 是 OpenAI 的安全护栏机制，在 Agent 执行前后进行验证和过滤。

### 为什么这样设计？

```
核心问题：Agent 有能力执行危险操作，需要安全护栏

OpenAI 的解决方案：

1. 输入护栏（Input Guardrails）
   - 在 Agent 接收用户输入前验证
   - 检查是否包含敏感信息、恶意指令
   - 过滤不合法的输入

2. 输出护栏（Output Guardrails）
   - 在 Agent 输出结果前验证
   - 检查是否包含敏感信息、错误格式
   - 过滤不合法的输出

3. 工具护栏（Tool Guardrails）
   - 在工具执行前后验证
   - 检查参数是否合法
   - 检查结果是否安全

关键洞察：
- 护栏在 Agent 执行前后进行验证
- 防止 Agent 接收或输出危险内容
- 快速失败，不让 Agent 继续执行
```

### 具体怎么做的？

```python
from agents import Agent, input_guardrail, output_guardrail, tool_guardrail

# 输入护栏：检查用户输入
@input_guardrail
def check_sensitive_input(ctx, agent, input):
    if contains_sensitive_data(input):
        return GuardrailResult(
            tripwire_triggered=True,  # ← 触发护栏，终止执行
            message="输入包含敏感信息，已拦截"
        )
    return GuardrailResult(tripwire_triggered=False)

# 输出护栏：检查 Agent 输出
@output_guardrail
def check_sensitive_output(ctx, agent, output):
    if contains_sensitive_data(output):
        return GuardrailResult(
            tripwire_triggered=True,
            message="输出包含敏感信息，已拦截"
        )
    return GuardrailResult(tripwire_triggered=False)

# 工具护栏：检查工具参数
@tool_guardrail
def check_tool_params(ctx, agent, tool_call):
    if tool_call.function.name == "delete_file":
        # 删除文件需要用户确认
        return GuardrailResult(
            tripwire_triggered=True,
            message="删除文件需要用户确认"
        )
    return GuardrailResult(tripwire_triggered=False)

# 创建 Agent，添加护栏
agent = Agent(
    name="SafeAgent",
    input_guardrails=[check_sensitive_input],
    output_guardrails=[check_sensitive_output],
    tool_guardrails=[check_tool_params],
)
```

### 这样设计的好处

| 好处 | 具体价值 |
|:---|:---|
| **多层防护** | 输入、输出、工具三层护栏，全方位保护。 |
| **快速失败** | 发现危险操作立即终止，防止损害。 |
| **可扩展** | 可以随时添加新的护栏规则。 |
| **并行执行** | 护栏与 Agent 执行并行，不影响性能。 |

### 类比解释

| 类比 | 说明 |
|:---|:---|
| **Guardrails** | 像护栏：AI 在高速公路上行驶，护栏防止它冲出道路，即使它想犯错也被限制在安全范围内。 |
| **Input Guardrails** | 像安检：你进入机场前，安检检查你的行李，发现危险物品就拦截。 |
| **Output Guardrails** | 像出口检查：你离开机场前，检查你的行李，发现违禁物品就拦截。 |
| **Tool Guardrails** | 像操作确认：你按"删除"按钮前，系统弹出确认框，让你再次确认。 |

---

## 四、Handoffs（任务移交）深度解析

### 是什么？

**Handoffs** 是 OpenAI 的 Agent 间任务传递机制，一个 Agent 可以把任务移交给另一个专业 Agent。

### 为什么这样设计？

```
核心问题：单一 Agent 无法处理所有类型的任务

OpenAI 的解决方案：

1. 专业分工
   - 每个 Agent 专注一个领域
   - 例如：BillingAgent（账单）、SupportAgent（支持）、CodingAgent（编程）

2. 任务移交
   - 当前 Agent 发现任务不属于自己
   - 把任务移交给专业 Agent
   - 传递上下文（用户信息、历史对话）

3. 上下文传递
   - 移交时传递必要的上下文
   - 接收 Agent 不需要重新收集信息
   - 提高效率，减少重复
```

### 具体怎么做的？

```python
from agents import Agent, handoff

# 定义专业 Agent
billing_agent = Agent(
    name="BillingAgent",
    instructions="你专门处理账单相关问题...",
)

support_agent = Agent(
    name="SupportAgent",
    instructions="你专门处理客户支持问题...",
)

# 定义主 Agent，可以移交任务
main_agent = Agent(
    name="MainAgent",
    instructions="你是主 Agent，可以移交任务给专业 Agent...",
    handoffs=[
        handoff(billing_agent),  # ← 可以移交给 BillingAgent
        handoff(support_agent),  # ← 可以移交给 SupportAgent
    ],
)

# 执行时，MainAgent 可以自动移交
# 例如：用户问"我的账单有问题"，MainAgent 会移交给 BillingAgent
```

### 这样设计的好处

| 好处 | 具体价值 |
|:---|:---|
| **专业分工** | 每个 Agent 专注一个领域，提高质量。 |
| **自动移交** | Agent 自动判断是否需要移交，无需人工干预。 |
| **上下文传递** | 移交时传递上下文，接收 Agent 不需要重新收集信息。 |
| **可扩展** | 可以随时添加新的专业 Agent。 |

---

## 五、Native Sandbox（原生沙箱）深度解析

### 是什么？

**Native Sandbox** 是 OpenAI 的 Computer Use Layer，让 Agent 在安全环境中执行代码和操作。

### 为什么这样设计？

```
核心问题：Agent 需要执行代码、读写文件，但可能造成损害

OpenAI 的解决方案：

1. 容器隔离
   - Agent 在容器中执行
   - 与宿主系统隔离
   - 即使 Agent 执行恶意代码，也不会影响宿主

2. 资源限制
   - 限制 CPU、内存、网络
   - 防止 Agent 消耗过多资源
   - 防止 Agent 执行无限循环

3. 网络隔离
   - 限制 Agent 的网络访问
   - 只允许访问特定 API
   - 防止 Agent 访问敏感服务

关键洞察：
- 沙箱隔离防止 Agent 影响宿主系统
- 资源限制防止 Agent 消耗过多资源
- 网络隔离防止 Agent 访问敏感服务
```

### 这样设计的好处

| 好处 | 具体价值 |
|:---|:---|
| **安全隔离** | Agent 在容器中执行，不会影响宿主系统。 |
| **资源控制** | 限制 CPU、内存、网络，防止资源耗尽。 |
| **网络隔离** | 限制网络访问，防止访问敏感服务。 |
| **原生集成** | 与 OpenAI API 原生集成，无需额外配置。 |

---

## 六、关键特性总结

| 特性 | 描述 | SOTA价值 |
|-----|------|---------|
| **Native Sandbox** | Computer Use Layer，原生沙箱 | 🔴 最高 |
| **Handoffs** | Agent间任务移交 | 🟡 高 |
| **Guardrails** | 安全护栏 | 🔴 最高 |
| **Model-Native** | GPT-5 + o3 原生支持 | 🔴 最高 |
| **Parallel Agents** | 并行代理执行 | 🟡 高 |
| **Persistent Memory** | 持久化记忆 | 🟡 高 |
| **Function Calling** | 结构化工具调用 | 🔴 最高 |

---

## 七、官方资源

- OpenAI Documentation: https://platform.openai.com/docs
- OpenAI Cookbook: https://cookbook.openai.com
- Agents SDK: https://github.com/openai/openai-agents-sdk
- Guardrails Guide: https://openai.github.io/openai-agents-python/guardrails/