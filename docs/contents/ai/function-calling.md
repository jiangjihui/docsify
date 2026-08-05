# Function Calling / Tool Use

## 概览

**Function Calling（函数调用，也称 Tool Use）** 是让 LLM"会动手"的关键机制：模型不再只输出文本，而是可以**输出"要调用哪个函数、参数是什么"的结构化指令**，由你的程序真正执行，再把结果喂回模型继续推理。

它是 Agent、RAG 工具、自动化工作流的底层引擎。

```
用户："北京天气怎么样？"
      │
      ▼
模型思考 → 决定调用 get_weather(city="北京")
      │
      ▼
运行时执行 get_weather() → "晴 25°C"
      │
      ▼
模型拿到结果 → 生成自然语言回答
```

> 前置阅读：[LLM 基础](/contents/ai/llm-basics.md)、[提示词工程](/contents/ai/prompt-engineering.md)。进阶见 [AI Agent](/contents/ai/ai-agent.md)、[LangGraph](/contents/ai/langgraph.md)、[多智能体协作模式](/contents/ai/multi-agent-patterns.md)。

---

### 什么是 Function Calling？

Function Calling 让模型具备**结构化行动能力**：

1. 你向模型声明一组**工具（函数）**及其参数 schema（JSON Schema）。
2. 模型根据用户意图，**判断是否调用、调用哪个、参数填什么**，返回结构化的 `tool_calls`。
3. 你的代码执行该函数，拿到真实结果。
4. 把结果作为 `tool` 消息回传给模型，模型据此生成最终回答或决定下一步。

模型本身**不执行函数**，它只"提建议"；真正执行的是你的运行时。这一分工既灵活又可控。

> 不同厂商叫法不同：OpenAI 称 tool/function calling，Anthropic 称 tool use，Google 称 function declarations——本质相同。

### 为什么需要它？

纯文本 LLM 只能"说"，不能"做"。Tool Use 把模型接入真实世界：

- **计算**：数学、日期、单位换算（比模型心算可靠）。
- **检索**：查数据库、搜索引擎、知识库（见 [RAG](/contents/ai/rag.md)）。
- **操作**：发邮件、写文件、调 API、下单。
- **感知**：读文件、读传感器、读外部状态。

### 工作流程

```
① 声明工具（名称/描述/参数 schema）
      │
② 带 tools 调用模型
      │
③ 模型返回 tool_calls（函数名 + 参数）
      │
④ 运行时执行函数 → 结果
      │
⑤ 结果以 tool 消息回传模型
      │
⑥ 模型继续推理 → 最终回答 / 再调工具（循环）
```

这是一个**"思考—行动—观察"的循环**（ReAct），可重复多次，直到模型认为任务完成。

### 工具定义（Schema）

工具用 **JSON Schema** 描述，让模型知道"能调什么、参数怎么填"：

```json
{
  "type": "function",
  "function": {
    "name": "get_weather",
    "description": "查询指定城市的当前天气",
    "parameters": {
      "type": "object",
      "properties": {
        "city": { "type": "string", "description": "城市名，如 北京" }
      },
      "required": ["city"]
    }
  }
}
```

`description` 越清晰，模型越会正确选工具、填对参数——这是 [提示词工程](/contents/ai/prompt-engineering.md) 的延伸。

### 代码示例

**原生（OpenAI 风格）**：

```python
import openai

tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "查询指定城市的天气",
        "parameters": {
            "type": "object",
            "properties": {"city": {"type": "string"}},
            "required": ["city"],
        },
    },
}]

resp = openai.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "北京天气怎么样？"}],
    tools=tools,
)
print(resp.choices[0].message.tool_calls)
# → [ChatCompletionMessageToolCall(...)]，含函数名与解析好的参数
```

**LangChain（更简洁）**：

```python
from langchain_core.tools import tool

@tool
def get_weather(city: str) -> str:
    """查询指定城市的天气。"""
    return f"{city} 晴 25°C"

# 模型可在 Agent 中直接调用 get_weather
print(get_weather.name, get_weather.description, get_weather.args)
```

### 与 ReAct / Agent 的关系

Tool Use 是 **Agent 的发动机**：Agent = 模型 + 工具 + 循环决策，而"循环决策"正是通过反复 Function Calling 实现的（见 [AI Agent](/contents/ai/ai-agent.md)、[LangGraph](/contents/ai/langgraph.md)）。多 Agent 系统里，每个 Agent 通常也以 Tool Use 与其他 Agent/工具交互（见 [多智能体协作模式](/contents/ai/multi-agent-patterns.md)）。

### 常用模式

| 模式 | 说明 |
|------|------|
| **单工具调用** | 一次调用完成任务 |
| **多工具选择** | 模型从多个工具里挑最合适的 |
| **并行调用** | 一次返回多个互不依赖的 tool_calls，并发执行提速 |
| **强制调用** | 用 `tool_choice` 强制先调某个工具（如先抽取结构化字段） |
| **调用前确认** | 敏感操作先让人批准（见 [AI 安全与防护](/contents/ai/ai-safety.md)） |

### 风险与注意

- **提示注入**：工具返回的内容可能含恶意指令，诱导模型执行危险操作（见 [AI 安全与防护](/contents/ai/ai-safety.md)）。
- **参数不可信**：模型填的参数要做校验与类型约束（Pydantic schema）。
- **成本与延迟**：每轮工具调用都是一次模型交互，需控制循环次数。
- **副作用**：写操作（发邮件、下单）必须有确认/幂等设计。

### 相关资源

- [LLM 基础](/contents/ai/llm-basics.md) / [提示词工程](/contents/ai/prompt-engineering.md) — 前置
- [AI Agent（智能体）](/contents/ai/ai-agent.md) — Tool Use 之上的 Agent
- [LangGraph](/contents/ai/langgraph.md) — 用图编排工具调用循环
- [多智能体协作模式](/contents/ai/multi-agent-patterns.md) — 工具在多 Agent 间的协作
- [OpenAI Tool Calling 文档](https://platform.openai.com/docs/guides/function-calling)
