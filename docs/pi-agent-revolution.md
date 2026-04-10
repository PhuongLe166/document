# PI Agent Revolution: Building Customizable, Open-Source AI Coding Agents

> **Adapted for this documentation site.** Source material: blog article by **Atal Upadhyay**, dated **February 24, 2026** (“PI Agent Revolution…”). Commands and package names (e.g. `@pi-agent/cli`, `@pi-agent/sdk`) reflect the original article; verify against current official PI tooling and your stack (including `@mariozechner/*` if you use that ecosystem).

---

## Introduction: Why Your Tools Are Limiting Your Potential

If you have been building software recently, you may feel friction when an AI coding assistant does not match your workflow: too opinionated, wrong model, or too much abstraction over details you need to control.

Every engineer is limited by the tools they use. Tools shape what feels possible, workflow, mental model, and the ceiling of what you can build.

**Question to ask:** *How is my agentic coding tool limiting me?*

This guide is framed as a path from foundations through labs, production, strategy, and interview-style practice.

**Topics covered:**

- Conceptual foundations: what an “agent harness” is and why it can matter more than the raw model
- Tool comparison: Claude Code vs. PI Agent — philosophy, capabilities, trade-offs
- Hands-on labs: setup through meta-agent style patterns (as described in the source article)
- Customization: extensions, hooks, widgets, themes, prompts (article examples)
- Production: security, scalability, enterprise readiness framing
- Strategy: when to use which tool, hedging
- Interview-style prompts: questions and strong answer sketches

Whether you are early-career, mid-level, or leading teams, use the sections that match your depth.

---

## Part 1: Conceptual Foundations – Understanding the Agent Harness

### 1.1 What Is an Agent Harness, Really?

Many people focus on the model (Claude, GPT, Gemini, etc.). The model matters, but a deeper layer determines what the agent can do in practice:

**The agent harness** is the framework that connects the model to tools, manages conversation state, executes tools, and orchestrates the reasoning loop.

```text
┌─────────────────────────────────────────┐
│           THE AGENT STACK               │
├─────────────────────────────────────────┤
│  Model Layer                            │
│     • The LLM reasons and generates     │
│     • Claude, GPT, Gemini, open-source  │
│                                         │
│  Harness Layer  ← focus                 │
│     • Tool registration and execution   │
│     • Conversation / memory management  │
│     • Prompt templating and injection   │
│     • Event hooks and lifecycle control   │
│     • UI/UX rendering and interaction      │
│                                         │
│  Tool Layer                             │
│     • File ops (read/write/edit)        │
│     • Shell (bash)                      │
│     • APIs (MCP, custom)                │
│     • Domain skills                     │
│                                         │
│  User Layer                             │
│     • Terminal, IDE, web UI             │
│     • Commands, prompts, feedback        │
└─────────────────────────────────────────┘
```

**Key insight:** Two agents using the same model can behave very differently based on harness design. The harness shapes:

- Which tools exist
- How the agent chooses when to use them
- What it “sees” about its own actions
- Error handling and recovery
- The user experience

### 1.2 The Four Pillars of Agent Design

| Pillar | Key questions | Impact on UX |
|--------|----------------|--------------|
| **Philosophy** | Opinionated vs minimal? Safety enforced vs delegated? | Control vs guidance |
| **Observability** | What is visible about reasoning and tool runs? | Debugging, trust, learning |
| **Extensibility** | Custom tools, prompts, UI? | Specialization |
| **Model agnosticism** | Tied to one vendor vs many? | Cost, flexibility, future-proofing |

### 1.3 Why “Minimal” Can Be Powerful (article framing)

Article framing: PI Agent emphasizes *“If I don’t need it, it won’t be built.”*

Starts with four core tools (in the article’s narrative):

- **read** — read file contents  
- **write** — write new files  
- **edit** — modify existing files  
- **bash** — run shell commands  

Plus a very small system prompt (“helpful coding assistant; use these tools”).

**Why minimalism can help:**

- Less to learn up front  
- Clearer mental model  
- Less “baked in” that you cannot change  
- Faster iteration on what you truly need  

**Trade-off:** You must understand implications of filesystem and shell access.

### 1.4 The Customization Spectrum (extensions)

Composable extensions stack on a base agent, e.g.:

- Pure focus UI  
- Context footer (model + tokens)  
- Skills loader from multiple directories  
- Purpose widget  
- Theme cycling  
- Tool counter  
- Sub-agent patterns (via extensions)  
- Till-done task lists  
- Agent teams  

Each extension hooks lifecycle events; you mix and match for your workflow.

---

## Part 2: Claude Code vs. PI Agent – Comparison

### 2.1 Design Philosophy

**Claude Code (article view):** broad accessibility, guardrails, large opinionated prompt, polished UX, enterprise features.

**PI Agent (article view):** minimal prompt, “YOLO” default access unless you add restrictions, transparent internals, model-agnostic posture, extension-based customization.

### 2.2 Side-by-Side Feature Table (from article)

| Feature | Claude Code | PI Agent | Why it matters |
|---------|-------------|----------|----------------|
| Source | Closed source | Open source (MIT) cited in article | Auditability, forkability |
| System prompt | ~10k tokens, opinionated | ~200 tokens, minimal | Behavioral control |
| Default tools | 8+ | 4 core + custom | Bloat vs intentionality |
| Safety | Multiple modes | None by default in article | Speed vs control |
| Observability | More abstracted | Fully exposed in article framing | Debugging |
| Models | Anthropic-first in practice | Any via API/local in article | Flexibility |
| Customization | Config + limited hooks | TS extensions, many hooks | Depth |
| Sub-agents | Native patterns | Build via extensions | Convenience vs control |
| Enterprise | SOC2, audits, SSO | Experimental; built in-house | Regulated industries |
| Pinning | Limited per article | Full (open source) | Stability |

### 2.3 “Cancer of Success” Framing

Article note: successful commercial products optimize for growth, revenue, supportability, and pace — which can pull product design away from niche power-user needs. Open/composable stacks are positioned as a counter-strategy for the “long tail” of workflows.

### 2.4 When to Use Which

**Use Claude Code when:** onboarding, enterprise compliance, polished UX, team consistency, Anthropic-centric stack.

**Use PI Agent when:** you want control, non-Anthropic models, niche automation, version pinning, custom safety via your own layers.

**Hedge:** majority on one tool for day-to-day; minority on the other for experiments and flexibility.

---

## Part 3 – Lab 1: Getting Started (Beginner)

### 3.1 Prerequisites

- Node.js **18+**  
- npm or yarn, Git, editor  
- Model access: Anthropic / OpenAI / Google / local (Ollama, etc.)  
- Terminal (article mentions macOS/Linux or WSL)

### 3.2 Install (article examples)

```bash
npm install -g @pi-agent/cli
pi --version
```

From source (article):

```bash
git clone https://github.com/mariozhan/pi-agent.git
cd pi-agent
npm install
npm link
```

### 3.3 Environment (`.env` example from article)

```text
PI_MODEL=claude-sonnet-4
ANTHROPIC_API_KEY=sk-ant-...
# or other providers / Ollama as in original post
PI_TEMPERATURE=0.2
PI_MAX_TOKENS=4096
PI_EXTENSIONS_DIR=./my-extensions
```

**Security:** do not commit `.env`; add to `.gitignore`.

### 3.4 First session

```bash
cd ~/my-project
pi
```

Article-style UI sketch: version, model, context usage, prompt line.

### 3.5 Basic interactions

Examples: list files, create `hello.ts`, run `npm install` — agent uses read/write/edit/bash as appropriate.

### 3.6 Slash commands (article lists)

Examples cited: `/new`, `/compact`, `/resume`, `/fork`, `/tree`, `/tools`, `/context`, `/debug`, `/help`.

### 3.7 Agent loop (high level)

1. User input  
2. Prompt construction (system, history, tools, message)  
3. Model inference  
4. Tool execution  
5. Response synthesis  
6. UI update  
7. Loop or terminate  

### 3.8 Lab 1 reflection

Checklist: install, basic loop, four tools, slash commands. Next: extensions.

---

## Part 4 – Lab 2: Extensions (Intermediate)

### 4.1 What an extension does

Hooks lifecycle, registers commands/tools/UI, may adjust prompts or widgets.

**Skeleton pattern (article style):**

```ts
// Illustrative — API per @pi-agent/sdk in original article
import { Extension } from "@pi-agent/sdk";

export default class MyExtension extends Extension {
  name = "my-extension";
  version = "1.0.0";

  async onInput(input: string, context: AgentContext): Promise<void> {
    /* ... */
  }
  async onToolCall(tool: string, params: unknown, context: AgentContext): Promise<void> {
    /* ... */
  }
  async onOutput(output: string, context: AgentContext): Promise<void> {
    /* ... */
  }

  registerCommands() {
    return [
      {
        name: "my-command",
        description: "Does something cool",
        handler: async (_args: string[]) => {
          /* ... */
        },
      },
    ];
  }

  registerTools() {
    return [
      {
        name: "my-tool",
        description: "A custom tool",
        schema: {
          /* JSON Schema */
        },
        handler: async (_params: unknown) => {
          /* ... */
        },
      },
    ];
  }
}
```

### 4.2–4.7 Stacking examples (article)

Article walks through enabling extensions such as:

- `pi -e pure-focus`  
- `pi -e pure-focus -e context-footer`  
- Skills directories via `pi.config.ts`  
- `purpose-widget`, theme cycling (**Theme Cylinder** in corrected spelling), `tool-counter`  

### 4.8 Custom `/ping` extension (article)

Article provides bash `mkdir`, `npm init`, `tsc`, and a minimal `PingExtension` class; run with `pi -e ./dist/index.js`.

### 4.9 Lab 2 reflection

Extensions, UI/footer/themes, skills, custom commands. Next: multi-agent patterns in article narrative.

---

## Part 5 – Lab 3: Multi-Agent Patterns (Advanced)

### 5.1 Why multiple agents

**Limits of one agent:** context pressure, specialization, blocking, mixed debugging.

**Benefits:** specialization, parallelism, isolation, scale-out story.

### 5.2 Sub-agent extension (article)

Example: `pi -e sub-agent-support`, `/sub "..."` with widget UI for status/result.

### 5.3 Agent teams (`teams.yaml` – article excerpt)

YAML defines teams like `research-team` and `build-team` with named agents, roles, models, tools. Enable: `pi -e agent-teams --config teams.yaml`. Commands: `/teams`, `/team research-team`, then natural-language orchestration (as described in article).

### 5.4 Agent chains (`chains.yaml`)

Sequential pipeline (`triple-scout` example). `pi -e agent-chains --config chains.yaml`, `/chain triple-scout --query "..."`.

### 5.5 Till-done list

Forces task list + evidence before certain actions; article shows blocked action → add task → proceed.

### 5.6 Damage control

`pi -e damage-control` — block patterns (`rm -rf`, `chmod 777`, `curl | sh`, etc.), confirm sensitive ops, allowlist safe commands. Config example in article: `damage-control.config.ts`.

### 5.7 System prompt selector

`pi -e system-select` — `/system` lists personas; switch e.g. to `browser-agent` (article narrative).

### 5.8 Lab 3 reflection

Sub-agents, teams, chains, till-done, safety gating, personas. Next: meta-agents.

---

## Part 6 – Lab 4: Meta-Agents (Expert)

### 6.1 Meta-agent idea

Generate/configure agents, compose specialists, analyze performance, improve over time (article’s framing).

### 6.2 Experts (`experts.yaml`)

Defines experts like `prompt-engineer`, `tool-builder`, `harness-architect` with prompts and tool lists. `pi -e meta-agent --experts experts.yaml` (article).

### 6.3 Agent factory (TypeScript sketch)

Article shows `SecurityAgentFactory` generating configs, tools, extensions programmatically.

### 6.4 Orchestrator pattern

`OrchestratorExtension` classifies tasks, gets/creates worker agents, delegates, synthesizes responses.

### 6.5 Self-improvement loop

`LearningExtension` stores feedback, triggers improvement when ratings low — explicit vs implicit vs automated evaluation (article lists strategies).

### 6.6 Combined system layout

Article project tree: `config/`, `extensions/`, `skills/`, `tools/`, `pi.config.ts` with agent, extensions, experts, teams, chains, safety, learning blocks.

### 6.7 Lab 4 reflection

Experts, factories, orchestrators, learning, integrated config.

---

## Part 7: Production Considerations

### 7.1 Security beyond YOLO

- Least privilege (filesystem, network, command allow/deny)  
- Audit logging with redaction  
- Optional hash-chained / tamper-evident logs (article sketch)

### 7.2 Scalability

Load balancer → multiple agent pods → shared vector store, audit, feedback DB, inference endpoint (diagram in article).

**Kubernetes example (corrected `metadata`):** the original paste used `meta`; valid Kubernetes uses `metadata`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi-agent
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pi-agent
  template:
    metadata:
      labels:
        app: pi-agent
    spec:
      containers:
        - name: agent
          image: my-registry/pi-agent:1.2.3
          env:
            - name: PI_MODEL
              value: "claude-sonnet-4"
            - name: ANTHROPIC_API_KEY
              valueFrom:
                secretKeyRef:
                  name: api-secrets
                  key: anthropic-key
          volumeMounts:
            - name: workspace
              mountPath: /workspace
          securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false
      volumes:
        - name: workspace
          emptyDir: {}
```

### 7.3 Enterprise gap / hybrid workflow

Article: PI for R&D/experimentation; commercial tool for compliance-heavy production; migration path between them.

### 7.4 Cost management

Budget extension sketch; model tiering YAML; caching extension; context trimming extension (all from article narratives).

---

## Part 8: Strategic Guidance

### 8.1 Hedge (80/20)

Majority on polished/enterprise tool for daily work; minority on flexible stack for experiments — reduces lock-in.

### 8.2 Decision matrix (article)

| Task characteristic | Prefer Claude Code | Prefer PI Agent |
|--------------------|--------------------|-----------------|
| Team size | Large / consistency | Small / flexible |
| Compliance | Regulated | Internal-only |
| Model preference | Anthropic OK | Multi-provider / local |
| Customization | Prompts/skills enough | Deep UI/tools/orchestration |
| Stability | SLAs, pin versions | Can tolerate edge |
| Budget | Standard vendor pricing | Optimize models |
| Innovation | Stable patterns | Novel agent designs |

### 8.3 Migration outlines

Article lists phased moves **Claude → PI** and **PI → Claude** (parallel run, port skills, migrate workflows, optimize).

### 8.4 Future-proofing

Abstract `Agent` interface behind implementations; invest in portable prompt skills; monitor ecosystem; contribute extensions back.

---

## Part 9: Interview Questions – Sketches

### Q1 – Model vs harness?

**Strong answer sketch:** Model = generation; harness = tools, state, execution loop, UX. Same model, different harness ⇒ different behavior.

### Q2 – Minimal vs opinionated harness?

**Sketch:** Flexibility and clarity vs needing more expertise; fewer surprises vs more assembly.

### Q3 – Model agnostic?

**Sketch:** Swap providers for cost, privacy, performance, avoid lock-in.

### Q4 – Add custom API tool?

**Sketch:** Schema + handler + register in extension; validate input, rate limits, sanitize output.

### Q5 – Agent stuck in loop?

**Sketch:** Logging, iteration caps, stuck detection, better tool descriptions, reflection step; check extensions for side effects.

### Q6 – Till-done pattern?

**Sketch:** Task state machine + UI widget + tool-call gate + evidence before “done”.

### Q7 – Multi-agent code review architecture?

**Sketch:** Triage → parallel specialists → synthesis → human escalation for high risk; consensus for auto-approve; audit trail.

### Q8 – Cold start, no feedback?

**Sketch:** Rule-based validators, transfer prompts, high human oversight early, active learning, sandbox simulation; metrics from day one.

### Q9 – Ethics of feedback loops?

**Sketch:** Manipulation, filter bubbles, accountability, consent; transparency, opt-out, governance; open extensions for inspectable guardrails.

---

## Part 10: Conclusion

Recap: harness concepts, comparison, labs, customization, production, strategy, interview practice.

**Next steps (article):** install and run; add one extension; one custom tool; try orchestration patterns; document learnings.

**Mindset shift (article):** from “use as designed” toward “build what you need”; transparency and responsibility.

---

## Appendix A: Extension Cheat Sheet

```ts
import { Extension, Command, Tool } from "@pi-agent/sdk";

export default class MyExtension extends Extension {
  name = "my-extension";
  version = "1.0.0";

  async onInput(_input: string, _context: AgentContext): Promise<void> {}
  async onToolCall(_tool: string, _params: unknown, _context: AgentContext): Promise<void> {}
  async onOutput(_output: string, _context: AgentContext): Promise<void> {}
  async onAgentEnd(_context: AgentContext): Promise<void> {}

  registerCommands(): Command[] {
    return [
      {
        name: "my-command",
        description: "What it does",
        handler: async (_args, _context) => "Response to user",
      },
    ];
  }

  registerTools(): Tool[] {
    return [
      {
        name: "my-tool",
        description: "What it does",
        schema: {
          /* JSON Schema */
        },
        handler: async (_params, _context) => ({ result: "data" }),
      },
    ];
  }

  renderFooter(context: AgentContext): string {
    return `Custom: ${context.someValue}`;
  }

  registerWidgets() {
    return [
      {
        id: "my-widget",
        render: (_context: AgentContext) => "Content here",
        updateInterval: 1000,
      },
    ];
  }
}
```

---

## Appendix B: Resource Toolkit (from article; verify URLs)

- GitHub (article): `https://github.com/mariozhan/pi-agent`  
- Docs (article): `https://pi.dev/docs`  
- Extensions (article): `https://github.com/pi-agent/extensions`  
- Community (article): `https://discord.gg/pi-agent`  

Also cited: LangChain/LangGraph, Claude Code docs, AutoGen, CrewAI, NIST AI RMF, OWASP LLM top 10, observability and testing stacks.

---

## Appendix C: Troubleshooting (summary)

| Issue | Ideas |
|--------|--------|
| Extension not loading | Default export, ES2022 build, path in `PI_EXTENSIONS_DIR` or `-e` |
| Looping agent | Verbose logs, max iterations, stuck detection, clearer tools |
| Silent tool failure | try/catch, structured errors, timeouts, isolate tests |
| Context filling fast | Compression, fewer tools in prompt, sub-agents, bigger window |
| Theme not applying | Required theme keys, render hooks, isolate extensions |

---

## Appendix D: Glossary

| Term | Definition |
|------|------------|
| Agent harness | Connects LLM to tools; manages state and loop |
| Extension | TS module hooking PI Agent lifecycle |
| Hook | Lifecycle callback (`onInput`, `onToolCall`, …) |
| Widget | Persistent UI element |
| Skill | Reusable prompt template |
| Tool | Callable capability (read, bash, custom, …) |
| YOLO mode | Article term for permissive default access |
| Till-done | Enforced task list pattern |
| Meta-agent | Builds/manages other agents |
| Agent chain | Sequential pipeline |
| Agent team | Parallel/specialized group |
| Model agnostic | Not tied to one LLM vendor |

---

*End of adapted article. For authoritative API details, follow official PI documentation and the packages you actually depend on.*
