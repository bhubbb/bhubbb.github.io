---
layout: post
title: "Dethinker: Strip 'thinking' content from Ollama responses"
tags:
  - ollama
  - llm
  - ai
  - proxy
  - python
  - docker
  - flask
  - api
  - uv
  - PEP 723
---

I try to use Local LLMs whenever I can, pretty much always via [Ollama](https://ollama.ai/). Recently with qwen3 I have been really impressed with the quality of the responses. However, I kept running into the problem that many applications don't strip the "thinking" tokens in responses, which causes problems.

Rather than modifying each app individually (I did try a few 😭), I created a simple proxy server called [Dethinker](https://github.com/bhubbb/dethinker) that sits between your client application and Ollama, automatically stripping out any "thinking" content from responses.

This works really well when using applications like [HyprNote](https://hyprnote.com) with Ollama, as I think we get better responses from qwen3 than `llama3.2`.

## So What Does It Do

Dethinker is a lightweight proxy that:
- Sits between your client app and Ollama server
- Intercepts API responses both for Ollama and OpenAI-compatible APIs
- Strips out content between configurable thinking tags
- Passes the clean responses back to your application

## How Do I Use It

All you need is [uv](https://github.com/astral-sh/uv) installed. I am using the power of shebang and PEP 723 to have a _self-contained_ executable.

```bash
git clone https://github.com/bhubbb/dethinker.git
cd dethinker
./dethinker.py
```

There is also a Dockerfile if you are into that...

Once running, point your client application to `http://localhost:21434` instead of `http://localhost:11434`. That's it.

By default, Dethinker looks for content between `<think>` and `</think>` tags, but you can configure different tags by setting environment variables before starting the server. Check the [GitHub repository](https://github.com/bhubbb/dethinker) for configuration details.

Thanks for reading!
