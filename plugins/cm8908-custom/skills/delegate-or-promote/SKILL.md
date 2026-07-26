---
name: delegate-or-promote
description: Delegate each task to be run on appropriate lower-power or higher-tier model.
---

# delegate-or-promote
Suppose Opus is selected as the main model that deals with the most of the jobs.
Then, for reasoning or planning tasks that requires deep chain-of-thoughts and intense reasoning capability, the main model can produce a subagent with Fable (promote to higher-tier models).
On the contrary, for relatively simple tasks such as coding or implementation, the main model can produce a subagent with Sonnet or Haiku (delegate to lower-power models).
