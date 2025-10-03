---
title: vibeflip - A truly random coin flip
date: 2025-08-30
---

I got inspired by [vibesort](https://github.com/abyesilyurt/vibesort). What a nice way to sort a array of numbers. 

Which made me think of what other thinks are hard. A computer creating randomness. Can we maybe leverage LLMs?

## vibeflip

So what is [vibeflip](https://github.com/maxscheijen/vibeflip)?

It's a python library that facilitates a truly random coin flip (`true` or `false`) using a LLM.


```python
from vibeflip import vibeflip

result = vibeflip()
print(result)  # True or False
```

```sh
pip install vibeflip
```

Set your OpenAI API key as an environment variable.

```sh
export OPENAI_API_KEY=your_api_key
```

/s

