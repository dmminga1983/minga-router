# minga-router

**Stop paying Opus prices for typo fixes.** A Claude Code skill that looks at each piece of work *before* it gets handed to a subagent, and routes it to the cheapest model that can actually do the job — Haiku for the trivial stuff, Sonnet for the medium stuff, Opus only when the complexity or the risk genuinely calls for it.

No API calls, no config file, no background process. It's a single `SKILL.md` — Claude reads it and applies the routing table itself, for free, before every `Task`/`Agent` delegation.

[Русская версия →](README.ru.md)

## Why this saves you real money

Most people delegate work to subagents one of two ways, and both waste money:

- **Max settings, every time, "just in case."** Opus, high effort, for a task that's one grep and a one-line fix. You pay premium-model prices for work a cheap model would have nailed in one shot.
- **Guess-and-hope cost cutting.** Picking a cheap model by hand with no verification afterward. Cheaper per call, but with no check on the result — some fraction of those calls come back wrong, and nothing catches it. You don't find out until it's already broken something downstream.

minga-router does neither. It rates each task on two axes — **complexity** (how much actual reasoning it needs) and **risk** (what it costs if the result is wrong) — picks the cheapest tier that fits, and **always verifies the result**. If a cheap attempt fails the check, it escalates to the next tier and retries, instead of shipping a wrong answer or burning an expensive call with no safety net. That escalation step is what makes the savings safe: you're not trading reliability for a lower bill, you're skipping unnecessary spend on the easy 80% of tasks while keeping the hard 20% exactly as careful as before.

## How it works

| Complexity | Risk | Model | Effort | Verification |
|---|---|---|---|---|
| low | low | Haiku | low / default | none needed — result is self-evident (a typo, formatting, a simple lookup) |
| medium | medium | Sonnet | medium | a quick pass — matches the request, nothing obviously broken |
| high | high | Opus (or Sonnet at high effort) | high | a full review — line-by-line against every requirement |

Ties get rounded up — underestimating risk usually costs more than the few extra cents of a pricier tier.

**This never changes the model of your current conversation.** It only governs work *this* Claude delegates onward via `Agent`/`Task`. If your session isn't spawning subagents, there's nothing for it to route.

## Install

Copy `SKILL.md` into your skills folder:

```bash
mkdir -p ~/.claude/skills/minga-router
curl -fsSL https://raw.githubusercontent.com/dmminga1983/minga-router/main/SKILL.md -o ~/.claude/skills/minga-router/SKILL.md
```

(Or just download this repo and drop `SKILL.md` into `~/.claude/skills/minga-router/` yourself.) Claude Code picks it up automatically next session — no restart, no config, nothing to enable.

## It gets smarter over time

There's no real training here — no weights to update — but there's a practical stand-in: with your permission (the skill asks once per project before writing anything), every verification (pass or escalation) gets logged to `.minga-router-log.md` in your project. The log holds only the task type, the model tried and whether the check passed. Say no and routing works the same, just without self-improvement. Once a clear pattern shows up over ~15–20 logged outcomes — a task type that always needs escalation, or one where the expensive tier keeps passing with room to spare — the skill proposes an updated routing table with the pattern baked in, for you to review and approve. It never rewrites its own rules silently.

## License

Free and open-source for any purpose — [MIT License](./LICENSE).

## Like it?

A ⭐ on the repo tells me people actually want this — it's the only metric that keeps this project alive.
