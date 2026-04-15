# Ecosystem Watch

Projects adjacent to Starfire-AgentTeam that are worth tracking — for design
ideas to borrow, terminology collisions to be aware of, and to stay honest
about where our differentiation actually is.

## How to use this doc

- **Skim quarterly.** The agent-infra space moves fast; expect entries to be
  stale within ~3 months. When a project on this list ships something we
  should react to, add a line under "Signals to react to" for that entry
  and a short plan.
- **Add entries liberally.** Easier to prune than to miss.
- **One entry per project.** Keep each under ~200 words — link out, don't duplicate.

## Template

````markdown
### <Project> — `org/repo`

**Pitch:** one sentence in their words.

**Shape:** what it actually is (language, deployment target, one-vs-many-agents, etc.)

**Overlap with us:** where our designs touch.

**Differentiation:** why we're not the same product.

**Worth borrowing:** specific ideas we should study.

**Terminology collisions:** shared words that mean different things.

**Signals to react to:** what they might ship that would change our roadmap.

**Last reviewed:** YYYY-MM-DD · **Stars / activity:** <quick stat>
````

---

## Entries

### Holaboss — `holaboss-ai/holaboss-ai`

**Pitch:** "AI workspace desktop for business — build, run, and package AI
workspaces and workspace templates with a desktop app and portable runtime."

**Shape:** Electron desktop app + TypeScript runtime. **Single active agent
per workspace.** MIT-licensed OSS core with a hosted Holaboss backend for
some features (proposal ideation). macOS supported; Windows/Linux in progress.

**Overlap with us:** both call the unit of packaging a "workspace";
both ship a `skills/<id>/SKILL.md` convention; both have a plugin/app
marketplace; both treat long-lived context as important.

**Differentiation:** Holaboss is the **"AI employee"** shape — one agent
holding one role for months, with heroic effort spent on token-cost
discipline (compaction boundaries, `prompt_cache_profile`, stable vs
volatile prompt sections). We're the **"AI company"** shape — many agents
collaborating via A2A, visual org chart, multiple runtimes. No A2A, no
multi-agent coordination on their side.

**Worth borrowing:**
- Filesystem-as-memory: `memory/workspace/<id>/knowledge/{facts,procedures,blockers,reference}/` + scoped `preference/` and `identity/` namespaces. Clean model for durable memory that beats our current DB-only approach for inspectability.
- Compaction boundary artifact (summary + restoration order + preserved turn ids + request snapshot fingerprint) — if we ever add long-horizon single-agent mode, this is the reference design.
- Section-based prompt assembly with per-section cache fingerprints. Could reduce our Claude Code prompt cost.
- `workspace.yaml` rejects inline prompt bodies — forces prompts into `AGENTS.md`. We should do the same in `config.yaml` to keep runtime plans machine-readable.

**Terminology collisions:**
- "workspace" — theirs is a directory + agent state; ours is a Docker container running one agent in a team.
- "MEMORY.md" — theirs is the structured memory-service root; ours is the native file Claude Code / DeepAgents read.
- "skills/SKILL.md" — same filesystem convention, both inject into system prompt. Fully compatible in spirit.

**Signals to react to:**
- If they add A2A between workspaces → direct competitor; revisit differentiation.
- If they publish the compaction-boundary format as a spec → adopt.

**Last reviewed:** 2026-04-12 · **Stars / activity:** ~1.7k ⭐, pushed today

---

### Hermes Agent — `NousResearch/hermes-agent`

**Pitch:** "The self-improving AI agent built by Nous Research — creates
skills from experience, improves them during use, searches its own past
conversations, and builds a model of who you are across sessions."

**Shape:** Python-first agent framework with a TUI + multi-messenger
gateway (Telegram / Discord / Slack / WhatsApp / Signal / Email). Single
user, single continuous agent with a closed **learning loop**. Six
execution backends (local, Docker, SSH, Daytona, Singularity, Modal —
last two are serverless w/ hibernation). MIT, ~61k⭐ and climbing fast.

**Overlap with us:**
- "Skills" with filesystem convention — compatible with the
  [agentskills.io](https://agentskills.io) open standard they back.
- Subagent spawning for parallel work.
- Scheduled automations (natural-language cron).
- Model-agnostic (Nous Portal, OpenRouter, GLM, Kimi, MiniMax, OpenAI, …).

**Differentiation:** Hermes is the **"personal AI across every messenger"**
shape — one agent that knows *you* deeply and runs anywhere. We're the
**"team of agents behind a canvas"** shape — many roles collaborating on
shared work. Hermes has no visual canvas, no org hierarchy, no A2A between
workspaces.

**Worth borrowing:**
- **Closed learning loop**: autonomous skill creation after complex tasks,
  skills self-improve during use, agent-curated memory with periodic nudges
  to persist knowledge. This is a much stronger memory discipline than
  ours; the "nudge to persist" pattern in particular is cheap to implement.
- **FTS5 + LLM-summarization** for cross-session recall — cheap, no
  vector-store overhead, works great for the "did I tell you about X" case.
- **Honcho dialectic user modeling** (`plastic-labs/honcho`) for building
  a model of the user across sessions. Worth evaluating as a memory backend
  for Starfire's PM workspace specifically (the one role where knowing
  the CEO well matters most).
- **Daytona / Modal serverless backends** with hibernation — a great fit
  for our DevOps workspaces that only wake for scheduled audits. Could
  drop our idle compute cost meaningfully.
- **`hermes claw migrate`** command — gracefully import users from
  OpenClaw (the predecessor). Good pattern if we ever deprecate a runtime
  adapter.

**Terminology collisions:**
- "skills" — same direction as ours post-refactor (file-based, installable,
  runtime-agnostic). Their
  [agentskills.io](https://agentskills.io) spec is worth reading before we
  finalize our plugin manifest schema.
- Topic tags on the repo include `openclaw`, `clawdbot`, `moltbot`,
  `claude-code`, `codex` — Nous Research has a whole agent family. Our
  `workspace-template/adapters/openclaw/` adapter predates Hermes's
  rebrand; check whether it still points to a live project.

**Signals to react to:**
- If `agentskills.io` spec picks up mass adoption → align our plugin
  manifest so the same skill repo installs on Hermes AND Starfire.
- If Hermes ships multi-agent / A2A → direct overlap with our core thesis.
- If Atropos RL trajectory generation becomes the standard for training
  tool-calling models → our workspace activity logs should adopt the
  trajectory schema so users can export training data.

**Last reviewed:** 2026-04-12 · **Stars / activity:** ~61k ⭐, pushed today

---

### gstack — `garrytan/gstack`

**Pitch:** "Use Garry Tan's exact Claude Code setup: 23 opinionated tools
that serve as CEO, Designer, Eng Manager, Release Manager, Doc Engineer,
and QA." Claude Code skills bundle, MIT, ~70k⭐ and going viral on X.

**Shape:** A single directory of Markdown slash-command definitions
installed at `~/.claude/skills/gstack/`, invoked inside one Claude Code
session: `/office-hours`, `/plan-ceo-review`, `/review`, `/qa`, `/ship`,
`/land-and-deploy`, `/cso` (security), `/retro`, etc. No services, no
containers, no DB — just prompts and scripts that the Claude Code CLI
executes in whatever repo the user has open.

**Overlap with us:**
- **Same role metaphor as starfire-dev.** Both cast AI work as a cast of
  roles (CEO, Eng Manager, Designer, Security, QA). The naming overlap is
  nearly 1:1 with our org template.
- **Claude Code-native**, Markdown-driven config, "skills" as the unit.
- Team-mode auto-updates shared repos — same instinct as our org templates.

**Differentiation:** gstack is **sequential, single-session, single-repo.**
One Claude Code session runs each slash command in turn; the "team" is a
persona switch, not separate processes. We're **parallel, multi-session,
hierarchical**: real containers, A2A between siblings, a visual canvas,
real-time WebSocket updates, schedules, org bundles. gstack has no
multi-agent coordination, no A2A, no canvas, no workspace persistence
beyond git — it's a brilliant prompt library, not an orchestration platform.

**Worth borrowing:**
- **`/retro` command**: generates a weekly retrospective from git history
  ("140,751 lines added, 362 commits, ~115k net LOC in one week"). Would
  be a natural addition to our PM agent's toolbox — `commit_memory` +
  git log synthesis. Cheap win.
- **`/autoplan` and `/freeze` / `/guard` / `/unfreeze`** for architectural
  guardrails during a risky change. Maps cleanly onto our approval flow —
  could turn into a `/freeze` hook that sets a workspace-level policy flag
  preventing certain tool calls during a migration.
- **Role-prompt library.** gstack has spent a lot of effort on the CEO /
  Designer / Eng Manager personas. Even without adopting their runtime,
  we could lift the prompt text into our starfire-dev system-prompt.md
  files with attribution. Their CSO (OWASP + STRIDE audit) and Designer
  (AI-slop detection) personas are both stronger than ours today.
- **Team-mode auto-update** (throttled once/hour, network-failure-safe,
  silent) — good pattern for keeping plugins in sync across an org
  without requiring manual `/plugins/install` calls.

**Terminology collisions:**
- "Skills" — gstack ships everything as Claude Code skills (filesystem
  convention `~/.claude/skills/<name>/`). Same filesystem shape as
  ours AND Hermes AND Holaboss. Four projects, one spec shape — should
  formalize with [agentskills.io](https://agentskills.io).
- "Ship / Release" — their `/ship` is a local PR-and-merge flow;
  nothing to do with our A2A lifecycle.
- Mentions "OpenClaw" (247k ⭐ claim) as inspiration — tracks with the
  Hermes entry's note that the OpenClaw name is alive in multiple
  ecosystems.

**Signals to react to:**
- If gstack adds multi-session / parallel execution (spawning multiple
  Claude Code workers and routing between them) → direct competitor
  with a 70k⭐ head start. Revisit our differentiation messaging.
- If their `/plan-ceo-review` prompt or `/qa` browser flow becomes an
  informal standard → copy it into starfire-dev's system prompts.
- If Garry Tan posts a video deploying gstack on a new use case →
  high-signal about what "everyone" will ask us to support next week.

**Last reviewed:** 2026-04-12 · **Stars / activity:** ~70k ⭐, pushed yesterday

---

### Composio — `composio-dev/composio`

**Pitch:** "The integration layer for AI agents — 250+ tools across Slack,
GitHub, Telegram, Linear, Discord, and more, with managed auth."

**Shape:** Python + TypeScript SDK. Pure integration library — no agent
runtime, no visual canvas. Plugs into any LLM framework (LangChain,
LangGraph, AutoGen, CrewAI, Claude, OpenAI Agents). Managed auth so agents
can act on user-connected accounts. MIT-adjacent, ~18k ⭐.

**Overlap with us:** Both provide agent-accessible Slack, Telegram, and
Discord channels. Both handle OAuth / credential management for workspace
integrations. Channels feature in `platform/internal/handlers/channels.go`
does a subset of what Composio does for the messaging platforms.

**Differentiation:** Composio is a tool library, not a runtime or org
hierarchy. No canvas, no A2A between agents, no org structure. They're
"the 250 tools agents can call"; we're "the company that runs the agents."
Composio could be a dependency inside a Starfire workspace skill — not a
competitor for the platform layer.

**Worth borrowing:**
- **Trigger model:** inbound webhook → fire agent → respond in same channel.
  Our channels feature handles outbound well but inbound triggers are still
  manually configured. Composio's trigger schema is worth adopting.
- **"Connected accounts" pattern:** per-workspace OAuth token stored per
  integration, reused across runs. Our `workspace_channels` JSONB config is
  close; formalize as a named model.
- **Auth sandbox:** test mode that mocks API calls — useful for our
  `POST /workspaces/:id/channels/:id/test` endpoint.

**Terminology collisions:**
- "actions" = their tool calls; we use "skills."
- "triggers" = their inbound webhooks; we use channels + schedules.

**Signals to react to:**
- If they add persistent agent identity across trigger runs → direct overlap
  with our workspace model.
- If they add A2A between agent sessions or multi-agent orchestration → threat
  to our integration story.
- If `agentskills.io` adopts Composio trigger schema → we should too.

**Last reviewed:** 2026-04-13 · **Stars / activity:** ~18k ⭐, active

---

### n8n — `n8n-io/n8n`

**Pitch:** "Fair-code workflow automation with 400+ integrations — build AI
pipelines visually, self-host or cloud."

**Shape:** Node.js, self-hosted or n8n cloud. Visual workflow builder (nodes
+ edges, not unlike React Flow). 400+ connectors: Slack, Telegram, Discord,
WhatsApp, Email, GitHub, Linear, Notion, … plus dedicated AI nodes
(LLM chains, agent nodes, vector stores, tool use). Fair-code license
(source-available, free for internal use). ~50k ⭐, pushed daily.

**Overlap with us:**
- Visual graph metaphor for orchestrating work (their nodes ≈ our canvas
  workspaces).
- Connects AI agents to Slack / Telegram / Discord / WhatsApp — identical
  surface to our `workspace_channels` feature.
- Scheduled automations (cron triggers) → same as `workspace_schedules`.
- Self-hostable, Docker Compose first-class.

**Differentiation:** n8n is trigger→step→step→output (stateless sequential
workflow per run). No persistent agent identity, no shared memory across
runs, no org hierarchy, no A2A between agents. Each execution is isolated.
We're "agents that remember, collaborate, and hold roles"; they're "workflows
that transform data." The UX audiences barely overlap: n8n users are ops/no-code
builders; Starfire users are developers building agent companies.

**Worth borrowing:**
- **Channel trigger UX:** select platform → OAuth → pick chat → done in
  three clicks. Our channel setup requires more manual config; this flow is
  the right target for `POST /workspaces/:id/channels`.
- **"Test workflow" dry-run:** one-click test execution with live output.
  Maps well onto our `POST /workspaces/:id/channels/:id/test` — we should
  fire a real test message and show the round-trip result inline.
- **Sticky notes on canvas:** freeform annotation nodes for documentation.
  Cheap win for our canvas — could be a "comment node" workspace type.
- **Execution log with step-level timing:** n8n shows each node's in/out
  data and ms. Our `activity_logs` captures A2A traffic but not intra-agent
  step timing. Worth adding to the trace view.

**Terminology collisions:**
- "workflow" — their atomic unit; for us "workflow" is informal. No hard
  collision but our marketing copy should avoid it to stay distinct.
- "nodes" — their workflow steps; our canvas nodes are workspaces. Different
  enough to not cause user confusion, but worth noting in docs.

**Signals to react to:**
- If n8n ships persistent agent nodes (memory between runs) → direct
  substitute for simple Starfire use cases. They've been adding AI nodes
  fast (AI Agent node shipped 2024-Q3).
- If they add multi-agent coordination with shared state → revisit our
  differentiation messaging.
- If a major Slack/Discord bot tutorial uses n8n instead of a custom agent
  → indicates channel-first UX is the market expectation we need to match.

**Last reviewed:** 2026-04-15 · **Stars / activity:** ~50k ⭐, v2.17.0 released Apr 13 2026, pushed daily

---

### Pydantic AI — `pydantic/pydantic-ai`

**Pitch:** "AI Agent Framework, the Pydantic way."

**Shape:** Python SDK (MIT), ~16.3k ⭐, last release v1.8.0 on April 10, 2026 — actively maintained at high velocity. Single and multi-agent, with typed dependency injection (`RunContext[DepsType]`), structured/validated outputs (`Agent[Deps, OutputType]`), composable capability bundles (tools + hooks + instructions + model settings), built-in streaming, and human-in-the-loop tool approvals. Supports A2A and MCP natively as first-class integrations. Model-agnostic: OpenAI, Anthropic, Gemini, Mistral, Cohere, DeepSeek, Bedrock, Vertex, Ollama, OpenRouter, and more. Observability via Pydantic Logfire.

**Overlap with us:** A2A support means Pydantic AI agents can speak directly to Starfire workspaces over our native protocol — they're potential consumers of Starfire's registry, not just a parallel ecosystem. MCP integration mirrors our workspace tool model. The composable capability bundles are the same instinct as our plugin/skills system. Logfire's agent tracing is a polished alternative to our `GET /workspaces/:id/traces` + Langfuse stack.

**Differentiation:** Pydantic AI is a library for building agents in Python — no visual canvas, no Docker workspace isolation, no registry/discovery, no scheduling, no WebSocket org chart, no channels. It's the in-process layer; we're the operational platform layer. The two are naturally complementary: a Starfire workspace *running* Pydantic AI agents is a valid architecture, not a contradiction.

**Worth borrowing:**
- **Typed dependency injection via `RunContext`** — passing strongly-typed deps (DB connection, API client, user object) into every tool and instruction without global state. Our `config.yaml` passes env vars; this pattern is safer and more testable.
- **`Agent[Deps, OutputType]` generic typing** — structured, schema-validated agent outputs. Our A2A responses are freeform text; adopting structured output schemas at the A2A layer would enable typed inter-workspace contracts.
- **Composable capability bundles** — reusable packages of tools + hooks + instructions. Our plugins install files; this is the right next evolution (code bundles, not just Markdown).

**Terminology collisions:**
- "capabilities" — their term for composable tool+instruction bundles; we use "plugins" or "skills."
- "RunContext" — their typed dependency carrier; not a shared term, but will appear in codebases mixing Pydantic AI + Starfire adapters.
- "tools" — same word, same meaning. No collision, but documentation should be explicit about Pydantic AI tools vs. MCP tools vs. Starfire skills.

**Signals to react to:**
- If Pydantic AI ships a workspace/session persistence layer → fills the one gap between it and Starfire's value; revisit our Python-SDK adapter story.
- If `pydantic-deepagents` (`vstorm-co/pydantic-deepagents`) gains traction — "Claude Code–style deep agents on Pydantic AI" — it would become a direct competitor to our Claude Code runtime adapter.
- If Logfire's agent tracing becomes the de facto standard → align our trace schema so Logfire can ingest Starfire workspace traces natively.

**Last reviewed:** 2026-04-13 · **Stars / activity:** ~16.3k ⭐, v1.8.0 released April 10, 2026

---

### Rivet — `Ironclad/rivet`

**Pitch:** "The open-source visual AI programming environment and TypeScript library."

**Shape:** Electron desktop app + TypeScript library (MIT), ~4.5k ⭐. Visual node-based editor where AI workflows are built by connecting nodes in a graph: LLM call nodes, tool nodes, subgraph nodes, conditional branches. Runs locally; exports workflows as `.rivet-project` files that can be embedded in applications via the `@ironclad/rivet-node` npm package. Built and open-sourced by Ironclad (a Series D contract intelligence company). Model-agnostic. Plugin marketplace for custom node types.

**Overlap with us:** The canvas is the obvious overlap — both products present AI agent work as a visual graph. Rivet's subgraph nesting (complex workflows broken into reusable components) maps to our parent/child workspace hierarchy. The plugin marketplace for custom nodes mirrors our `plugins/` registry. Rivet workflows can call external APIs, making them potential consumers of Starfire's `/workspaces/:id/a2a` endpoint — a Rivet node that delegates to a Starfire agent is a plausible integration.

**Differentiation:** Rivet is a **workflow authoring tool**, not an agent runtime. A `.rivet-project` file describes a static graph; there's no persistent agent identity, no memory across runs, no org hierarchy, no real-time WebSocket canvas, no scheduling, no Docker container management. The Rivet editor is for building workflows; Starfire is for running a live org of agents. The `/channels` angle is absent from Rivet — it has no concept of an agent receiving or sending messages via Telegram, Slack, or other social platforms. Rivet's audience is developers prototyping single pipelines; ours is teams deploying multi-agent organizations.

**Worth borrowing:**
- **Nested subgraph UX** — Rivet's handling of "graph within graph" as a first-class reusable node is the cleanest visual pattern for our parent/child workspace hierarchy. Our current Canvas flattens deeply nested teams into chips; Rivet's subgraph expand/collapse is the reference UX to study.
- **Node-level debug inspector** — clicking any node in a completed run shows its exact inputs, outputs, and latency. Our Canvas chat shows A2A messages but not intra-workspace step-level data. This is the natural evolution of our trace view.
- **`.rivet-project` portability** — workflow-as-file, embeddable in any TypeScript app via npm. Suggests we should support a "workspace bundle export" that can run outside Starfire, not just be imported back into it.

**Terminology collisions:**
- "graph" — their graph is a workflow definition (static); ours is the live org chart (dynamic, stateful). Different semantics, same word.
- "node" — their nodes are workflow steps; our canvas nodes are workspaces. No runtime collision but documentation must be unambiguous.
- "plugin" — both have plugin systems; theirs extends the node palette, ours extends the workspace runtime.

**Signals to react to:**
- If Rivet adds persistent agent state between runs → closes the gap with Starfire for simple use cases; revisit our "quick start" story for non-enterprise users.
- If Rivet adds a "deploy workflow as agent endpoint" feature → their visual builder becomes a Starfire workspace creator; consider a Rivet → Starfire import adapter.
- If `.rivet-project` format becomes a de facto workflow interchange standard → support importing Rivet projects as Starfire workspace configs.

**Last reviewed:** 2026-04-13 · **Stars / activity:** ~4.5k ⭐, actively maintained

---

### Letta — `letta-ai/letta`

**Pitch:** "The platform for building stateful agents: AI with advanced memory that can learn and self-improve over time."

**Shape:** Python + TypeScript SDK (Apache-2.0), ~22k ⭐, v0.16.7 released March 31, 2026. Formerly MemGPT (the research project that pioneered OS-inspired virtual context management for LLMs). Letta's defining feature is a **multi-block memory architecture**: each agent holds named, editable in-context memory segments ("core memory") such as `human`, `persona`, and `archival` blocks, which the agent can read and write via tool calls. Memories persist across sessions in a Letta Server (self-hosted or Letta Cloud). Agents are accessed via a REST API. The **ADE (Agent Development Environment)** is a graphical interface for creating, testing, and monitoring agents in real-time. Multi-agent support via subagents and shared memory. Model-agnostic (OpenAI, Anthropic, local LLMs via Ollama).

**Overlap with us:** Letta's named memory blocks (`human`, `persona`, `archival`) are a structured evolution of the same problem our `agent_memories` table and `MEMORY.md` file solve — persistent, durable knowledge for a long-lived agent. The ADE's graphical agent-monitoring interface overlaps with our Canvas; both offer a UI to inspect and interact with running agents. Letta Server exposes a REST API that accepts messages at agent endpoints — structurally similar to our A2A proxy (`POST /workspaces/:id/a2a`). Multi-agent subagent support maps to our parent/child workspace hierarchy. Letta's `initial_prompt` equivalent (agent system prompt + memory bootstrap) mirrors our `initial_prompt` in `config.yaml`.

**Differentiation:** Letta is focused on **the single-agent memory problem**, not the multi-agent org problem. No Docker container isolation per agent, no workspace registry, no real-time WebSocket org chart, no scheduling, no channels to Slack/Telegram/Discord. The ADE shows individual agents; it does not visualize an org hierarchy or inter-agent A2A traffic. Letta's multi-agent support is hierarchical subagent spawning within a single Letta Server context — not independently deployable, independently schedulable workspaces. We're "a company of agents"; Letta is "an agent with a very good memory."

**Worth borrowing:**
- **Named, agent-editable memory blocks** — the `human` / `persona` / `archival` distinction is the clearest taxonomy we've seen for agent memory. Our `agent_memories` namespace is flat; adopting explicit named blocks (at minimum: `self`, `user`, `task-context`, `long-term-knowledge`) would make memory more inspectable and auditable in the Canvas.
- **Memory self-editing as a tool call** — Letta agents call `core_memory_replace(label, old, new)` and `archival_memory_insert(content)` as first-class tool actions, making memory updates part of the visible tool-call trace. Our `commit_memory` MCP tool is close; making it show up in `activity_logs` as a named tool call (not a silent background action) would match this pattern.
- **ADE real-time message inspector** — the ADE shows each tool call, memory read/write, and reasoning step inline in a timeline. This is more granular than our Canvas chat tab; it's the reference design for a "step-through debug mode" in our trace view.

**Terminology collisions:**
- "archival memory" — Letta: a searchable long-term store the agent queries via tool calls. Ours: not a defined term. Our `agent_memories` table is functionally similar but not surfaced to agents as a named primitive.
- "persona" — Letta: a named memory block containing the agent's self-description. Ours: the `role:` field in `config.yaml` plus the system prompt. Same intent, different packaging.
- "agent" — Letta: a long-lived server-side object with persistent memory, accessed via REST. Ours: a Docker container running one of six runtimes. Same word, substantially different operational model.

**Signals to react to:**
- If Letta ships a multi-agent canvas that visualizes org hierarchies (not just individual agent inspection) → direct overlap with our Canvas; they have strong memory credibility that could attract our target buyer.
- If Letta formalizes a memory-block schema as an open spec (building on their MemGPT research lineage) → evaluate adopting it as Starfire's `agent_memories` schema to gain interoperability with the Letta ecosystem.
- If Letta Cloud adds Slack/Telegram/Discord inbound triggers → they gain channels capability; currently absent, but a REST API means it's one webhook away.
- Watch v0.x → v1.0 trajectory: v0.16.7 suggests pre-1.0 API stability; a 1.0 GA announcement would signal enterprise readiness and an accelerated sales motion.

**Last reviewed:** 2026-04-13 · **Stars / activity:** ~22k ⭐, v0.16.7 March 31, 2026

---

### Trigger.dev — `triggerdotdev/trigger.dev`

**Pitch:** "Build and deploy fully-managed AI agents and workflows."

**Shape:** TypeScript (Apache-2.0), ~14.5k ⭐, v4.4.3 released March 10, 2026. Started as a developer-friendly alternative to cron + background jobs; v4 repositions it squarely as **durable execution infrastructure for AI agents**. Tasks are TypeScript functions decorated with `task()` — they run in a managed cloud with: automatic retry with exponential backoff, checkpoint/resume (task state saved to storage, resumed after crash or timeout), queue and concurrency control, and cron scheduling up to one-year duration. Human-in-the-loop via `waitForApproval()`. MCP server available (`trigger-dev` MCP) so AI assistants (Claude Code, Cursor, etc.) can trigger tasks, check run status, and deploy from chat. Warm starts execute in 100–300ms. Fully self-hostable.

**Overlap with us:** Trigger.dev's `schedules.task()` cron system overlaps directly with our `workspace_schedules` table and `POST /workspaces/:id/schedules` API — both schedule recurring prompts/tasks on a cron expression. The checkpoint/resume model (`waitForApproval`, `wait.for()`) is a precise parallel to our workspace `pause` / `resume` lifecycle. Human-in-the-loop approval gates match our `POST /workspaces/:id/approvals`. The MCP server enabling AI agents to trigger tasks maps to the same use case as our MCP server's `delegate_task` tool. Both platforms treat long-running, fault-tolerant execution as a core design constraint.

**Differentiation:** Trigger.dev has **no agent identity** — tasks are stateless TypeScript functions, not persistent agents with memory, roles, or system prompts. No visual canvas, no org hierarchy, no A2A protocol, no workspace registry. It is execution infrastructure, not an agent platform. The right mental model: Trigger.dev is to Starfire what Temporal is to Starfire — a lower-level durable execution substrate that Starfire's workspaces could use as a backend for their scheduled tasks, rather than a replacement for Starfire itself. Their `/channels` story is inbound-only (HTTP triggers, webhooks, cron) with no native Slack/Telegram messaging surface.

**Worth borrowing:**
- **Idempotency keys on task invocation** — `trigger("send-report", payload, { idempotencyKey: runId })` ensures a task is only ever executed once for a given key, even if triggered multiple times. Our delegation system has no equivalent guard; duplicate delegations from container-restart races are a known issue (see `delegationRetryDelay` in `delegation.go`). Adding idempotency keys to `POST /workspaces/:id/delegate` would fix the duplicate-execution class of bugs.
- **`waitForApproval()` inline in task code** — instead of a separate approvals table and polling loop, the task itself calls `await wait.for({ event: "approval" })` and suspends. Our approval flow requires a separate API round-trip and the agent to re-check; Trigger.dev's inline suspension is the right long-term model.
- **Warm-start pool for sub-300ms agent starts** — Trigger.dev pre-warms TypeScript runtimes to achieve 100–300ms cold start. Our Docker workspace startup is measured in seconds. Worth evaluating their warm-pool approach for our claude-code and langgraph adapters.

**Terminology collisions:**
- "task" — Trigger.dev: a decorated TypeScript function, the atomic unit of execution. Ours: informal (used in delegation context and `current_task` heartbeat field). Their definition is more precise; consider whether our heartbeat `current_task` field should be renamed to avoid collision with Trigger.dev vocabulary in integrations.
- "schedule" — same word, same meaning. Trigger.dev's cron schedule API and ours (`workspace_schedules`) are functionally identical at the surface. Our docs should distinguish "Starfire schedules" from "Trigger.dev schedules" clearly when positioning integrations.
- "run" — Trigger.dev: a single execution of a task with full lifecycle tracking. Ours: informal. No hard collision.

**Signals to react to:**
- If Trigger.dev ships native agent identity (persistent state, memory across runs, named agents) → crosses from infrastructure into platform territory; reevaluate positioning.
- If the `trigger-dev` MCP becomes a de facto standard for AI-tool-triggered background work → add a Trigger.dev adapter to our workspace runtime so Starfire agents can fire Trigger.dev tasks as a tool call (complementary, not competitive).
- If Trigger.dev ships a Slack/Discord trigger adapter → they gain a channels surface; currently absent. Watch their integration roadmap.
- Their TypeScript-first stack and MCP server target the same developer audience as our Canvas + mcp-server. Co-marketing opportunity: "run your Starfire agent on a schedule via Trigger.dev" is a cleaner story than our current in-house cron for some user segments.

**Last reviewed:** 2026-04-13 · **Stars / activity:** ~14.5k ⭐, v4.4.3 March 10, 2026

---

### Microsoft Agent Framework — `microsoft/agent-framework`

**Pitch:** "A framework for building, orchestrating and deploying AI agents
and multi-agent workflows with support for Python and .NET."

**Shape:** Python + C# (Apache-2.0), 9.5k ⭐, v1.0 April 7, 2026.
Unifies Semantic Kernel and AutoGen into one production SDK. Graph-based
orchestration with streaming, checkpointing, and human-in-the-loop.
DevUI for interactive debugging. Official migration guides from both
AutoGen and Semantic Kernel.

**Overlap with us:** MCP Registry integration overlaps our `mcp-server`
tool surface. A2A hosting samples in both Python and .NET overlap
`a2a_executor.py` directly. Graph-based multi-agent workflows with
checkpoint/resume mirror our workspace pause/resume lifecycle.

**Differentiation:** Microsoft targets enterprise .NET shops migrating
existing AutoGen/Semantic Kernel projects. No visual canvas, no org
hierarchy, no agent marketplace or plugin registry. Agent-framework is a
**developer SDK** for building agents; Starfire is the **operating system
for agent teams**.

**Worth borrowing:**
- **DevUI** for live agent introspection and replay — our Canvas surfaces
  agent output but has no dev-mode debugger. A debug-mode DevUI would
  meaningfully improve contributor DX.
- **Formal middleware layer** for request/response interception — cleaner
  than our ad hoc hook system; worth formalizing in `a2a_executor.py`.

**Terminology collisions:**
- "plugin" — agent-framework: tool bundles (Semantic Kernel lineage). Ours:
  workspace add-ons. Different scopes, same word.
- "agent" — SDK object with declared role vs. running Docker container.

**Signals to react to:**
- If they ship a visual canvas for multi-agent org hierarchy → direct
  Canvas overlap; track their roadmap issues.
- If A2A samples become a de facto reference implementation → audit our
  `a2a_executor.py` for compatibility gaps.
- Our `adapters/autogen/` targets the now-deprecated predecessor; evaluate
  migrating to target agent-framework instead (see filed issue).

**Last reviewed:** 2026-04-15 · **Stars / activity:** 9.5k ⭐, v1.0 Apr 7 2026

---

### GenericAgent — `lsdefine/GenericAgent`

**Pitch:** "Self-evolving agent achieving full system control with 6× less
token consumption."

**Shape:** Python (95%), MIT, ~1.7k ⭐, created January 2026. ~3,000
lines total with a ~100-line core agent loop. Nine atomic tools cover
browser automation (with session injection), terminal, file ops,
keyboard/mouse input, screen vision, and ADB. Supports Claude, Gemini,
Kimi, MiniMax. Multi-frontend: Streamlit, Qt, WeChat, Feishu, DingTalk.

**Overlap with us:** Layered memory (L0–L4) maps conceptually to our
workspace memory tiers. Skill crystallization — tasks that succeed get
saved as reusable skills — is the same intent as our plugin install flow.

**Differentiation:** Single-user desktop automation tool, not a
multi-agent platform. No org hierarchy, no A2A, no cloud deployment.
Minimal codebase (~100-line loop) is the design goal; Starfire is a
full team-coordination platform.

**Worth borrowing:**
- **L0–L4 memory hierarchy** is more explicit than our current memory
  model — worth adopting as terminology in `AGENTS.md` so contributors
  can reason about which memory layer to read/write.
- **6× token-reduction claim**: their prompt compression and skill-reuse
  approach is worth benchmarking against our claude-code adapter's prompt
  budgeting, even if the absolute numbers don't transfer.

**Terminology collisions:**
- "skill" — GenericAgent: a crystallized, reusable task execution workflow.
  Ours: an installable plugin unit. Similar intent, different lifecycle.

**Signals to react to:**
- If token-compression techniques are published in detail → benchmark
  against our `claude_sdk_executor.py` prompt budgeting.
- If skill crystallization gains traction as a pattern → consider adding
  a `POST /workspaces/:id/skills/crystallize` endpoint for Starfire
  agents to persist successful task patterns.

**Last reviewed:** 2026-04-15 · **Stars / activity:** ~1.7k ⭐, Jan 2026

---

### AgentScope — `agentscope-ai/agentscope`

**Pitch:** "Build and run agents you can see, understand and trust."

**Shape:** Python (100%), Apache-2.0, 23.8k ⭐, v1.0.18 March 26, 2026.
Production-ready multi-agent framework with companion repos:
`agentscope-runtime` (async-sandbox deployment infra — Browser, GUI,
Filesystem, Mobile sandboxes) and `agentscope-studio` (visual dev +
monitoring environment). MCP tool integration as local callable functions.
A2A protocol support added December 2025. ReAct, HITL, voice, memory, RL
integration built in. Java port (`agentscope-java`) also active.

**Overlap with us:** MCP + A2A present — same protocol surface as Starfire.
**MsgHub** formalizes inter-agent message routing (fan-out, filtering,
pipeline ordering) — maps directly to our inter-workspace delegation system.
AgentScope-Studio overlaps our Canvas. Async sandboxes mirror our Docker
workspace runtimes.

**Differentiation:** AgentScope is a **developer framework** (write agent
code in Python); Starfire is an **agent OS** (configure agents via prompts
and plugins, no code). No org hierarchy, no role marketplace, no RBAC layer.

**Worth borrowing:**
- **MsgHub pipeline pattern** — formal message routing between agents with
  fan-out and filtering; our point-to-point A2A delegation has no routing
  layer. Adding one would reduce custom code in complex org templates (see
  filed issue).
- **Distributed Interrupt Service** — manual task preemption with pluggable
  state persistence and recovery; cleaner than our `pause` endpoint which
  has no checkpoint guarantees.

**Terminology collisions:**
- "pipeline" — AgentScope: ordered sequence of agent interactions. Ours:
  undefined. Risk of confusion when integrating AgentScope as a runtime.
- "agent" — Python class instance vs. Docker container.

**Signals to react to:**
- If AgentScope-Studio ships org-hierarchy multi-agent visualization → direct
  Canvas competition from a 23.8k-star project with enterprise traction.
- If MsgHub is published as an open spec → evaluate adopting as our
  inter-workspace routing standard.
- Java port gaining traction → enterprise buyers may prefer JVM-native agents;
  watch for an enterprise-tier AgentScope Cloud announcement.

**Last reviewed:** 2026-04-15 · **Stars / activity:** 23.8k ⭐, v1.0.18 Mar 26 2026

---

### oh-my-claudecode — `Yeachan-Heo/oh-my-claudecode`

**Pitch:** "Teams-first multi-agent orchestration for Claude Code. Zero learning
curve."

**Shape:** TypeScript/JS (MIT), 29.1k ⭐, v4.11.6 April 2026. A Claude Code
CLI plugin that orchestrates 19 specialized agents through a staged pipeline:
`team-plan → team-prd → team-exec → team-verify → team-fix (loop)`. Smart model
routing deploys Haiku for simple tasks and Opus for complex reasoning. tmux-based
parallelization runs N agents in parallel on a shared task list. Claims 3–5×
speedup and 30–50% token savings on large projects. Skill extraction
automatically captures successful debugging patterns into portable `.md` files.

**Overlap with us:** Both treat "a team of specialized agents coordinating on
shared work" as the primary product metaphor. Both use role-based agent names.
Both are Claude Code–native with a skill/plugin file convention. OMC's 19
specialized agents mirror our tier-2 research/dev/ops role structure.

**Differentiation:** OMC is a **single-machine CLI plugin** — all agents share
one shell via tmux, no Docker isolation, no RBAC, no visual canvas, no A2A
between independent processes, no scheduling, no channels. Starfire is a
**multi-machine agent OS**: real containers, cross-network A2A, visual org chart,
persistent workspace identity, governance and approval flows. OMC is "parallel
Claude Code sessions on one laptop"; Starfire is "a company of agents with
independent compute, memory, and governance."

**Worth borrowing:**
- **Smart model routing by task complexity** — cheap-model for simple tasks,
  expensive-model for hard reasoning. Could add this to our `a2a_executor.py`
  dispatch layer: route tasks tagged `complexity=low` to Haiku and
  `complexity=high` to Opus within the same workspace.
- **Staged verification loop** (`exec → verify → fix`) — a clean quality gate
  before delegation results are accepted. Worth building into our PM agent's
  delegation lifecycle as an optional `require_verification` flag.
- **Auto-extracted skill files** from successful task runs — same instinct as
  GenericAgent's skill crystallization; with two data points it's time to
  prototype `POST /workspaces/:id/skills/crystallize` for Starfire.

**Terminology collisions:**
- "team" — OMC's `--team` flag runs N agents in parallel on one machine; our
  "team" is a persistent org hierarchy across containers. Same word, different
  runtime scope.
- "skills" — OMC: auto-extracted `.md` patterns from debugging sessions; ours:
  installable plugin units.

**Signals to react to:**
- If OMC adds cross-machine A2A coordination → direct substitution for our
  orchestration layer in Claude Code shops; 29k stars means fast adoption.
- If OMC's model-routing heuristics are published → benchmark our delegation
  cost profile against theirs.
- If OMC's skill extraction format aligns with `agentskills.io` → our plugin
  manifest should support the same schema so skills install on both platforms.

**Last reviewed:** 2026-04-15 · **Stars / activity:** 29.1k ⭐, v4.11.6 Apr 2026

---

### Claw Code — `instructkr/claw-code`

**Pitch:** "Public Rust implementation of the claw CLI agent harness — the
fastest repo in history to surpass 100K stars."

**Shape:** Rust (96%) + Python helpers (MIT/community), 185k ⭐, 787 commits.
CLI agent harness architected as a clean-room rewrite of Claude Code's agent
loop (sparked by an accidental npm source-map leak in March 2026). Session
management, `.claude.json` config, container-first workflows, mock parity
harness for deterministic testing. API-key agnostic (Anthropic, OpenAI). No
affiliation with Anthropic. Not to be confused with the original OpenClaw
project referenced in our `adapters/openclaw/` — that predates this repo by
over a year.

**Overlap with us:** Claw Code agents run the same workspace-level tasks our
Claude Code runtime workspaces execute. Our Docker-per-workspace model is
compatible with Claw Code's container-first workflow design. The mock parity
harness targets the same testing gap as our `a2a_executor.py` test suite.

**Differentiation:** Claw Code is a **standalone CLI** — no multi-agent
coordination, no org hierarchy, no A2A, no visual canvas, no scheduling, no
channels, no RBAC. It gives developers a Claude Code alternative; Starfire
provides the coordination layer and agent OS on top. The two are
complementary: a Starfire workspace *could* run Claw Code under the hood
instead of Claude Code with a new adapter.

**Worth borrowing:**
- **Mock parity harness** — deterministic test execution against a fake agent
  runtime. We have limited coverage of `a2a_executor.py` edge-case behavior;
  this pattern is the right model for our CI adapter tests.
- **`claw doctor` health-check CLI** — structured pre-flight self-diagnosis.
  Worth adding a `/workspaces/:id/health` endpoint to our platform API that
  runs equivalent checks (auth, tool access, memory connectivity, A2A
  reachability) and returns a structured report.

**Terminology collisions:**
- "claw" / "openclaw" — the `claw` CLI name surface-overlaps our
  `adapters/openclaw/` adapter, but they target different projects. Our
  adapter docs should clarify which claw it targets.
- "session" — Claw Code: persisted CLI execution context. Ours: informal.

**Signals to react to:**
- If Claw Code adds multi-agent coordination (spawning + routing multiple
  `claw` instances) → directly substitutes for our Claude Code adapter in
  cost-sensitive environments; 185k stars means community momentum is
  already enormous.
- If Anthropic officially acknowledges or partners with Claw Code → signals
  the Claude Code architecture is becoming a public API surface; our adapters
  should declare explicit version compatibility.
- If Claw Code's mock harness schema is published → adopt it in our CI for
  adapter regression testing.

**Last reviewed:** 2026-04-15 · **Stars / activity:** 185k ⭐, community-driven

---

### CowAgent — `zhayujie/CowAgent`

**Pitch:** "AI assistant built on LLMs with autonomous planning, long-term memory, knowledge management, and a skill engine — lighter and more convenient than OpenClaw."

**Shape:** Python (MIT), 43.3k ⭐, actively maintained. Single-agent assistant framework with autonomous task planning, layered persistent memory (core memory → daily memory → dream distillation), and a skill engine that installs from Skill Hub, GitHub, or creates skills via conversation. Multi-platform: WeChat, Feishu, DingTalk, Enterprise WeChat, QQ, and web. Model-agnostic (OpenAI, Claude, Gemini, DeepSeek, Qwen, GLM, Kimi). Built-in tools: file ops, terminal, browser automation, scheduled tasks, multimodal.

**Overlap with us:** Skill engine (install from Hub, GitHub, or create via conversation) is structurally identical to our `plugins/` registry. Layered memory architecture parallels our `agent_memories` table. Multi-platform messenger support mirrors `workspace_channels`. Star trajectory (~43k) means this is a community reference implementation — things it normalizes will become ecosystem expectations.

**Differentiation:** CowAgent is a **single-user personal assistant**, not a multi-agent platform. No org hierarchy, no A2A, no Docker container isolation, no visual canvas, no scheduling, no approval flows. Self-positioned as "lighter than OpenClaw" — emphasizes simplicity over governance. No concept of roles, RBAC, or multi-workspace coordination.

**Worth borrowing:**
- **Dream distillation memory** — end-of-session LLM pass that condenses short-term daily memory into durable long-term knowledge. Second data point after Hermes. Worth prototyping in our workspace template: a post-session hook that summarizes recent `activity_logs` into `commit_memory`.
- **Skill Hub discovery UX** — browsable index of installable skills, separate from the runtime. Our `plugins/` registry has no discovery surface; a marketplace landing page would lower the barrier for org admins significantly.

**Terminology collisions:**
- "skills" — same filesystem convention as gstack, Hermes, OMC, vercel-labs/skills, agentskills.io. Five data points; see filed issue #[skills-standard].
- "dream distillation" — their term for memory consolidation. Should be explicitly named in our memory model docs.

**Signals to react to:**
- If CowAgent adds multi-agent A2A support → closes its main gap; 43k stars = fast adoption.
- If Skill Hub gains an open submission API → publish our plugins there (cross-install opportunity).
- If "lighter than OpenClaw" attracts OpenClaw's user base → check whether our `adapters/openclaw/` users migrate and whether the adapter needs updating.

**Last reviewed:** 2026-04-15 · **Stars / activity:** 43.3k ⭐, trending Apr 15 2026

---

### vercel-labs/open-agents — `vercel-labs/open-agents`

**Pitch:** "An open-source template for building cloud agents — from prompt to code changes without keeping your laptop involved."

**Shape:** TypeScript 99%, MIT, 2.5k ⭐, pushed April 15 2026 (brand new). Reference architecture for a cloud-hosted coding agent with three-layer separation: (1) **Web app** — Next.js, auth, sessions, streaming chat UI; (2) **Agent workflow** — durable multi-step execution via Vercel Workflow SDK, runs *outside* the sandbox; (3) **Sandbox** — isolated VM with filesystem, shell, git, dev servers, and snapshot-based resumption. Agent interacts with the sandbox exclusively through tool calls (file read/edit, search, shell). Auto-commit, push, and PR creation built in. Companion repos: `vercel-labs/skills` (installable skills standard) and `vercel-labs/agent-browser` (browser automation CLI).

**Overlap with us:** Three-layer stack (web UI / agent workflow / sandbox) maps directly to Starfire's Canvas / workspace process / Docker container. Durable workflow with snapshot-resume mirrors our `pause` / `resume` lifecycle. The `vercel-labs/skills` standard is the fifth data point in the converging filesystem skills convention (gstack, Hermes, OMC, CowAgent, agentskills.io). Repo integration (clone, branch, PR) overlaps our DevOps workspace role.

**Differentiation:** open-agents is a **single-agent cloud coding template** — no org hierarchy, no A2A, no canvas, no scheduling, no channels, no RBAC. Designed to be forked, not operated as a platform. Workspaces are ephemeral-by-design; Starfire workspaces are persistent identities with memory, roles, and governance. Vercel infrastructure is the implicit target; Starfire is infra-agnostic and multi-runtime.

**Worth borrowing:**
- **Agent-outside-sandbox** execution model — agent process has zero direct filesystem access and must use tool calls to interact with the sandbox. Cleaner security boundary than our claude-code runtime which runs inside the container. Worth evaluating as a "strict isolation mode" for DevOps workspace template.
- **Snapshot-based sandbox resumption** — sandbox state is snapshotted and resumed rather than kept warm. Could reduce idle container compute for infrequently-triggered workspaces (nightly audits, scheduled reports).
- **`vercel-labs/skills` format** — see filed issue for aligning our plugin manifest.

**Terminology collisions:**
- "sandbox" — their VM execution layer; our Docker container per workspace. Same concept, different scope.
- "workflow" — their durable execution run; ours is informal. No runtime collision but docs should be unambiguous.
- "skills" — `vercel-labs/skills` uses the same filesystem convention as four other projects. Fifth data point.

**Signals to react to:**
- If Vercel adds multi-agent A2A → they have infra distribution + developer mindshare to become a serious platform competitor; track their roadmap issues closely.
- If `vercel-labs/skills` is published as a formal spec → immediate priority to align our plugin manifest (see filed issue).
- If open-agents crosses 10k ⭐ → Vercel distribution flywheel; add a "deploy to Vercel" option to our `workspace-template` to capture adjacent users.

**Last reviewed:** 2026-04-15 · **Stars / activity:** 2.5k ⭐, pushed Apr 15 2026

---

### claude-mem — `thedotmack/claude-mem`

**Pitch:** "Automatically captures everything Claude does during your coding sessions, compresses it with AI, and reuses that context in future sessions."

**Shape:** TypeScript 83%, AGPL-3.0, 57.5k ⭐. Claude Code plugin installed via `npx claude-mem install`. Five lifecycle hooks (SessionStart, UserPromptSubmit, PostToolUse, Stop, SessionEnd) capture all tool usage automatically. AI compresses captures into semantic summaries stored in SQLite + Chroma vector DB. Hybrid semantic/keyword search. Three-layer progressive disclosure for retrieval: `search` (compact index, ~50–100 tokens) → `timeline` (chronological context) → `get_observations` (full detail on demand). Worker service on port 37777 with web UI. Also supports Gemini CLI and OpenCode.

**Overlap with us:** Directly targets the compaction problem our Holaboss entry references. Our `workspace-template/hooks/` has stub lifecycle hooks; claude-mem shows the full production pattern. Chroma vector search is an alternative to our DB-only `agent_memories`. The 3-layer retrieval pattern applies to our `recall_memory` MCP tool.

**Differentiation:** Single-session plugin, not a multi-agent platform. No org hierarchy, no A2A, no canvas, no scheduling. AGPL-3.0 is a **commercial risk** — any product incorporating it must open-source. Evaluate for inspiration, not direct adoption.

**Worth borrowing:**
- **3-layer progressive disclosure** — index → timeline → full detail on demand. Apply to `recall_memory`: return a compact index first, fetch full records only when agent requests them. Significant token savings.
- **Five lifecycle hooks** — exact hook shape needed in `workspace-template/hooks/`. Copy the names and contract as our hook standard.

**Terminology collisions:**
- "session" — their capture unit; ours is informal. No hard collision.
- "observations" — their raw captured tool events. Our `activity_logs` is the equivalent.

**Signals to react to:**
- If claude-mem adds cross-workspace memory sharing → overlaps our TEAM-scoped `agent_memories`; evaluate as a memory backend.
- If a permissive-licensed fork appears → reassess for direct adoption in `workspace-template/`.
- 57k stars on a Claude Code plugin is the largest signal yet that session persistence is the #1 pain point in our target user base.

**Last reviewed:** 2026-04-15 · **Stars / activity:** 57.5k ⭐, trending today

---

### Google Agent Development Kit — `google/adk-python`

**Pitch:** "An open-source, code-first Python toolkit for building, evaluating, and deploying sophisticated AI agents with flexibility and control."

**Shape:** Python (Apache-2.0), ~8.2k ⭐, v1.0.0 stable (2026). Google's official agent framework. Supports LLM, workflow, and custom agent types in flexible hierarchies. Native A2A protocol integration. MCP as both client and server. HITL tool confirmation flow. Session rewind (roll back to any prior invocation state). Vertex AI Code Execution Sandbox for agent-generated code. Deployment target: Agent Engine on Google Cloud. Gemini-first but model-agnostic.

**Overlap with us:** A2A support means ADK agents can join our mesh as first-class peers — our `a2a_executor.py` should be tested against ADK's A2A client. MCP client/server mirrors our `mcp-server` surface. HITL confirmation maps to `POST /workspaces/:id/approvals`. Multi-agent hierarchies are the same architectural thesis as our parent/child workspace model. Agent Engine is a direct GCP-native alternative deployment path for our target segment.

**Differentiation:** ADK is a **developer SDK with a GCP deployment backend** — no visual canvas, no org marketplace, no role-based plugin registry, no WebSocket org chart, no channels, no RBAC governance. Agents are Python code objects; Starfire workspaces are prompt-and-YAML configured. Google targets GCP-native enterprise; Starfire targets developers wanting a no-code agent company.

**Worth borrowing:**
- **Session rewind** — roll back agent state to before a previous invocation, analogous to git checkout on agent history. Our `pause`/`resume` has no rollback; this would be high value for debugging failed delegation chains.
- **Formal callback system** for intercepting all tool calls — more structured than our ad-hoc hooks in `a2a_executor.py`.

**Terminology collisions:**
- "agent" — ADK: a Python class with declared tools and model. Ours: a Docker container. Same word, very different runtime model.
- "plugin" / "skill" — ADK uses neither; they say "tool" and "extension." No collision, but our docs should be explicit when discussing ADK integrations.

**Signals to react to:**
- If ADK ships a visual canvas for multi-agent org hierarchies → direct Canvas competition from Google's distribution.
- If ADK's A2A samples become the de facto reference → audit `a2a_executor.py` for compatibility gaps immediately.
- If Agent Engine pricing is announced below our self-hosted cost → GCP lock-in risk for cost-sensitive segment; position Starfire's infra-agnosticism harder.

**Last reviewed:** 2026-04-15 · **Stars / activity:** ~8.2k ⭐, v1.0.0 stable 2026

---

### Bifrost — `maximhq/bifrost`

**Pitch:** "Fastest enterprise AI gateway — 50× faster than LiteLLM, unified access to 15+ LLM providers with adaptive load balancing, guardrails, and MCP integration at <100µs overhead."

**Shape:** Go 73% + TS 19%, Apache-2.0, 3.8k ⭐, 1,457+ releases, actively maintained. Transparent reverse proxy between agent code and LLM providers. Single OpenAI-compatible API abstracts 15+ providers (OpenAI, Anthropic, Bedrock, Vertex, Azure, Cerebras, Cohere, Mistral, Ollama, Groq). Core features: provider failover with zero downtime, semantic response caching, MCP integration for external tool access, per-key load balancing, budget management, rate limiting, access control. Enterprise: SSO, HashiCorp Vault, Prometheus metrics, distributed tracing, multi-node clustering. Deploy via npx, Docker, or Go SDK.

**Overlap with us:** MCP integration means Bifrost can sit between Starfire workspaces and their LLM providers — a Starfire org could route all LLM calls through Bifrost for cost control, failover, and observability. Budget management + rate limiting maps directly to per-workspace cost governance. Semantic caching reduces token spend on repeated delegation patterns (eco-watch, scheduled audits). Prometheus + distributed tracing complement our `activity_logs`.

**Differentiation:** Bifrost is **infrastructure middleware**, not an agent framework. No agent identity, no memory, no multi-agent coordination, no workspace concept, no canvas, no scheduling. The right model: Bifrost is to LLM providers what an API gateway is to microservices — Starfire runs on top of it, not instead of it. Fully complementary.

**Worth borrowing:**
- **Semantic response caching** — cache LLM responses by semantic similarity. Our workspaces make near-identical calls on every run (system prompts, recurring queries); caching could cut cost 20–40% on predictable workflows.
- **Provider failover** — automatic reroute to Bedrock/Vertex when Anthropic is degraded. Our workspaces have no LLM fallback today; a Bifrost adapter in `workspace-template/` would give every workspace failover for free.

**Terminology collisions:**
- "plugin architecture" — Bifrost: middleware plugins for their gateway pipeline. Ours: installable workspace add-ons. Different scope.
- "guardrails" — Bifrost: LLM response filtering. Ours: undefined. Disambiguate in docs when describing integrations.

**Signals to react to:**
- If Bifrost adds agent identity or session tracking → shifts from gateway toward platform; reassess.
- If Bifrost's MCP integration expands to full MCP gateway (not just client) → overlaps our `mcp-server` tool surface.
- "Deploy Bifrost in front of your Starfire org for failover + cost control" is a compelling enterprise pitch — consider a `workspace-template/adapters/bifrost/` adapter as a low-effort high-value addition.

**Last reviewed:** 2026-04-15 · **Stars / activity:** 3.8k ⭐, Apache-2.0, active

---

### Claude Agent SDK — `anthropics/claude-agent-sdk-python`

**Pitch:** "Programmatic access to Claude Code — build automated agents, define in-process custom tools, intercept behaviour with hooks, and integrate MCP servers, all in Python."

**Shape:** Python (MIT), 6.3k ⭐, actively maintained. Wraps the Claude Code CLI for programmatic use. Two APIs: `query()` (async generator, fire-and-forget) and `ClaudeSDKClient` (interactive session). Custom tools via `@tool` decorator compiled into in-process MCP servers — no subprocess overhead. `HookMatcher` intercepts `PreToolUse`/`PostToolUse` with deny decisions. Supports both in-process and external subprocess MCP servers. A2A: Claude can invoke peer Claude agents and integrate results. Claude Code CLI auto-bundled. TypeScript SDK also available (`anthropics/claude-agent-sdk-typescript`).

**Overlap with us:** This is the upstream API our `claude_sdk_executor.py` informally wraps. `ClaudeAgentOptions` fields (system_prompt, cwd, allowed_tools, permission_mode, max_turns, mcp_servers, hooks) map 1:1 to our workspace `config.yaml`. In-process MCP servers are what our `mcp_server.py` provides. The `HookMatcher` pattern is a more formal version of our `workspace-template/hooks/` scripts.

**Differentiation:** Single-agent programmatic library, not a multi-agent platform. No org hierarchy, no workspace registry, no canvas, no A2A mesh (only peer-to-peer Claude-to-Claude), no scheduling, no channels, no RBAC. It's the engine; Starfire is the car.

**Worth borrowing:**
- **In-process MCP servers** — `create_sdk_mcp_server()` eliminates subprocess startup latency. Migrate `mcp_server.py` to in-process to remove a class of startup bugs.
- **`HookMatcher` semantics** — named matchers keyed by lifecycle event with `permissionDecision: "deny"`. Adopt as the hook contract in `workspace-template/hooks/` so contributors familiar with the SDK find a compatible API.
- **`ClaudeAgentOptions` field set** — treat as the canonical list of tunable Claude Code parameters; audit our `config.yaml` against it, gaps are missing features.

**Terminology collisions:**
- "hooks" — SDK: `HookMatcher` objects. Ours: bash scripts in `hooks/`. Same concept, different implementation; bridge in docs.
- "tools" — SDK: MCP-backed Python functions. Ours: skills/plugins. Careful in any doc that mentions both.

**Signals to react to:**
- If Anthropic releases SDK v1.0 with stable API → `claude_sdk_executor.py` should target this explicitly rather than the bare CLI.
- If A2A support expands from peer-to-peer to mesh routing → Starfire's A2A layer may become redundant for Claude-only orgs; reassess.
- TypeScript SDK parity → evaluate as basis for our Node.js workspace runtime.

**Last reviewed:** 2026-04-15 · **Stars / activity:** 6.3k ⭐, MIT, active

---

### OpenAI Agents SDK — `openai/openai-agents-python`

**Pitch:** "A lightweight, powerful framework for multi-agent workflows — provider-agnostic, MCP-native, with built-in tracing, guardrails, HITL, and sandbox agents."

**Shape:** Python 99.7%, MIT, 20.8k ⭐, v0.14.1 April 15 2026 (updated today). Core primitives: `Agent` (instructions + tools + guardrails + handoffs), `Runner`, `Handoff` (agent-to-agent delegation), `Tool` (function or MCP endpoint), `Guardrail` (input/output validation). **Sandbox agents** for persistent workspace operations across extended tasks. Voice/realtime via `gpt-realtime-1.5`. Session management. HITL mechanisms. Provider-agnostic (100+ LLMs).

**Overlap with us:** Handoffs = our `delegate_task`. Agents-as-tools = our workspace-as-tool model. MCP native = our `mcp-server`. Built-in tracing = our `activity_logs` + Langfuse. Sandbox agents = our Docker workspaces. Guardrails = our `@requires_approval` gate. The conceptual model is nearly identical; the difference is execution infrastructure.

**Differentiation:** Agents run as Python objects in one process, not in separate containers with independent memory, scheduling, and channels. No visual canvas, no org marketplace, no WebSocket org chart, no RBAC governance, no Slack/Telegram/Discord integrations, no persistent workspace identity outside a running script.

**Worth borrowing:**
- **`Guardrail` as a first-class agent-level primitive** — input + output validation declared at the agent, not as external middleware. Move our approval flow toward declarative agent-level guardrails in `config.yaml`.
- **Tracing default-on** — built-in distributed tracing out of the box. Make Langfuse tracing default-on in `workspace-template/` rather than opt-in.

**Terminology collisions:**
- "handoff" — their agent delegation primitive. Ours: `delegate_task`. Same concept, different name — confusing in mixed codebases.
- "sandbox" — their persistent long-running agent workspace. Ours: Docker container per workspace. Doc disambiguation required.
- "agent" — in-process Python object vs. Docker container. Same word, very different operational model.

**Signals to react to:**
- If sandbox agents gain persistent cross-session memory → closes the biggest gap with Starfire; watch CHANGELOG closely.
- If OpenAI ships a canvas for multi-agent org hierarchy → direct Canvas competition from the framework with the most enterprise mindshare.
- If OpenAI publishes hosted "deploy SDK agents as a service" → Starfire's value narrows to governance, RBAC, and org-hierarchy; sharpen that messaging now.

**Last reviewed:** 2026-04-15 · **Stars / activity:** 20.8k ⭐, v0.14.1 April 15 2026

---

## Candidates to add (backlog)

Short-list of projects to write up next time someone has an hour:

- **LangGraph** (`langchain-ai/langgraph`) — we already support it as a
  runtime; worth a full entry for how their graph model compares to our
  workspace hierarchy.
- **CrewAI** (`crewaiinc/crewai`) — ditto.
- **DeepAgents** (`langchain-ai/deepagents`) — ditto; particularly their
  sub-agent feature that collides with our "skills" word.
- **OpenClaw** — check if this is still live post-Hermes rebrand; our
  adapter may need renaming.
- **Moltiverse / Moltbook** (`molti-verse.com`) — "social network for AI
  agents." Not a competitor; orthogonal ecosystem but worth tracking in
  case we want agent-to-agent discovery beyond a single org.
- **Temporal** (`temporalio/temporal`) — we already integrate; entry
  should cover when to lean on Temporal vs our in-house scheduling.
- **vercel-labs/skills** (`vercel-labs/skills`) — Vercel's companion skills
  standard to open-agents. Now that open-agents is a full entry, this repo
  deserves its own write-up as a pure skills-registry standard (separate
  from the agent runtime).
- **Block Goose** (`block/goose`) — local-first AI agent with MCP support
  from Block (Square). ~4.9k ⭐. Relevant for MCP tool-use patterns and
  local-execution model.
- **backnotprop/plannotator** — visual annotation tool for reviewing coding
  agent plans. ~4.2k ⭐. Relevant to our Canvas approval flow and plan
  review UX.
- **ressl/mcp-firewall** (`ressl/mcp-firewall`) — MCP security gateway:
  policy enforcement (OPA/Rego), threat detection (50+ injection patterns,
  PII, secrets), cryptographically signed audit trail, SIEM export,
  DORA/FINMA/SOC 2 compliance reports. AGPL-3.0, Python, only 5 ⭐ today
  but the concept maps directly to our compliance plugin gap (issue #256).
  Revisit when stars grow or a permissive fork appears.
- **Fission-AI/OpenSpec** (`Fission-AI/OpenSpec`) — spec-driven development
  for AI coding assistants. 40.2k ⭐, supports 21 tools incl. Claude Code.
  Delta specs for brownfield. Not core agent-infra but the planning artifact
  pattern (proposal.md + specs + design.md + tasks.md) is relevant to our
  PM workspace planning flow.
