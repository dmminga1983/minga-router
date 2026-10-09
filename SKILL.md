---
name: "minga-router"
description: "Before delegating work to a subagent (Task/Agent), decides which model (Haiku/Sonnet/Opus), effort level, and verification depth to use — based on the task's complexity and risk. Over time, refines its own thresholds from a log of actual outcomes."
---

# Model Router (minga-router)

When work gets split up and handed to subagents (via Agent/Task), don't send
everything through the same model on max settings "just in case" — that's
expensive and doesn't make the result any more reliable. And don't blindly
cut costs by picking a model by hand with no check on the result — that's
cheap, but unreliable: something will eventually come back wrong, and you
won't know it.

Before every Agent/Task call, rate the piece of work along two axes and
assign a model, effort level, and verification method using the table below.

## 1. Rate complexity and risk

**Complexity** — how much actual reasoning is required: finding and reading
a file isn't complexity; designing an architecture, tracking down a
non-obvious bug, or reconciling conflicting requirements is.

**Risk** — what happens if the result turns out wrong: does it break
production, cost money, affect a legally binding document, compromise
security, or trigger an irreversible action — or does it just need to be
regenerated.

## 2. Routing table (baseline — see section 5 on refining it)

| Complexity | Risk | Model | Effort | Verification |
|---|---|---|---|---|
| low | low | Haiku | low / default | not needed (result is self-evident — a typo, formatting, a simple lookup) |
| medium | medium | Sonnet | medium | quick pass — result matches the request, nothing obviously broken |
| high | high | Opus (or Sonnet at high effort if Opus isn't available) | high | full review — line-by-line against every requirement before handing it off further |

If you're torn between two rows, pick the more expensive one: underestimating risk usually costs more than a few extra cents in tokens.

## 3. Escalating on a failed check

If a cheap attempt (Haiku, or Sonnet at low effort) fails verification —
don't give up and don't ship it as-is: bump it up a step (Sonnet → Opus) and
retry. This is the "senior engineer" move: cheap first, verify, didn't pass
→ go up a tier. Never work like the "mid-level" failure mode either — never
burn an expensive call without verifying its result afterward. That's
exactly how money gets spent with no guarantee of a correct result.

This rule is what guarantees a correct final result regardless of how well
"trained" the router currently is — the savings come from picking a cheaper
starting point, never from weakening the check.

## 4. Important caveat

This does not switch the model of the CURRENT session — the Claude you're
talking to right now is already running on whatever model was chosen when
the session started. Routing only applies to work this Claude delegates
further via Agent/Task: if no delegation happens in the conversation, the
router has nothing to route.

## 5. Log and self-improvement

This isn't neural-network training (there are no weights here) — the
practical equivalent is: keep a log of actual outcomes and periodically
refine the table's thresholds from it, proposing changes to the user for
confirmation rather than silently rewriting the rules yourself.

**After every verification** (whether it passed or needed escalation), add
one line to `.minga-router-log.md` at the root of the current project (if
there's no suitable project, just skip this step — don't create the file
for its own sake):
```
- 2026-10-09: [task type] haiku→failed→sonnet passed; reason: ...
- 2026-10-09: [task type] sonnet passed on the first try
```

**Once enough entries have built up** (rule of thumb: 15–20 new ones since
the last review) **and a clear pattern shows up**, for example:
- the same task type regularly fails the cheap attempt and always needs
  escalation — the starting tier was picked wrong for it, it's worth
  starting one tier higher right away;
- a task type where the expensive model repeatedly passes verification
  "with room to spare" (simple result, trivial to check) — it's worth
  carefully trying a cheaper tier for it, without removing the check itself;

— **propose the user an updated version of this same skill** (through this
session's skill-saving mechanism) with the exception added to the table for
that task type and a short justification (how many observations, what
pattern). Never change the routing rules silently — the user needs to see
and confirm the change, like any other skill update.
