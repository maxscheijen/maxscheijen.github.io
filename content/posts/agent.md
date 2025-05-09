---
title: Basic LLM Agent with Tool Calling
date: 2025-05-08
---

Over the past few years, agents have become a popular topic in the AI community. Many frameworks allow you to build agents, often serving as abstractions for calling LLMs, parsing, using tools, and chaining calls. However, the basic concept behind a tool-calling agent is simple.

In this post, I share a straightforward, bare-bones implementation of an agent that can call tools.

## Agent

Let's start by defining what I mean by an **agent**. Here, I follow Anthropic's definition of an agent.

> "***[Agents](https://www.anthropic.com/engineering/building-effective-agents)** are systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks"*

The **key points** are:

1. Direct own processes and tool usage.
2. Accomplish tasks.

Let's operationalize these two points:

1. Implement a loop that keeps calling an LLM (optional but advised: set a maximum number of iterations).
2. Identify when a task is accomplished (e.g., when the LLM stops calling tools).

This is the structural setup of a simple agent:

- System prompt/instruction (Optional).
- A large language model client.
- List of tools, e.g. a list of functions (Optional).

Let's create a simple `Agent` class based on this structure.

```python
from dataclasses import dataclass, field
from typing import Any, Callable


@dataclass
class Agent:
    system_prompt: str = ""
    model: str = "gpt-4o-mini"
    tools: list[Callable] = field(default_factory=list)
```

The `Agent` class is a simple `dataclass` that takes a system prompt, a model name and a list of tools.

In the `__post_init__` method, we initialize the `messages` list and the `tool_mapping` dictionary. The `messages` list is used to store the messages that are sent to the LLM. The `tool_mapping` dictionary is used to map the tool names to their corresponding functions.

The `tool_schema` is a list of function schemas. The `function_schema` function is used to create the schema for each tool. This is just a helper function that takes a function and returns function schema expected by OpenAI.

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

Let's now look at implementing those two key points.

### Agentic Loop

The `run` method implements a loop that calls the OpenAI API with a schema of available tools.

The main idea is to keep calling the large language model until it stops calling tools or until the `max_iterations` is reached. Setting `max_iterations` is optional, but it can be useful to **avoid infinite loops**.

The tool calls, parameters and tool results are added to the `messages` list. The `messages` list is used to keep track of the **conversation history**, which is essential because the LLM needs to know the **context of the conversation** in order to generate relevant responses.

My assumption is that the task is **accomplished when the LLM stops calling tools**.

This is a simplification, but it works for most cases. In practice, you might want to add more sophisticated logic to determine when a task is accomplished.

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

The `call_tool` method is used to call the tool with the parameters passed in the tool call message. The `tool_mapping` dictionary (constructed in the `__post_init__`) maps the tool name to its corresponding function.

The function is called with the parameters identified by the large language model based on the function/tool schema.

```python
@dataclass
class Agent:
    ...
    def call_tool(tool: ChatCompletionMesssageToolCall) -> Any:
        return self.tool_mapping[tool.function.name](
            **json.loads(tool.function.arguments)
        )
```

The `Agent` classes uses helper methods to add tool call messages and tool result messages to the `messages` list.

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

We can now run the agent with a query and see if it uses tools and stops when the task is accomplished.

I'll add a simple `get_weather` tool that retreives weather data based on longitude and latitude. The code for this can be found at the end of this post.

```python
agent = Agent(
    system_prompt="You are a helpful assistant.",
    model="gpt-4o-mini",
    tools=[get_weather],
)
agent.run(query="What is the weather in Amsterdam?")
```

This results in:

```
The current weather in Amsterdam is as follows:
- Temperature: 15.2°C
- Wind Speed: 11.2 km/h
```

Looking at the messages we can see that the agent called the `get_weather` tool and passed the parameters to it. The result of the tool call was then added to the messages list.

```json
[
    {
        "role": "system",
        "content": "You are a helpful assistant."
    },
    {
        "role": "user",
        "content": "Whats the weather in Amsterdam?"
    },
    {
        "role": "assistant",
        "content": null,
        "tool_calls": [
            {
                "id": "call_NaBl8NWIHZRBU92bhfc5kCq1",
                "type": "function",
                "function": {
                    "name": "get_weather",
                    "arguments": {
                        "latitude":52.3676,
                        "longitude":4.9041
                    }
                }
            }
        ]
    },
    {
        "role": "tool",
        "name": "get_weather",
        "tool_call_id": "call_NaBl8NWIHZRBU92bhfc5kCq1",
        "content":
            {
                "latitude": 52.366,
                "longitude": 4.901,
                "current": {
                    "time": "2025-05-08T09:45",
                    "interval": 900,
                    "temperature_2m": 15.2,
                    "wind_speed_10m": 11.2
                }
            }
        },
    {
        "role": "assistant",
        "content": "The current weather in Amsterdam is as follows:\n- Temperature: 15.2\n- Wind Speed: 11.2 km/h\n\nIf you need more details or a forecast, let me know!"
    }
]

```

This is a super simple implementation of an agent that can call tools. Tools can be anything, including API requests to external services, database queries, or any other function that can be called with parameters.

Tools can also be large language model calls; for example, you could add a reflection tool that reflects on the answer.

## Appendix

Some additional details to run the full example.

### Tool

Here is the simple `get_weather` tool that gets the weather data based on longitude and latitude. It uses the Open-Meteo API to get the weather data.


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

### Tool/Function Schema Generation

The `function_schema`  function is used to create the schema for each tool. This is a helper function that takes a function and returns the function schema expected by OpenAI. You can also define this schema manually.

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

