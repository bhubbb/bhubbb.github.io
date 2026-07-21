---
layout: post
title: "Fireworks AI Cache Optimization: The Session Affinity Gap"
tags:
  - fireworks
  - ai
  - llm
  - caching
  - opencode
  - plugin
---

I have an agent I built for a customer that runs on a full Anthropic stack. It works great, I am happy with it, and I am also very aware of what it costs.

That was what sent me looking at GLM-5.2 on Fireworks. If the capability was anywhere near what was being claimed, it looked like a much cheaper option than just staying on straight Anthropic pricing.

What made this annoying was caching. Anthropic caches so well that, if you are using large system prompts, a lot of tools, and repeated context, the real cost can feel a lot lower than the raw input pricing suggests. You stop thinking about it. I had. Other providers do not behave the same way, so this stopped being a simple model-pricing question pretty quickly.

## Where OpenCode came into it

We use OpenCode as one layer inside a larger agent system we built. Our own harness takes input from a few systems, does some orchestration itself, and runs OpenCode as the agent layer. The part I was testing here was the worker side of that setup.

When I started looking at Fireworks properly, I found two things that mattered:

- Fireworks prompt caching is automatic, but it only works within one replica
- if you want repeated requests to benefit from the same cached prefix on serverless, you need a stable routing hint like `user` or `x-session-affinity`

That is straight from the Fireworks prompt caching docs [[1]](#ref-1).

OpenCode was not setting that affinity hint natively for Fireworks, so I was not giving Fireworks what it needed to make the cache useful across repeated requests in a session.

## Why that matters

If you are trying to work out whether a provider is actually cheaper, you cannot just compare the posted token prices. You also need to know whether you are really hitting cache.

Anthropic had basically trained me not to think about that part too much. Fireworks made me think about it very quickly, and that is why I built the plugin. Not to change the model. Not to improve capability. Just to stop wasting money on prompt prefixes that should have been cached.

## The fix

The fix is tiny: set `x-session-affinity` on every chat request, keyed to the OpenCode session ID.

That is it.

```typescript
import type { Plugin } from "@opencode-ai/plugin"

export const SessionAffinityPlugin: Plugin = async () => {
  return {
    "chat.headers": async (input, output) => {
      try {
        output.headers["x-session-affinity"] = input.sessionID
      } catch {
        // Never let a header hiccup break the run.
      }
    },
  }
}
```

I kept it boring on purpose. It sets one header, does not branch on provider, and does not touch anything except the request headers.

OpenCode's `chat.headers` hook does receive session and provider context, so you could make this more provider-specific if you wanted to. If you want to go that way, start with the OpenCode plugin docs [[4]](#ref-4) and the current plugin type definitions in source [[5]](#ref-5). I did not need that extra logic for my setup, and I preferred the simpler version.

In my setup, sending the extra header to non-Fireworks providers did not cause problems.

## How to use it

Copy the full file above into `session-affinity.ts`, then drop it into OpenCode's plugin directory.

### Project-local

```bash
mkdir -p .opencode/plugins
cp session-affinity.ts .opencode/plugins/session-affinity.ts
```

### Global

```bash
mkdir -p ~/.config/opencode/plugins
cp session-affinity.ts ~/.config/opencode/plugins/session-affinity.ts
```

## How I would check it

The right way to check this is an A/B run.

Run the same workload once **without** the plugin, then again **with** it. Keep the prompts, tools, and general session shape as close as you can.

Then compare three things:

1. **Fireworks response metadata**
   - compare `fireworks-prompt-tokens` and `fireworks-cached-prompt-tokens` from Fireworks' response headers [[1]](#ref-1)
2. **OpenCode's own token and cache reporting in your setup**
   - if you are already surfacing cache-related token information in your harness or session output, compare that between the run without the plugin and the run with it
3. **Actual spend at the end of the run**
   - this is the bit that matters if you are trying to work out whether the provider is really cheaper

What you should expect is pretty simple: with the plugin on, repeated requests with stable prompt prefixes should show more cached prompt usage than the same run without it. Without the plugin, cache reuse is likely to be lower or noisier because requests are not being given that stable routing hint.

That still is not a perfect benchmark. Prompt stability, cache lifetime, backend state, and load all matter. But it is a much better test than staring at list pricing and guessing.

## One caveat

This does not guarantee cache hits.

You still need stable prompt prefixes, and Fireworks is still doing cache reuse based on what is already sitting on a backend at that point in time.

What this does is remove one obvious reason you would miss cache when you did not need to.

That was enough reason for me to make it.

If you are looking at Anthropic versus Fireworks in OpenCode, this is worth checking before you decide one is cheaper than the other.

## One last note on GLM-5.2

Separate from the caching work, GLM-5.2 itself has been promising for worker-agent use. In my setup it has been a high-performing model, especially on Fireworks, and it is materially cheaper than Sonnet.

I am not treating that as a formal benchmark yet. My harness is designed to push agents toward evidence-driven behavior, so I have more signal than a few casual prompts, but I still want to be careful about overclaiming. I also have not had anyone come back to me and say the agent has become materially dumber, which matters. The practical takeaway for me is simpler: GLM-5.2 is cheap enough, and good enough in this kind of worker-agent role, that it is worth taking seriously.

## References

<ol>
  <li id="ref-1"><a href="https://docs.fireworks.ai/guides/prompt-caching">Fireworks AI — Prompt caching</a></li>
  <li id="ref-2"><a href="https://docs.fireworks.ai/serverless/pricing">Fireworks AI — Serverless pricing (includes cached input pricing for GLM-5.2)</a></li>
  <li id="ref-3"><a href="https://docs.fireworks.ai/models/fireworks/glm-5p2">Fireworks AI — GLM-5.2 model page</a></li>
  <li id="ref-4"><a href="https://opencode.ai/docs/plugins/">OpenCode plugins docs</a></li>
  <li id="ref-5"><a href="https://github.com/anomalyco/opencode/blob/dev/packages/plugin/src/index.ts">OpenCode plugin type definitions (`chat.headers`, `sessionID`, `provider`, `output.headers`)</a></li>
</ol>