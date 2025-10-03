---
title: refire - A lightweight retrier
date: 2025-09-15
---

Failures happen. APIs timeout, flaky network calls fail, and sometimes random conditions causes your code to misbehave. Instead of writing repetitive retry logic in every function, wouldn’t it be better to have a clean, reusable decorator that takes care of retries for you? 

That’s exactly why I built [refire](https://github.com/maxscheijen/refire), a simple (no dependencies) yet powerful Python package that helps you retry functions on failure with configurable retry policies, exponential backoff, and jitter.


## Why refire?

When dealing with unreliable resources like APIs, databases, or external services, retrying operations is often the simplest way to build resilience. But manually implementing retries quickly leads to messy, repetitive code.

refire makes retrying clean and declarative:

- Retry on specific exceptions
- Control number of attempts (or retry indefinitely)
- Apply exponential backoff
- Add jitter to prevent retry storms
- Cap retries with a maximum delay
- Log retry attempts automatically

## Installation

```bash
pip install refire
# or 
uv add refire
```

Requires Python **3.10+**.

## Quick Start

Here’s a simple example of using `refire`:

```python
import random
from refire import refire


@refire(tries=5, delay=1, backoff=2, jitter=(0, 1))
def flaky_function():
    if random.random() < 0.7:
        raise ValueError("Unlucky!")
    return "Success!"


print(flaky_function())
```

In this example:

- The function will retry **up to 5 times**.
- The first retry waits **1 second**, then doubles each time (backoff=2).
- A random jitter between **0 and 1 second** is added to the delay.
- If all retries fail, the last exception is re-raised.

So, you might see log output like:

```text
WARNING:root:Caught ValueError: Unlucky!. Retrying in 1.00s (remaining=4)
WARNING:root:Caught ValueError: Unlucky!. Retrying in 2.34s (remaining=3)
WARNING:root:Caught ValueError: Unlucky!. Retrying in 4.89s (remaining=2)
...
```

## Customizing Retry Policies

You have full control over retry behavior:

**Retry on specific exceptions:**

```python
@refire(exceptions=(ConnectionError, TimeoutError), tries=10)
def fetch_data():
    ...
```

**Infinite retries until success:**

```python
@refire(tries=None, delay=2)
def always_retry():
    ...
```

**Cap the maximum delay**

```python
@refire(delay=1, backoff=2, max_delay=30)
def capped_backoff():
    ...
```

**Integrate with your own logger:**

```python
import logging

logger = logging.getLogger("myapp")

@refire(logger=logger, log_level=logging.INFO)
def with_logging():
    ...
```

## Retrying on Custom Exceptions

You can also define your own exceptions and configure refire to retry only when they occur:

```python
class TemporaryAPIError(Exception):
    """Raised when the API fails temporarily."""


class CriticalAPIError(Exception):
    """Raised when the API fails permanently (don’t retry)."""


@refire(exceptions=TemporaryAPIError, tries=3, delay=2)
def fetch_from_api():
    import random
    if random.random() < 0.8:
        raise TemporaryAPIError("Temporary failure, please retry.")
    elif random.random() < 0.9:
        raise CriticalAPIError("Critical failure, do not retry!")
    return "Fetched data!"
```

- If a `TemporaryAPIError` is raised, the function will retry up to **3 times** with a **2s delay**.
- If a `CriticalAPIError` is raised, it will **not retry** and immediately propagate the exception.

This makes refire flexible: you can fine-tune retry behavior depending on whether an error is transient or permanent.

## When to Use (and When Not To)

Use refire when:

- Making unreliable API calls
- Accessing flaky external services
- Running operations that often succeed on retry

Don’t use refire when:

- Failures are deterministic and retries won’t help
- Retrying could cause unintended side effects (e.g., duplicate financial transactions)

## Roadmap

The current version (0.1.0) is lightweight and focused. Future plans include:

- Support for async functions
- Custom retry policies (e.g., Fibonacci backoff)
- Context-aware retries (timeouts, cancellation)

## Try It Out

The package is open-source under the **Apache-2.0 license** and available here:

GitHub – [maxscheijen/refire](https://github.com/maxscheijen/refire)

If you want a **clean, reusable way to add resilience** to your Python code, give refire a try — and let me know what you think!
