---
title: Basic LLM Agent with Tool Calling
date: 2025-05-08
---

Over the past few years, agents have become a popular topic in the AI community. Many frameworks now allow you to build agents, often serving as abstractions for calling LLMs, parsing inputs, using tools, and chaining calls. However, the basic concept behind a tool-calling agent is straightforward.

In this post, I’ll share a simple, bare-bones implementation of an agent that can call tools.

## Agent

Let’s begin by defining what I mean by an agent. Here, I adopt Anthropic’s definition:

> "***[Agents](https://www.anthropic.com/engineering/building-effective-agents)** are systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks"*

The **key characteristics** of agents are:

1.	They direct their own processes and tool usage.
2.	They accomplish tasks.

Let’s operationalize these two points:

1.	Implement a loop that keeps calling an LLM (optionally, set a maximum number of iterations).
2.	Detect when a task is accomplished (e.g., when the LLM stops calling tools).

This is the basic structure of a simple agent:

- System prompt/instruction (optional)
- A large language model client
- A list of tools (e.g., functions) (optional)

Let’s create a simple `Agent` class based on this structure:
```python hl_lines="5 6"
from dataclasses import dataclass, field
from typing import Any, Callable


@dataclass
class Agent:
    system_prompt: str = ""
    model: str = "gpt-4o-mini"
    tools: list[Callable] = field(default_factory=list)
```

The `Agent` class is a `dataclass` that takes a system prompt, a model name, and a list of tools.

In the `__post_init__` method, we initialize the `messages` list and the tool_mapping dictionary. The messages list stores the conversation history sent to the LLM. The `tool_mapping` dictionary maps tool names to their corresponding functions.

The `tool_schema` is a list of function schemas. The `function_schema` function helps create schemas expected by the OpenAI API. Alternatively, you can define this schema manually.

You can also define this tool/function schema manually.

```python
@dataclass
class Agent:
    ...
    def __post_init__(self):
        self.messages: list[dict[str, Any]] = []
        self.tool_mapping = {tool.__name__: tool for tool in self.tools}
        self.tool_schema = [function_schema(tool) for tool in self.tools]

        if self.system_prompt:
            self.messages.append({"role": "system", "content": self.system_prompt})
```

### Agentic Loop

The `run` method implements a loop that calls the OpenAI API with the available tool schemas.

The core idea is to keep calling the LLM until it stops invoking tools or until `max_iterations` is reached. Setting a `max_iterations` value is advisable to **avoid infinite loops**.

Tool calls, parameters, and results are added to the messages list. Maintaining the **conversation history** is crucial so that the LLM retains context and generates relevant responses.

Here, the assumption is that a task is **accomplished when the LLM stops calling tools**—a useful simplification for many use cases.

```python
from openai import OpenAI


@dataclass
class Agent:
    ...
    def run(self, query: str, max_iterations: int = 5) -> str:
        client = OpenAI()

        self.messages.append({"role": "user", "content": query})

        # 1. Some kind of loop that keeps calling an LLM
        for _ in range(max_iterations):
            response = client.chat.completions.create(
                messages=self.messages,
                model=self.model,
                temperature=0,
                tools=self.tool_schema,
            )

            if tool_calls := response.choices[0].message.tool_calls:
                for tool in tool_calls:
                    tool_result = self.call_tool(tool)
                    self.add_tool_call_message(tool, tool_result)
                    self.add_tool_result_message(tool)

            # 2. No more tool calls: Accomplish tasks
            else:
                self.add_assistant_message(response.choices[0].message.content or "")
                break

        return self.messages[-1]["content"]
```

### Tool Calling

The `call_tool` method invokes a tool with the parameters provided by the LLM. The `tool_mapping` dictionary maps function names to actual implementations.

```python
@dataclass
class Agent:
    ...
    def call_tool(self, tool: ChatCompletionMesssageToolCall) -> Any:
        return self.tool_mapping[tool.function.name](
            **json.loads(tool.function.arguments)
        )
```

The `Agent` class also includes helper methods to record tool calls and results in the messages list:

```python
from openai.types.chat import ChatCompletionMessageToolCall


@dataclass
class Agent:
    ...
    def add_tool_call_message(self, tool: ChatCompletionMessageToolCall):
        self.messages.append(
            {
                "role": "assistant",
                "content": None,
                "tool_calls": [
                    {
                        "id": tool.id,
                        "type": tool.type,
                        "function": {
                            "name": tool.function.name,
                            "arguments": tool.function.arguments,
                        },
                    }
                ],
            }
        )

    def add_tool_result_message(
        self,
        tool: ChatCompletionMessageToolCall,
        tool_result: Any
    ):
        self.messages.append(
            {
                "role": "tool",
                "name": tool.function.name,
                "tool_call_id": tool.id,
                "content": f"{tool_result}",
            }
        )

```

### Running the Agent

Now, let’s run the agent using a simple query. We’ll use a basic `get_weather` tool to retrieve weather data based on geographic coordinates. (The code for this tool is included in the appendix.)

```python
agent = Agent(
    system_prompt="You are a helpful assistant.",
    model="gpt-4o-mini",
    tools=[get_weather],
)
agent.run(query="What is the weather in Amsterdam?")
```

This yields:

```
The current weather in Amsterdam is as follows:
- Temperature: 15.2°C
- Wind Speed: 11.2 km/h
```

The agent called the get_weather tool, passed the necessary parameters, and included the tool’s output in the final response.

## Appendix

### Tool Implementation

Here is the simple `get_weather` function, which uses the Open-Meteo API:


```python
import requests


def get_weather(latitude: float, longitude: float) -> dict | str:
    """Get the weather data for a given latitude and longitude using Open-Meteo API.

    Args:
        latitude (float): Latitude of the location.
        longitude (float): Longitude of the location.
    """

    with requests.Session() as session:
        response = session.get(
            f"https://api.open-meteo.com/v1/forecast?latitude={latitude}&longitude={longitude}&current=temperature_2m,wind_speed_10m&hourly=temperature_2m,relative_humidity_2m,wind_speed_10m",
        )
        if response.status_code == 200:
            data = response.json()
            data.pop("hourly_units")
            data.pop("hourly")
            return data
        else:
            return "Error: Unable to fetch data"

```

### Function Schema Generation

The `function_schema` helper generates an OpenAI-compatible schema from a function’s docstring and type annotations:

```python
from typing import Any, Callable


def openai_type_mapping(param_type: Any) -> str:
    if param_type is int:
        return "number"
    if param_type is float:
        return "number"
    if param_type is str:
        return "string"
    if param_type is bool:
        return "boolean"
    else:
        return "object"


def parse_param_types_from_docstring(docstring: str) -> dict:
    param_descriptions = {}
    current_param = None

    for line in docstring.split("\n"):
        if line.startswith("Args:"):
            continue

        elif line and "(" in line and ")" in line and ":" in line:
            param = line.split("(")[0].strip()
            desc = line.split("):")[1].strip()
            param_descriptions[param] = desc
            current_param = param

        elif current_param and line:
            param_descriptions[current_param] += " " + line.strip()

    return param_descriptions


def function_schema(function: Callable) -> dict:
    name, description = function.__name__, function.__doc__ or ""
    param_descriptions = parse_param_types_from_docstring(description)

    annoations = {
        param: {
            "type": openai_type_mapping(param_type),
            "description": param_descriptions[param].strip(),
        }
        for param, param_type in function.__annotations__.items()
        if param != "return"
    }

    return {
        "type": "function",
        "function": {
            "name": name,
            "description": description.split("Args:")[0].strip(),
            "parameters": {
                "type": "object",
                "properties": annoations,
                "required": [
                    key for key in function.__annotations__.keys() if key != "return"
                ],
                "additionalProperties": False,
            },
            "strict": True,
        },
    }
```

