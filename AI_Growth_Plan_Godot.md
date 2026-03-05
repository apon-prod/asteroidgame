# 🚀 AI Growth Plan — Godot Edition

> Build a 2D space mining game. Learn agentic AI the way that actually sticks.

*Sequential agents · Tool integrations · Improvement loops · Technical depth*

---

## Why a Game is the Right Learning Environment

Most AI growth plans fail because the exercises are abstract. A game solves this: every feature is a clear pass/fail test, systems depend on each other in ways that force real orchestration decisions, and the feedback loop is instant — it either works or it doesn't.

Your space mining game has natural complexity layers that map directly onto your growth areas:

| Game Feature | AI Challenge | Growth Area |
|---|---|---|
| Spaceship movement & asteroids | Single agent, focused scope | Foundation & workflow habits |
| Mining + resource system | Output feeds into upgrade system — first chain | Sequential agent orchestration |
| Upgrade tree | Agent B depends on Agent A's resource schema | Error propagation & validation |
| Enemy AI & weapons | Multi-system coordination, GitHub tracking | Tool integration + orchestration |
| Polish & juice | Improvement loops, parameter tuning | Technical depth & evals |

---

## How to Work With Claude Code on Godot

Before starting each phase, set up these three files at the root of your Godot project. They are what turn a series of evening sessions into a compounding system rather than a series of one-offs.

### CLAUDE.md
Your persistent project-level context — Claude Code reads it at the start of every session. Include:
- Game name, genre, and one-sentence description
- Tech: Godot 4, GDScript, 2D
- Current feature status (what's done, what's in progress)
- Code conventions you want followed consistently
- Known issues or constraints

### PROJECT-STATUS.md
You already use this pattern. Keep it here too. Update it after every session with what was completed, any open questions, and what the next agent should pick up. This is your continuity layer between sessions.

### SKILLS-REGISTRY.md
Log every Claude skill or workflow you build for this project. One line per entry: what it does, when it breaks, what version it's on. This is your improvement loop in practice.

> **Treat these three files as sacred.** They are what turn a series of evening sessions into a compounding system rather than a series of one-offs.

---

## 🛸 Phase 1 — Get Something Moving
*Foundation — single agent, build the habit*

**Goal:** A working scene with a spaceship that moves, and asteroids that float. Nothing fancy — the point is establishing your working pattern with Claude Code on a Godot project.

| Challenge | What You Ship | AI Skill Practiced |
|---|---|---|
| Set up the Godot project + CLAUDE.md | A Godot 4 project with your three MD files in place and a first commit to GitHub | Workflow foundation + GitHub integration |
| Spaceship movement | Player scene with keyboard-controlled ship, screen wrap, and basic feel | Single agent, focused prompting |
| Asteroid field | Asteroids that spawn, drift, and wrap. Varying sizes. | Parallel agents — ship and asteroids are independent |
| Collision detection | Ship detects asteroid collision. Game over state. | Prompt chaining — build on what exists without breaking it |

### The habit to build in Phase 1

After each challenge session, spend 5 minutes updating PROJECT-STATUS.md and adding one line to SKILLS-REGISTRY.md. This feels unnecessary when you're in flow — do it anyway. It's the improvement loop in its smallest form.

> **Phase 1 AI focus:** You're not doing anything novel here orchestration-wise. The goal is to establish clean working patterns — good CLAUDE.md context, consistent PROJECT-STATUS.md updates, and learning how much context Claude Code needs to not break existing code when adding new features.

---

## ⛏️ Phase 2 — Mining & The First Agent Chain
*Sequential orchestration — this is where it gets interesting*

**Goal:** Build the mining system and wire it to an upgrade system using a sequential agent chain — where Agent A's output (resource schema) is Agent B's input (upgrade tree). This is the most important phase for your AI growth.

| Challenge | What You Ship | AI Skill Practiced |
|---|---|---|
| Asteroid mining mechanic | Ship can shoot asteroids, they break apart and drop resources (ore types) | Single agent extending existing codebase |
| Resource manager system | A resource manager that tracks ore, handles pickup, and exposes a clean data schema | Designing agent output intentionally — Agent A in your first chain |
| Upgrade tree — Agent B | Feed the resource schema into a second agent to build an upgrade system that reads from it. No manual copy-paste — output flows to input. | First sequential agent chain |
| Validation step | Add a check: before the upgrade system builds, validate the resource schema is complete. What happens if it isn't? | Error propagation between agents |

### The chain pattern to follow

When you build the resource manager, end your prompt with: *"Output a clean JSON schema of all resource types and their properties. This will be consumed by a second agent."* Then start your upgrade agent prompt with that schema explicitly in context. That handoff is the skill.

> **Phase 2 AI focus:** Sequential agents fail in two ways — the upstream output is ambiguous, or the downstream agent makes assumptions about it. You'll likely hit both. When you do, diagnose before rewriting: was it the instruction, the input, or your expectation that was wrong?

---

## 👾 Phase 3 — Enemies, Weapons & Tool Integration
*Connect your toolchain — GitHub and Slack enter the picture*

**Goal:** Add enemy ships and a weapon system, but do it while wiring GitHub and Slack into your Claude Code workflow. Features get tracked as issues, completed work triggers a Slack update — automatically.

| Challenge | What You Ship | AI Skill Practiced |
|---|---|---|
| Connect GitHub MCP to Claude Code | Set up GitHub MCP server. Create your first issue from Claude Code CLI. No browser. | GitHub tool integration |
| Enemy ship behaviour | Basic enemy that moves toward the player, with simple pathfinding. Tracked as a GitHub issue opened and closed via Claude Code. | MCP in a real workflow |
| Weapon system | Player can fire, projectiles have types, enemies have health. Each feature = a GitHub issue. | Agentic issue management |
| Slack standup bot | At end of each dev session, a Claude Code workflow posts a one-line update to a Slack channel: what shipped, what's next. | Slack MCP + sequential mini-chain |

### Why bother with GitHub issues for a personal project?

Because you're not just building a game — you're proving out an agentic development workflow. The game is the lab. When you can open, update, and close GitHub issues from Claude Code without touching a browser, that pattern transfers directly to client work with Jira (which you already do).

> **Phase 3 AI focus:** Notice how tool integrations change the feel of agentic work. When Claude Code can act on external systems (not just generate code), the gap between "AI assistant" and "AI agent" becomes concrete. That understanding is what you want to be able to explain to others.

---

## 🧠 Phase 4 — Polish, Depth & Understanding
*Close the technical gaps — RAG, parameters, improvement loops*

**Goal:** The game is playable. Now use it as a test bed to deliberately explore the technical concepts you identified as gaps — not abstractly, but in the context of something you've built.

| Challenge | What You Ship | AI Skill Practiced |
|---|---|---|
| Temperature experiment on game dialogue | Add enemy taunts or mining log flavour text. Run the generation prompt at temp 0.2 vs 0.9. Document what changes and what that tells you. | Model parameters — practical understanding |
| Build a RAG-lite pattern for game lore | Create a lore document for your game universe. Build a simple retrieval pattern — user query pulls relevant lore into context. Manual retrieval counts. | RAG — hands-on not theoretical |
| Write your RAG vs fine-tuning POV | One page: if a client asked you to make an AI feature domain-specific in a game, when would you use RAG vs fine-tuning? Get Claude to critique it. | Defensible technical advisory POV |
| Run a full improvement loop review | Audit your SKILLS-REGISTRY.md. What kept breaking? Rewrite the weakest skill using what you now know. Version it properly. | Improvement loop — closing the cycle |

> **Phase 4 AI focus:** The goal isn't to become an engineer. It's to reach the point where you can say "I've actually done this, not just read about it" when these topics come up in client conversations. That credibility is built through doing, even at small scale.

---

## Your Lightweight Improvement Loop

This applies to every phase. It takes less than 5 minutes per session and it's the single habit that separates people who get better from people who just get more experienced.

| # | When | Question to Ask | Where to Log It |
|---|---|---|---|
| 1 | Output underperforms | Was it the instruction, the input, or my expectation? | One sentence in SKILLS-REGISTRY.md before editing |
| 2 | Before rewriting a skill | Would the old version have done better on this input? | Keep the old version — append v2 |
| 3 | After a phase is done | What kept breaking across this phase? | A "lessons" section at bottom of SKILLS-REGISTRY.md |
| 4 | Monthly | What patterns do I see in what fails? | A running "known weaknesses" doc — becomes your next growth area |

---

## What 'Done' Looks Like

You'll know this plan has worked when you can honestly say all of the following:

- I have a playable 2D space game built almost entirely through agentic AI workflows
- I've built and debugged a sequential agent chain where output from one agent fed directly into another
- GitHub issues open and close from my CLI without touching a browser
- Slack gets automatic updates from my dev sessions
- I've run temperature experiments and can explain what the parameter actually does
- I can explain RAG vs fine-tuning from personal experience, not just reading
- My SKILLS-REGISTRY.md has version history and a lessons section
- I could walk a colleague through my Godot workflow and they'd get immediate value from it

> The game is the proof. When someone asks what you've been working on, you don't say "I've been studying AI." You say "I built a space mining game using agentic Claude Code workflows — here's what I learned about sequential agent chains in the process." That's the difference.

---

## Your First Session — Starter Prompt

Copy this into Claude Code to kick off Phase 1. Adapt it once you've set up your CLAUDE.md.

```
You are helping me build a 2D space mining game in Godot 4 using GDScript.

Game concept: The player pilots a spaceship, mines asteroids for resources,
uses resources to buy upgrades, and must defend against enemy ships.

Current status: Project just initialised. Nothing built yet.

Task for this session: Create the player scene with a spaceship that moves
using keyboard input (WASD or arrow keys), rotates to face movement direction,
and wraps around the screen edges. Keep the code clean and commented —
other agents will extend it.

After completing, summarise what was built, what files were created or
modified, and what the next logical step is. I will update PROJECT-STATUS.md
from your summary.
```

---

*Good luck. Ship the game. Learn the AI.*
