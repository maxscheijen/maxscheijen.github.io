---
id: html-form-and-json-endpoints-fastapi
aliases: []
title: HTML Form and JSON for single endpoint in FastAPI
date: 2024-10-05
---

A common challenge when building a FastAPI application is handling different content types, like `JSON` and `Form` data. Any many cases your API needs to support both formats, especially when dealing with web forms and modern front-end applications that often send json payloads.

This post demonstrates how to build a FastAPI application that handles and validates both JSON and from data inputs on a **single endpoint**.

## The Challenge: Handling Different Content Types

By default, web browsers use `application/x-www-form-urlencoded` or `multipart/form-data` content types when submitting forms. On the other hand, APIs and modern front-ends typically send `application/json`. Our goal is to handle both efficiently and return appropriate responses.

To achieve this I'll:

- Use dependency injection in FastAPI to dynamically process the request type (form or json).
- Validate the incoming data using Pydantic.
- Handle errors gracefully, including invalid content types and validation errors.

## Code Breakdown

Let’s dive into each part of this code to understand how it works.

### Validation

We define a Pydantic model `User` to enforce structure and validation on the incoming data. In this case, it expects a single field: `name`. This model will automatically validate the incoming data whether it comes from a form or a json body.

You can extend this model by adding more fields as needed.

```python
from pydantic import BaseModel


class User(BaseModel):
    name: str
```

### Request Type

Below, I use the `request_type` function to dynamically handle both `JSON` and `Form` content types. The `Request` object is used to inspect the content type of the incoming request:

- If the content type is `application/x-www-form-urlencoded`, we treat it as form data and convert it to a dictionary using `await request.form()`.
- If it’s `application/json`, we parse it into a dictionary using `await request.json()`.
- If the content type is unsupported, we raise a `ValueError`.

Later on the `request_type` function is injected into the route via FastAPI’s `Depends` feature.

```python
from fastapi import Request


async def request_type(request: Request) -> dict:
    # Get content-type (json or form) from request
    content_type = request.headers.get("Content-Type")

    # Convert form to dict
    if content_type == "application/x-www-form-urlencoded":
        form_data = await request.form()
        data = dict(form_data)

    # Convert json to dict
    elif content_type == "application/json":
        data = await request.json()

    # Handle unsupported content types
    else:
        raise ValueError(f"Unsupported content type: {content_type}")

    return data
```

### Handle Request in Route

This is the main route for handling incoming requests. The `data` returned by the `request_type` function is validated against the `User` model. If the validation succeeds, it means that the request body (whether JSON or form) is valid and contains the correct structure.

When the request’s content type is form data (`application/x-www-form-urlencoded`), we return a simple HTML response. This can of course be extended to return more complex HTML content based on your needs.

For JSON requests, the validated data is returned as the response.

```python
import json

from fastapi import Depends, FastAPI
from fastapi.responses import JSONResponse, HTMLResponse


app = FastAPI()


@app.post("/")
async def root(request: Request, data: dict = Depends(request_type)):
    try:
        # Validate data using pydantic model
        user = User(**data)

        # content-type is form, return HTML response
        if request.headers.get("Content-Type") == "application/x-www-form-urlencoded":
            return HTMLResponse(f"<p>Hello, {user.name}!</p>")

        # content-type not form return dict/json
        return user

    except ValidationError as e:
        # Exception handling when form or json data is invalid
        return JSONResponse(
            content=json.loads(e.json()),
            status_code=400,
        )

    except ValueError as e:
        # Handle unsupported content type
        return JSONResponse(
            content={"error": str(e)},
            status_code=415,  # Unsupported Media Type
        )
```

If the data doesn’t meet the Pydantic model’s validation rules, a `ValidationError` is raised, and a `400 Bad Request` error with the detailed validation errors is returned as a JSON response.

When the content type is unsupported, we catch the `ValueError` and return a `415 Unsupported Media Type` error with a descriptive message.

## Testing

We can now make a request using a json body or a form to the same endpoint

```sh
curl --location 'http://127.0.0.1:8000 ' \
--header 'Content-Type: application/json' \
--data '{
    "name": "John Doe"
}'
```

This will return the following json response.

```json
{
    "name": "John Doe"
}
```

We can also make a request using a form

```sh
curl --location 'http://127.0.0.1:8000 ' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'name=John Doe'
```

This results in a HTML string

```html
<p>Hello, John Doe!</p>
```

When incorrect fields are specified this will result in a nice pydantic validation error response.

```sh
curl --location 'http://127.0.0.1:8000 ' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'user=John Doe'
```
The incorrect field `user` is used this will result in the following response

```json
[
    {
        "type": "missing",
        "loc": [
            "name"
        ],
        "msg": "Field required",
        "input": {
            "user": "John Doe"
        },
        "url": "https://errors.pydantic.dev/2.6/v/missing"
    }
]
```

## Benefits

This pattern allows for seamlessly processing both form and JSON data, making our API more flexible.

It also leverages Pydantic’s integration with FastAPI ensures that incoming json or form data is validated automatically based on the schema we define.

Unsupported content types and validation errors are handled gracefully, providing clear error messages to the client.

By using dependency injection (`Depends`), the code remains modular. The logic for handling and converting request data is abstracted away, making the route handler simpler and more readable.

