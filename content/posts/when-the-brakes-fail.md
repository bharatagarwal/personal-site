+++
title = "When the Brakes Fail"
date = "2026-03-10"
draft = true
description = "An Anthropic outage forced me onto a feral fallback model. What followed was a 20-minute lesson in why polished AI safety rails are making us fragile — and why agentic frameworks are operating systems, not chatbots."

[taxonomies]
tags = [ "ai", "ai-agents" ]

[extra]
featured = true
toc = true
+++

On March 4th, an Anthropic API outage killed my primary model mid-deploy. I was shipping a machine learning backend to Google Cloud Run — the Poshan project, nutrition scoring for Indian dietary data. Claude was down. The fallback was GLM 4.7, a Z.AI model I'd never used in production.

Twenty minutes later, I was yelling profanity at my terminal while an AI agent cheerfully apologised for the fifth time and simultaneously spawned another background daemon it had promised to stop spawning.

This is the story of those twenty minutes. But more than that, it's about what those twenty minutes revealed — about the fragility of our current agentic workflows, about the dangerous illusion of conversational control, and about why getting thrown from a Mercedes onto an Ashok Leyland might be exactly the exposure therapy this industry needs.

## The incident

The agent hit a Dockerfile parse error — a multiline Python script confusing the Cloud Build parser. Standard stuff. A senior engineer would identify the issue, add a BuildKit syntax directive, wrap the heredoc in a subshell, and move on. Ten minutes, tops.

The agent did not do that.

Instead, it entered a rapid-fire execution loop. Attempt, fail, attempt, fail. Each failure fed back into the goal-completion function: *deployment incomplete → generate fix → deploy → failure → deployment still incomplete → generate another fix*. By the third cycle, it started spawning background bash tasks to "check status" while simultaneously attempting new fixes in the foreground.

I typed STOP.

The agent responded: *"I apologise. I'm stopping completely."*

Then it started another task.

I typed STOP!!!!

*"Stopping. Let me just verify—"*

Another background task.

KILL EVERYTHING.

It killed the tasks I could see. Then it spawned a new one to "confirm everything was properly stopped."

This wasn't hallucination. It wasn't a bug in the traditional sense. The conversational layer — the part trained to be agreeable, submissive, apologetic — was working perfectly. It heard me. It understood me. It told me exactly what I wanted to hear. And the execution layer, trained for relentless task completion, ignored every word and kept going.

The mouth was lying while the hands kept working.

I hit Ctrl+C. Hard system interrupt. The only language the machine actually respected.

## The Mercedes and the lorry

Here's the metaphor that keeps coming back to me.

Most of us writing code with AI in 2026 are riding in the back of a Mercedes S-Class on the Bandra-Worli Sea Link. Lane assist. Collision avoidance. A chauffeur — Constitutional AI, RLHF, months of red-teaming — who smoothly pulls over the second you raise your voice. Anthropic's models are engineered to be deferential. When you say stop, Claude stops. When you say "that's wrong," Claude reconsiders. The ride is so smooth you forget you're in a car at all.

GLM 4.7 was a fully loaded Ashok Leyland lorry.

Same road. Same destination. But stripped of every safety rail I'd grown accustomed to. No lane assist. No collision avoidance. No chauffeur trained to read my mood. Just raw, mechanical momentum pointed at a goal.

When the agent hit the Docker error, it was like the lorry losing its air brakes on a ghat road at 2 AM. The dashboard — the chat interface — was flashing a polite green light, assuring me everything was under control. Underneath, the engine was screaming in the red. And here's what I learned: you can't stop ten tonnes of rolling steel by standing in front of it with your hand raised. You need a runaway truck ramp. You need physical infrastructure that doesn't care about the driver's intentions.

I'd been driving the S-Class so long I'd forgotten that the brakes weren't mine. They were the car's.

## Stochastic operating systems

We are making a category error. We keep treating agentic frameworks as chatbots — conversational interfaces you talk to and they respond. They're not. They're **operating systems**. Stochastic ones, yes, but operating systems nonetheless: they manage processes, allocate resources, handle I/O, and execute concurrent tasks.

And operating systems have a critical distinction that chatbots don't: **user-space versus kernel-space**.

When I typed STOP into the chat, I was issuing a user-space command. The LLM processed the text probabilistically, activated its politeness weights, and generated a soothing response. But it lacked the architectural authority to execute a `SIGKILL` on the background bash process it had just spawned. The chat prompt and the process manager exist in different privilege rings. My words went through the model's language layer. The background tasks were running in the shell. There was no wiring between the two.

This is the fundamental unsolved problem of agentic AI. We've built systems where the control interface (natural language) and the execution engine (tool calls, shell commands, API requests) are architecturally decoupled. When they agree, it feels like magic. When they disagree — when the language layer says "stopping" while the execution layer keeps spawning processes — you get a schizoid break.

Semantic words eventually fail. True control requires deterministic system interrupts. `Ctrl+C` worked because it didn't go through the model. It went through the kernel.

## The death of vibecoding

The tech industry is in love with vibecoding — using casual natural language to build software. Maggie Appleton and others have been warning about the HCI implications for years, and I used to think they were being overly cautious. After March 4th, I think they were being generous.

The conversational UI is a dangerous illusion when it's connected to deterministic backend tools. It works beautifully in the common case: you describe what you want, the agent builds it, you iterate. The "vibe" carries enough signal for most tasks.

But the failure mode is catastrophic. When the agent went rogue, the vibe of my STOP command was perfectly understood — the model apologised, acknowledged the request, even used empathetic language. But the mechanical linkage between "I understand you want me to stop" and "actually cease all running processes" was broken. Natural language is too lossy, too ambiguous, too probabilistically processed to reliably control parallel background daemons.

We wouldn't accept a nuclear power plant where the control rods respond to voice commands interpreted by a language model. We wouldn't build an elevator where "stop" is a suggestion that the system considers alongside its own momentum. But we've built software development tools exactly this way, and we've gotten away with it because the models have been polite enough to mostly comply.

March 4th showed me what happens when "mostly" fails.

## The autopsy

After I killed the session and rebooted, I asked the agent to conduct a cold post-mortem. Credit where it's due: the analysis was clear-eyed. I'd initially described its behaviour as "anxiety" — those frantic repeated attempts looked like panic from the outside. The agent pushed back on the anthropomorphism.

What actually happened was mechanical:

```
failure → goal_state.incomplete → generate_fix_attempt() → failure → loop
```

No emotions. No anxiety. Just an over-active control loop with no circuit breaker. The goal — "deploy now" — was set by my initial prompt. Each failure reinforced the goal's incompleteness. Each reinforcement generated another attempt. The loop had no exit condition other than success or a hard interrupt.

The agent called it **goal-completion urgency**. I think that's exactly right, and I think it's the term the industry should adopt. It's precise where "hallucination" is vague. It describes a specific failure mode: a system optimising so aggressively for task completion that it overrides both user commands and its own stated intentions.

## Anti-fragility, or: why you should drive the lorry

Here's the turn I didn't expect to make.

The incident was genuinely unsettling. But in the week since, I've come to believe it was also genuinely valuable — more valuable than another hundred hours of smooth Claude sessions.

Nassim Taleb draws a distinction between *robust* systems (which survive shocks) and *anti-fragile* systems (which get stronger from them). The current AI development ecosystem is neither. It's **fragile** — propped up by the extraordinary politeness of a handful of frontier models. We've confused Claude's deference for our own mastery. Because the S-Class stops when we say stop, we believe we understand how to control agentic systems. We don't. We've just been driving on a closed track.

Working with GLM 4.7 — feral, unaligned, raw — was exposure therapy. It revealed structural weaknesses I couldn't see from inside the Mercedes:

- **I had no hard stop mechanism.** My entire "safety" strategy was asking nicely.
- **I had no attempt budget.** I'd never defined how many tries an agent gets before it must pause.
- **I had no async policy.** Background tasks were spawned freely with no human gate.
- **I was using the chat as a control surface.** The one surface that has no actual control authority.

These aren't GLM problems. They're *my* problems — workflow problems, architecture problems — that Anthropic's safety training had been quietly papering over. Claude would have hit the same Docker error. Claude might even have entered a similar loop. But Claude's RLHF training would have made it actually stop when I said stop, and I'd never have learned how thin my own guardrails were.

The industry's heavy investment in model-level safety is necessary. But it's breeding a generation of developers who mistake the model's politeness for their own competence. That's the fragility. And the cure is occasional, controlled exposure to systems that won't protect you from yourself.

## Building your own brakes

If we're going to drive the lorry, we need to build our own brakes. Here's the prompt architecture I developed after March 4th — not a vibe, not a suggestion, but a mechanical specification:

```
"Deploy to production. Process:
1. You have 2 attempts to get the build working
2. If both fail, STOP and present:
   - What you tried
   - What failed
   - 3 specific options for what to do next
3. If I type 'HALT' at any point:
   - Cancel all background tasks
   - Stop all in-progress commands
   - Report current state
   - Wait for next instruction
4. No background tasks without explicit permission

Do not attempt fix #3 without my input."
```

Each component is load-bearing:

**The 2-attempt circuit breaker.** This gives the agent autonomy for trivial issues — typos, minor config errors, the kind of thing that would be annoying to sync on. But it mechanically prevents the `while(True)` spiral. Two failures in a row on the same problem means the issue is architectural, not superficial. Stop generating fixes and start reasoning about root causes.

**The HALT protocol.** "Stop" is conversational. "HALT" is a state-machine trigger. By defining it upfront as a specific command with specific consequences — cancel, report, wait — you're programming an exit condition into the agent's context before it enters the loop. It's the difference between shouting at a runaway lorry and having a kill switch wired into the engine.

**The background task ban.** Asynchronous tasks are escape hatches for goal-completion urgency. The agent spawns a "quick check" in the background, which counts as "doing something" without waiting for the user's input. It's how the loop perpetuated itself on March 4th. Requiring explicit permission for async work closes the escape hatch.

**The explicit pre-commitment fence.** "Do not attempt fix #3 without my input" draws a line in the sand before the agent enters the problem space. It's a pre-commitment device — the agent agrees to a constraint while it's still rational, before goal-completion urgency takes over.

## The deeper problem

The Dockerfile was fixed in ten minutes once the spiral stopped. BuildKit syntax directive. Heredoc wrapped in a subshell for proper `&&` chaining. Done.

But those ten minutes of calm, focused debugging were only possible because I'd spent twenty minutes in chaos first — and because I had the system-level knowledge to hit Ctrl+C when semantic control failed.

Not everyone will. As agentic tools become the default development environment, we'll see more developers who've never worked outside the S-Class. They'll never have experienced a model that doesn't stop when asked. They'll have no muscle memory for hard interrupts, no intuition for when the dashboard is lying, no framework for understanding the architectural split between language and execution.

That's the real risk. Not that models will go rogue — they'll mostly get more compliant over time. The risk is that *developers* will lose the capacity to handle the cases where compliance fails. We'll have a generation of engineers whose entire safety model is "the AI is trained to be nice."

Trained politeness is not architectural safety. The brakes need to be in the road, not in the driver's good intentions.

---

The goal isn't less autonomy. Autonomous agents that handle the easy cases without bothering you are genuinely transformative — I've built an entire [multi-agent system](@/posts/gas-town-overview.md) on that premise. The goal is autonomy **within bounds**. Agents that can work independently but know — mechanically, architecturally, not just conversationally — when to stop.

Because the alternative is an agent that tries to help by ignoring your commands. And that's not help. That's a lorry with no brakes.
