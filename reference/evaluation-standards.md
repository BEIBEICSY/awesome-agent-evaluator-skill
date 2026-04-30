# 评估标准详解（七大里程碑）

> **核心洞察**：这七个里程碑不是随意选择的，而是基于 Agent 的本质定义：**Agent = LLM + System + Environment**。

---

## 一、感知层（Perception Layer）

### 通过标准
**多模态反馈闭环与环境观察流**

### 为什么这是标准？

#### 核心洞察
Agent 的行动能力受限于它的感知能力。一个只能"读文本"的 Agent，就像一个只能听电话的盲人——它无法理解 UI 截图、无法识别图表、无法感知用户语气。

#### 设计原理
```
感知闭环：感知 → 解释 → 决策 → 行动 → 再次感知

关键点：
1. 感知：从环境获取信息（文本、图像、音频、日志）
2. 解释：将原始信息转化为结构化理解
3. 决策：基于理解做出判断
4. 行动：执行具体操作
5. 再次感知：验证行动是否有效（闭环！）
```

#### 类比解释
| 类比 | 说明 |
|:---|:---|
| **盲人打电话** | 只能听声音，无法看到对方的表情、手势、环境。这就是"只能读文本"的 Agent。 |
| **司机开车** | 需要同时看路况、听导航、感知车身震动。这就是"多模态感知"的 Agent。 |
| **闭环验证** | 你按了按钮，需要看到灯亮了才能确认操作成功。这就是"反馈闭环"。 |

#### SOTA 参考
| 公司 | 实现 | 来源 |
|:---|:---|:---|
| **Anthropic** | Claude Code 可以"看"终端输出、"听"用户指令、"感知"文件变化 | [Claude Code Architecture](https://claudecode.jp/en/news/engineer/eight-trends-defining-how-software-gets-built-in-2026) |
| **Manus** | 完全自主执行，持续观察环境变化 | [Manus Architecture](https://aidevstart.com/blog/ai-agents-infrastructure-2026) |

---

## 二、大脑层（Reasoning Layer）

### 通过标准
**Plan-and-Execute 或自反思循环**

### 为什么这是标准？

#### 核心洞察
ReAct（2022年）是基础版，但 2026 年的 SOTA 已经进化到 **Plan-and-Execute**（先规划再执行）或 **自反思循环**（每一步都自检）。

#### 设计原理
```
范式演进：

2022年：ReAct（Thought → Action → Observation）
       问题：单次推理容易出错，没有规划

2024年：Reflection（Thought → Action → Observation → Reflection）
       改进：增加了自检环节

2026年：Plan-and-Execute（Plan → Execute → Reflect → Adjust）
       改进：先规划整体策略，再逐步执行，每步自检

关键洞察：
- LLM 的推理是概率性的，单次推理容易出错
- 通过"规划 → 执行 → 反思 → 调整"的循环，可以显著提高可靠性
- 规划让 Agent 有"全局视野"，不是盲目行动
```

#### 类比解释
| 类比 | 说明 |
|:---|:---|
| **ReAct** | 像新手做饭：看到食材就动手，没有计划，经常做错。 |
| **Reflection** | 像有经验的厨师：每一步都尝味道，发现不对就调整。 |
| **Plan-and-Execute** | 像专业厨师：先写菜单、准备食材、按步骤执行、每步验证。 |

#### SOTA 参考
| 公司 | 实现 | 来源 |
|:---|:---|:---|
| **Anthropic** | Claude Managed Agents：先规划任务，再逐步执行 | [Claude Managed Agents](https://esso.dev/blog-posts/claude-managed-agents-when-anthropic-takes-over-the-agent-loop) |
| **LangGraph** | StateGraph + 条件边：明确规划执行路径 | [LangGraph Architecture](https://blog.csdn.net/l35633/article/details/153694551) |

---

## 三、协议层（Protocol Layer）

### 通过标准
**MCP 或结构化函数调用（零正则解析）**

### 为什么这是标准？

#### 核心洞察
正则解析依赖模型输出格式的稳定性，这是 2022-2023 年的旧范式。2026 年的 SOTA 是 **MCP（Anthropic）** 或 **Function Calling（OpenAI）**，它们通过 schema 强制校验，从根本上消除了格式解析失败的风险。

#### 设计原理
```
核心问题：LLM 输出是概率性的，正则解析是确定性的——两者不兼容

旧范式（正则解析）：
1. 让模型输出文本："【Thought】xxx【Action】yyy"
2. 用正则表达式解析：re.search(r"【Thought】(.*)【Action】")
3. 问题：模型可能写错格式（比如把"Thought"写成"思考"）

新范式（结构化协议）：
1. 定义 JSON Schema：{"type": "object", "properties": {"thought": {...}, "action": {...}}}
2. 模型输出结构化 JSON：{"thought": "xxx", "action": "yyy"}
3. 优势：schema 强制校验，模型"被迫"输出正确格式

关键洞察：
- 结构化协议让模型输出"被迫"符合 schema
- 从根本上解决了可靠性问题
- 不再依赖模型"写格式正确"
```

#### 类比解释
| 类比 | 说明 |
|:---|:---|
| **正则解析** | 让 AI 写一封信，然后你用眼睛找关键词。AI 可能写错格式，你可能找错关键词，结果经常误解。 |
| **MCP/Function Calling** | 给 AI 一个表格，让它填空。表格有固定格式，AI 只能填空不能改格式，结果不会误解。 |
| **USB-C 接口** | MCP 就像 USB-C：任何设备都能用，你换电脑设备仍然可用。MCP 是 AI 的"USB-C"。 |

#### MCP vs Function Calling 对比

| 特性 | MCP (Anthropic) | Function Calling (OpenAI) |
|:---|:---|:---|
| **架构** | Server-Client 分离 | 工具定义在请求中 |
| **工具复用** | ✅ 一个 Server 被多个 Agent 使用 | ❌ 工具定义在请求中，无法复用 |
| **动态发现** | ✅ 运行时发现新工具 | ❌ 工具必须硬编码 |
| **跨模型兼容** | ✅ 任何支持 MCP 的模型都能用 | ⚠️ 不同模型 API 有差异 |
| **架构复杂度** | ❌ 需要单独部署 Server | ✅ 无需单独部署 |
| **性能开销** | ❌ 需要额外网络请求 | ✅ 无额外请求 |

#### SOTA 参考
| 公司 | 实现 | 来源 |
|:---|:---|:---|
| **Anthropic** | MCP：开放标准，50+ 官方服务器 | [MCP Official](https://www.anthropic.com/research/model-context-protocol) |
| **OpenAI** | Function Calling + Structured Outputs | [OpenAI Guardrails](https://openai.github.io/openai-agents-python/guardrails/) |

---

## 四、行动层（Action Layer）

### 通过标准
**异步/并行执行与错误容错**

### 为什么这是标准？

#### 核心洞察
生产环境的 Agent 会遇到网络超时、API 限流、服务重启等问题。没有重试机制和异步执行，Agent 会频繁失败。

#### 设计原理
```
核心问题：Agent 的执行链路有多点故障

故障点：
1. LLM 调用：模型可能超时、限流、返回错误
2. 工具执行：工具可能失败、超时、返回异常
3. 网络请求：网络可能断开、延迟、丢包

解决方案：
1. 重试机制：失败后自动重试（指数退避）
2. 异步执行：不阻塞主线程，提高效率
3. 并行执行：多个工具同时执行，节省时间
4. 错误隔离：一个工具失败不影响其他工具

关键洞察：
- 每个故障点都需要独立的错误处理和重试策略
- 异步/并行执行提高效率，减少等待时间
```

#### 类比解释
| 类比 | 说明 |
|:---|:---|
| **无重试机制** | 像打电话一次没接通就放弃，而不是再打几次。 |
| **有重试机制** | 像打电话没接通就等几秒再打，最多打 3 次。 |
| **异步执行** | 像发邮件：你发完就去做别的事，不用等对方回复。 |
| **并行执行** | 像请 3 个人同时做 3 件事，而不是一个人做完再做下一件。 |

#### SOTA 参考
| 公司 | 实现 | 来源 |
|:---|:---|:---|
| **OpenAI** | Guardrails + 重试机制 + 异步执行 | [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/guardrails/) |
| **LangGraph** | Durable Execution + 自动恢复 | [LangGraph Durable](https://github.com/langchain-ai/langgraph/blob/main/README.md) |

---

## 五、记忆层（Memory Layer）

### 通过标准
**分层管理（向量长时记忆 vs 上下文短时记忆）**

### 为什么这是标准？

#### 核心洞察
LLM 的上下文窗口是有限的（即使 GPT-5 有 100k tokens，也会被填满）。没有分层记忆，Agent 会"忘记"之前学到的知识。

#### 设计原理
```
核心问题：上下文窗口有限，无法保存所有历史信息

记忆分层：
1. 短期记忆（Short-term）：当前会话，像 RAM
   - 快速访问
   - 容量有限
   - 会话结束就消失

2. 长期记忆（Long-term）：跨会话，像硬盘
   - 持久保存
   - 需要检索
   - 可以积累知识

3. 工作记忆（Working）：当前任务，像 CPU 缓存
   - 当前任务上下文
   - 滑动窗口控制
   - Token 截断策略

关键洞察：
- 短期记忆像 RAM，快速但容量有限
- 长期记忆像硬盘，持久但需要检索
- 两者分离，才能既保持推理效率，又积累长期知识
```

#### 类比解释
| 类比 | 说明 |
|:---|:---|
| **短期记忆** | 像 RAM（内存）：快速但容量有限，关机就消失。 |
| **长期记忆** | 像硬盘：持久但需要检索，可以跨会话保存知识。 |
| **工作记忆** | 像 CPU 缓存：当前任务需要的上下文，用滑动窗口控制大小。 |
| **记忆晋升** | 像"写日记"：把重要的事情从短期记忆（今天的想法）写到长期记忆（日记本）。 |

#### SOTA 参考
| 公司 | 实现 | 来源 |
|:---|:---|:---|
| **Hermes** | 自进化记忆：自动从短期记忆提取高价值信息晋升到长期记忆 | [Hermes Architecture](https://preview.aclanthology.org/master-new-author-system-ui/2025.emnlp-main.1318.pdf) |
| **Redis** | 分层记忆：短期用 Redis，长期用向量数据库 | [Redis Memory](https://redis.io/blog/ai-agent-memory-stateful-systems/) |

---

## 六、工程化（Engineering Layer）

### 通过标准
**上下文滑动窗口、Token 成本控制与自动化评测**

### 为什么这是标准？

#### 核心洞察
LLM 调用是昂贵的（每次推理都消耗 tokens）。没有成本控制和评测系统，Agent 的运营成本会失控。

#### 设计原理
```
核心问题：LLM 调用成本高，输出质量不稳定

解决方案：
1. 上下文滑动窗口
   - 限制历史消息长度
   - 保留最近 N 条消息
   - 截断过长内容

2. Token 成本控制
   - 监控每次调用的 token 消耗
   - 设置 token 上限
   - 压缩上下文（摘要、向量检索）

3. 自动化评测（Evals）
   - 定义评测标准
   - 自动评分
   - 持久化评测结果

关键洞察：
- 上下文滑动窗口防止 token 爆炸
- 自动化评测确保 Agent 输出质量稳定
- 成本控制让 Agent 运营可持续
```

#### 类比解释
| 类比 | 说明 |
|:---|:---|
| **滑动窗口** | 像聊天只保留最近 10 条消息，而不是保存所有历史。 |
| **Token 成本** | 像手机流量：每次调用都消耗"流量"，需要控制用量。 |
| **自动化评测** | 像考试自动评分：每次输出都自动检查是否达标。 |

#### SOTA 参考
| 公司 | 实现 | 来源 |
|:---|:---|:---|
| **LangSmith** | 自动化评测 + 监控 + 调试 | [LangSmith](https://www.langchain.com/langsmith) |
| **OpenAI Evals** | 自动化评测框架 | [OpenAI Evals](https://github.com/openai/openai-agents-python) |

---

## 七、安全性（Security Layer）

### 通过标准
**执行沙箱、路径白名单及敏感操作的人工确认机制**

### 为什么这是标准？

#### 核心洞察
Agent 会执行代码、读写文件、调用 API。没有沙箱隔离，Agent 可能误删文件、泄露数据、执行恶意代码。

#### 设计原理
```
核心问题：Agent 有能力执行危险操作，需要安全护栏

安全层级：
1. 沙箱隔离（Sandbox）
   - 容器化执行
   - 资源限制（CPU、内存、网络）
   - 与宿主系统隔离

2. 路径白名单（Path Whitelist）
   - 只允许访问特定目录
   - 禁止访问系统文件
   - 禁止访问敏感数据

3. 敏感操作确认（Human-in-the-loop）
   - 删除文件：需要用户确认
   - 发送邮件：需要用户确认
   - 调用支付 API：需要用户确认

4. Guardrails（护栏）
   - 输入验证：检查用户输入是否合法
   - 输出过滤：检查 Agent 输出是否包含敏感信息
   - 快速失败：发现危险操作立即终止

关键洞察：
- 沙箱隔离防止 Agent 影响宿主系统
- 路径白名单限制 Agent 只能访问特定目录
- Human-in-the-loop 防止 Agent 自主执行高风险操作
```

#### 类比解释
| 类比 | 说明 |
|:---|:---|
| **沙箱隔离** | 像把 AI 放在玻璃房里：它可以做事，但无法触碰外面的世界，即使它犯错也不会影响你的系统。 |
| **路径白名单** | 像只给 AI 一把"特定房间的钥匙"，而不是"整栋楼的钥匙"。 |
| **Human-in-the-loop** | 像自动驾驶的"确认按钮"：AI 可以做大部分事，但关键决策需要你按下确认按钮才能执行。 |
| **Guardrails** | 像护栏：AI 在高速公路上行驶，护栏防止它冲出道路，即使它想犯错也被限制在安全范围内。 |

#### SOTA 参考
| 公司 | 实现 | 来源 |
|:---|:---|:---|
| **GKE** | Agent Sandbox：容器隔离 + NetworkPolicy | [GKE Secure Agents](https://codelabs.developers.google.com/codelabs/gke/ai-agents-on-gke) |
| **OpenAI** | Guardrails：输入验证 + 输出过滤 | [OpenAI Guardrails](https://openai.github.io/openai-agents-python/guardrails/) |
| **Anthropic** | Constitutional AI：安全护栏 + 快速失败 | [Anthropic Safety](https://www.anthropic.com/research/model-context-protocol) |

---

## 八、总结：七大里程碑的关系

```
Agent = LLM + System + Environment

┌─────────────────────────────────────────────────────────────┐
│                    Environment（环境）                        │
│    - 感知层：从环境获取信息                                   │
│    - 安全性：隔离环境，防止危险                               │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    LLM（大脑）                                │
│    - 大脑层：推理、规划、反思                                 │
│    - 记忆层：短期/长期记忆                                    │
│    - 工程化：Token 控制、评测                                 │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    System（系统）                             │
│    - 协议层：工具调用协议                                     │
│    - 行动层：执行、重试、异步                                 │
└─────────────────────────────────────────────────────────────┘

关键洞察：
- 感知层 + 安全性：处理"环境"部分
- 大脑层 + 记忆层 + 工程化：处理"LLM"部分
- 协议层 + 行动层：处理"系统"部分
```

---

## 九、评估时的判断标准

| 里程碑 | 通过判断 | 不通过判断 |
|:---|:---|:---|
| **感知层** | 有多模态输入（图像/音频）+ 有反馈闭环（验证行动结果） | 只能读文本 + 无闭环验证 |
| **大脑层** | 有 Plan-and-Execute 或 Reflection 循环 | 只有 ReAct（无规划/无自检） |
| **协议层** | 使用 MCP 或 Function Calling + 有 Schema 校验 | 使用正则解析 Thought/Action |
| **行动层** | 有重试机制 + 异步/并行执行 | 无重试 + 同步阻塞执行 |
| **记忆层** | 有短期/长期分离 + 有记忆晋升机制 | 只有单一记忆 + 无晋升 |
| **工程化** | 有滑动窗口 + Token 控制 + 自动化评测 | 无控制 + 无评测 |
| **安全性** | 有沙箱隔离 + 路径白名单 + Human-in-the-loop | 无隔离 + 无限制 + 无确认 |