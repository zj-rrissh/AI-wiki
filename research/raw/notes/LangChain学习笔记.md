---
tags:
  - 总结
---

# LangChain 学习笔记

> 本笔记覆盖 LangChain 1.0 的核心用法：Agent 构建流程、工具调用、记忆机制、LCEL 表达式语言。

---

## 目录

- [构建基本代理](#构建基本代理)
- [构建真实世界代理](#构建真实世界代理)
  - [定义系统提示](#定义系统提示)
  - [创建工具](#创建工具)
  - [配置模型](#配置模型)
  - [结构化输出](#结构化输出)
  - [添加记忆](#添加记忆)
  - [创建并运行代理](#创建并运行代理)
- [LCEL](#lcel)

---

## 构建基本代理

```python
from langchain.agents import create_agent


def get_weather(city: str) -> str:
    """获取指定城市的天气。"""
    return f"{city}总是阳光明媚！"

agent = create_agent(
    model="anthropic:claude-sonnet-4-5",
    tools=[get_weather],
    system_prompt="你是一个乐于助人的助手",
)

# 运行代理
agent.invoke(
    {"messages": [{"role": "user", "content": "旧金山的天气怎么样"}]}
)
```

> 相关笔记：[[导航|LangChain 1.0 核心抽象]] · [[阶段三#结构化输出链]]


---

## 构建真实世界代理

完整流程分为 6 个步骤：

1. **定义系统提示** → 获得更好的代理行为
2. **创建工具** → 与外部数据集成
3. **配置模型** → 确保响应一致性
4. **结构化输出** → 获得可预测的结果
5. **添加记忆** → 实现类似聊天的交互
6. **创建并运行代理** → 构建功能完整的代理

> 对应阶段三学习内容：[[阶段三]]


### 定义系统提示

```python
SYSTEM_PROMPT = """你是一位擅长用双关语表达的专家天气预报员。

你可以使用两个工具：

- get_weather_for_location：用于获取特定地点的天气
- get_user_location：用于获取用户的位置

如果用户询问天气，请确保你知道具体位置。如果从问题中可以判断他们指的是自己所在的位置，请使用 get_user_location 工具来查找他们的位置。"""
```


### 创建工具

```python
from dataclasses import dataclass
from langchain.tools import tool, ToolRuntime

@tool
def get_weather_for_location(city: str) -> str:
    """获取指定城市的天气。"""
    return f"{city}总是阳光明媚！"

@dataclass
class Context:
    """自定义运行时上下文模式。"""
    user_id: str

@tool
def get_user_location(runtime: ToolRuntime[Context]) -> str:
    """根据用户 ID 获取用户信息。"""
    user_id = runtime.context.user_id
    return "Florida" if user_id == "1" else "SF"
```

> LangChain 的 [`@tool` 装饰器](https://reference.langchain.com/python/langchain/tools/#langchain.tools.tool) 会添加元数据，并通过 `ToolRuntime` 参数启用运行时注入。


### 配置模型

```python
from langchain.chat_models import init_chat_model

model = init_chat_model(
    "anthropic:claude-sonnet-4-5",
    temperature=0.5,
    timeout=10,
    max_tokens=1000
)
```


### 结构化输出

```python
from dataclasses import dataclass

# 这里使用 dataclass，但也支持 Pydantic 模型。
@dataclass
class ResponseFormat:
    """代理的响应模式。"""
    # 带双关语的回应（始终必需）
    punny_response: str
    # 天气的任何有趣信息（如果有）
    weather_conditions: str | None = None
```


### 添加记忆

```python
from langgraph.checkpoint.memory import InMemorySaver

checkpointer = InMemorySaver()
```


### 创建并运行代理

```python
agent = create_agent(
    model=model,
    system_prompt=SYSTEM_PROMPT,
    tools=[get_user_location, get_weather_for_location],
    context_schema=Context,
    response_format=ResponseFormat,
    checkpointer=checkpointer
)

# `thread_id` 是给定对话的唯一标识符。
config = {"configurable": {"thread_id": "1"}}

response = agent.invoke(
    {"messages": [{"role": "user", "content": "外面的天气怎么样？"}]},
    config=config,
    context=Context(user_id="1")
)

print(response['structured_response'])
# ResponseFormat(
#     punny_response="佛罗里达今天依然是'阳光灿烂'的一天！阳光正在播放'rey-dio'热门歌曲！...",
#     weather_conditions="佛罗里达总是阳光明媚！"
# )

# 注意，我们可以使用相同的 `thread_id` 继续对话。
response = agent.invoke(
    {"messages": [{"role": "user", "content": "谢谢！"}]},
    config=config,
    context=Context(user_id="1")
)

print(response['structured_response'])
# ResponseFormat(
#     punny_response="你真是'雷'厉风行地欢迎！...",
#     weather_conditions=None
# )
```

> 关联项目：[[阶段三#PDF文档问答助手]]（生产级 RAG + 记忆实现）


---

## LCEL

**LCEL（LangChain Expression Language）** 是 LangChain 框架中一种声明式的编程方式，旨在用更简洁、灵活的语法组合不同组件（Prompt、模型、解析器）。

### 核心语法

将上一个组件的输出**自动作为下一个组件的输入**：

```python
chain = prompt | model | output_parser
```

### 标准接口：Runnable 协议

LCEL 中的所有组件都遵循 **Runnable 协议**，统一方法：

| 方法 | 用途 |
| :-- | :-- |
| `invoke()` | 处理单个输入 |
| `batch()` | 并行处理多个输入 |
| `stream()` | 流式返回响应内容 |
| `ainvoke/abatch/astream` | 对应的异步执行版本 |

### 示例：结构化输出链

完整示例见 [[阶段三#结构化输出链]]。

```python
from langchain_core.prompts import ChatPromptTemplate
from pydantic import BaseModel
from langchain_core.output_parsers import PydanticOutputParser
from langchain.chat_models import init_chat_model

# 定义输出schema
class IntentEntity(BaseModel):
    intent: str
    entities: dict

parser = PydanticOutputParser(pydantic_object=IntentEntity)

prompt = ChatPromptTemplate.from_template(
    "提取用户意图和实体...\n{format_instructions}\n\n用户输入: {input}"
).partial(format_instructions=parser.get_format_instructions())

model = init_chat_model("qwen2.5:7b", model_provider="ollama")

# LCEL 链：prompt → model → parser
chain = prompt | model | parser

result = chain.invoke({"input": "我是一个大学生，想要学习 agent 开发"})
```

> 相关学习路径：[[导航#阶段三：LangChain 1.0+/LlamaIndex精通]]
