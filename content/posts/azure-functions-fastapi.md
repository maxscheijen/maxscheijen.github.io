---
title: Azure Functions and FastAPI
date: 2024-12-14
tags: cloud
---

I'm a huge fan of [FastAPI](https://fastapi.tiangolo.com). It's a high-performance web framework for building APIs in Python, based on type hints and Pydantic, which enables automatic documentation and data validation.

At work, we extensively use [Azure Functions](https://learn.microsoft.com/en-us/azure/azure-functions/). These are serverless resources that reduce the need for infrastructure configuration and maintenance, allowing us to focus more on writing code. To create Azure Functions in Python, we can use the [azure-functions](https://pypi.org/project/azure-functions/) package.

However, I really love FastAPI as a framework for building APIs. Let's explore how we can create an Azure Function leveraging FastAPI.

## Setup

Let's start by setting up the following project structure:

```
azure-function-fastapi
├── api
│   └── __init__.py
├── function_app.py
├── host.json
└── local.settings.json
```

Where:

- `function_app.py` is the Azure Function entry point
- `__init__.py` is where we create our FastAPI app
- `host.json` defines global settings and behaviors
- `local.settings.json` contains settings used for local running and debugging

## Creating a FastAPI App

Let's build our FastAPI app in `__init__.py`. This can be done exactly as you would normally create a FastAPI application:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class User(BaseModel):
  name: str


@app.get("/hello/{name}", response_model=User)
async def get_name(name: str):
    return User(name=name)
```

This creates a parameterized route (`/hello/{name}`) that dynamically accepts a value in the URL path and returns a JSON response (`User`).

You can run this app from the root using:

```sh
uvicorn api.__init__:app
```

Let's make a `GET` request using `curl` to our API:

```sh
curl http://127.0.0.1:8000/hello/Max
```

Response

```json
{
  "name": "Max"
}
```

Automatic documentation can be found at `[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)`.

## Integrating in Azure Function

Integrating the FastAPI app into an Azure Function is straightforward:

Add the following to `function_app.py`:

```python
from azure.functions import AsgiFunctionApp, AuthLevel

from api import app as fastapi_app

app = AsgiFunctionApp(
  app=fastapi_app,
  http_auth_level=AuthLevel.ANONYMOUS
)
```

We simply import our FastAPI app and use it in `AsgiFunctionApp`. It's that simple.

Next, set the function app settings in `host.json`:

```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "excludedTypes": "Request"
      }
    }
  },
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[2.*, 3.0.0)"
  },
  "extensions": 
  {
    "http": 
    {
        "routePrefix": ""
    }
  }
}
```

Define the local settings in `local.settings.json` for running the function app locally:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "AzureWebJobsFeatureFlags": "EnableWorkerIndexing"
  }
}
```

You can now start the function app locally:

```sh
func host start
```

You now have a FastAPI app running in an Azure Function!

> **Tip**: You can still run your FastAPI app without the azure function, which can be helpful during development:

```sh
uvicorn api.__init__:app --reload
```

## Links

[[azure]] [[python]] [[fastapi]] [[serverless]]
