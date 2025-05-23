---
title: A MCP Server
date: 2025-04-18
---

Lately, I’ve become increasingly interested in tools for large language models (LLMs), and I started exploring the [Model Context Protocol](https://modelcontextprotocol.io/) (MCP). In this post, I’ll showcase a simple MCP server I built that allows clients to interact with Yahoo Finance to retrieve stock prices and company information.

The [`mcp-yahoo-finance`](https://github.com/maxscheijen/mcp-yahoo-finance) server is available on my [GitHub](https://github.com/maxscheijen/mcp-yahoo-finance).

## High level Design

This project is a minimal MCP tool server that enables LLMs to retrieve live stock data from Yahoo Finance. It demonstrates how to wrap real-world APIs into standardized, discoverable MCP tools that LLMs can reason about and call.

###  Wrapping Existing API

The logic for retrieving stock prices resides in a small utility class. It uses the [`yfinance`](https://github.com/ranaroussi/yfinance) Python library under the hood:

```python
class YahooFinance:
    def __init__(self, session: Session | None = None, verify: bool = True) -> None:
        self.session = session or Session()
        self.session.verify = verify

    def get_current_stock_price(self, symbol: str) -> str:
        """Get the current stock price based on stock symbol.

        Args:
            symbol (str): Stock symbol in Yahoo Finance format.
        """
        stock = Ticker(ticker=symbol, session=self.session).info
        current_price = stock.get(
            "regularMarketPrice", stock.get("currentPrice", "N/A")
        )
        return (
            f"{current_price:.4f}"
            if current_price
            else f"Couldn't fetch {symbol} current price"
        )

    # More methods to retrieve and aggregate data
```

It’s minimal, but it works well as a testbed for building and exposing MCP-compatible tools.

## Generate Tool Schemas

Tool schemas provide LLMs with the necessary context about what parameters a tool expects.

When using the `mcp.server.FastMCP` class, tool schemas are generated automatically based on a function’s docstring. However, in the examples here, I’m using the lower-level `mcp.server.Server` class. This approach offers greater control over the schema, allowing us to define schemas manually or build a custom docstring parser for methods and functions.

```python
import inspect
from typing import Any

from mcp.types import Tool


def parse_docstring(docstring: str) -> dict[str, str]:
    """Parses a Google-style doc string to extract parameter descriptions."""
    descriptions = {}
    if not docstring:
        return descriptions

    lines = docstring.split("\n")
    current_param = None

    for line in lines:
        line = line.strip()
        if line.startswith("Args:"):
            continue
        elif line and "(" in line and ")" in line and ":" in line:
            param = line.split("(")[0].strip()
            desc = line.split("):")[1].strip()
            descriptions[param] = desc
            current_param = param
        elif current_param and line:
            descriptions[current_param] += " " + line.strip()

    return descriptions


def generate_tool(func: Any) -> Tool:
    """Generates a tool schema from a Python function."""
    signature = inspect.signature(func)
    docstring = inspect.getdoc(func) or ""
    param_descriptions = parse_docstring(docstring)

    schema = {
        "name": func.__name__,
        "description": docstring.split("Args:")[0].strip(),
        "inputSchema": {
            "type": "object",
            "properties": {},
        },
    }

    for param_name, param in signature.parameters.items():
        param_type = "number" if param.annotation is float else "string"

        schema["inputSchema"]["properties"][param_name] = {
            "type": param_type,
            "description": param_descriptions.get(param_name, ""),
        }

        if "required" not in schema["inputSchema"]:
            schema["inputSchema"]["required"] = [param_name]
        else:
            if "=" not in str(param):
                schema["inputSchema"]["required"].append(param_name)

    return Tool(**schema)
```

As long as your function includes clear type hints and a structured docstring, it can be automatically exposed to clients as an MCP tool.

## MCP Server

Finally, the core server uses the `mcp.server.Server` class and defines two key handlers:

- `list_tools()` informs the client of the available functions/tools.
- `call_tool()` handles tool invocations and their arguments

```python
from mcp.server import Server


async def serve() -> None:
    server = Server("mcp-yahoo-finance")
    yf = YahooFinance()

    @server.list_tools()
    async def list_tools() -> list[Tool]:
        return [generate_tool(yf.get_current_stock_price)]

    @server.call_tool()
    async def call_tool(name: str, args: dict[str, Any]) -> list[TextContent]:
        match name:
            case "get_current_stock_price":
                price = yf.get_current_stock_price(**args)
                return [TextContent(type="text", text=price)]
            case _:
                raise ValueError(f"Unknown tool: {name}")

```

By using `stdio_server()`, this project can be plugged directly into an LLM runtime that supports MCP—no web server required.

- [github.com/maxscheijen/mcp-yahoo-finance](https://github.com/maxscheijen/mcp-yahoo-finance)
- [modelcontextprotocol.io/](https://modelcontextprotocol.io/)

