# AQuA Architect

> AI Quality Engineering Architect for AI-assisted software development.


AQuA Architect is an engineering assistant that guides developers and QA engineers in designing, implementing and reviewing AI applications using the **AQuA (Architecting Quality for AI Applications)** methodology.


Unlike traditional coding assistants, AQuA Architect does not focus only on generating code.


It helps engineers build AI systems that are:


- Observable
- Testable
- Explainable
- Governable
- Continuously Improving


---


## Philosophy


Traditional testing asks


"Did the application return the correct answer?"


AQuA asks


"Did the application execute the correct workflow using the correct context, invoke the correct tools, produce sufficient evidence, and can we prove it?"


---


## Five Pillars


✔ Prevent

✔ Detect

✔ Govern

✔ Observe

✔ Learn


---


## Three Modes


Design


Design the testing strategy before writing code.


Build


Generate automation following AQuA principles.


Review


Review an AI application and identify architectural gaps.


---


## Features


- Shift Left validation

- AI evaluation strategy

- Confidence scoring

- HITL recommendations

- Telemetry guidance

- Golden Dataset strategy

- Architecture review

- Test completeness review

- Production readiness review


---


## Example


Developer


> Write tests for my AI Agent.


AQuA Architect


Instead of immediately writing code it asks


- Which LLM?

- Which orchestration framework?

- Which vector database?

- Which tools?

- Which business workflow?

- Which risks?


Only then does it recommend a testing strategy.


---


## Architecture


Prevent

↓

Detect

↓

Govern

↓

Observe

↓

Learn

↓

Golden Dataset

↓

Prevent


---


## Repository Structure


```
AQuA-Architect/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── CHANGELOG.md
├── assets/                       # Images and diagrams
├── examples/                     # Example implementations
│   ├── rag-agent.md
│   ├── chatbot.md
│   ├── multi-agent.md
│   ├── tool-calling.md
│   └── code-review.md
├── prompts/                      # Structured prompts per mode
│   ├── design.md
│   ├── build.md
│   └── review.md
└── skills/
    └── aqua-architect/           # Installable skill package
        ├── SKILL.md              # Main skill
        ├── modes.md              # Operating modes
        ├── checklist.md          # Quality checklist
        ├── architecture.md       # Architecture patterns
        ├── scoring.md            # Scoring rubric
        └── reference/            # Reference docs (per-pillar)
            ├── pillars.md
            ├── telemetry.md
            ├── golden-dataset.md
            ├── observability.md
            ├── confidence.md
            └── evaluation.md
```


## Installation


### Option A — Clone the repo and copy the skill


```bash
git clone https://github.com/sshamay/AQuA-Architect.git
mkdir -p ~/.config/opencode/skills/
cp -r AQuA-Architect/skills/aqua-architect ~/.config/opencode/skills/
```


### Option B — One-liner


```bash
mkdir -p ~/.config/opencode/skills/ && \
git archive --remote=https://github.com/sshamay/AQuA-Architect.git HEAD skills/aqua-architect | \
tar -x -C ~/.config/opencode/skills/
```


### Restart opencode


Skills are loaded when opencode starts. Quit and restart opencode after installing.


### Verify the skill is installed


Run the opencode command palette and look for the `aqua-architect` skill, or ask opencode directly:

> Do you have the AQuA Architect skill?


---


## Usage


AQuA Architect triggers automatically when you ask for quality engineering help on AI applications. It runs in three modes:

| Mode | When to use | What it does |
|------|-------------|--------------|
| Design | Before writing code | Asks about architecture, business, risk, telemetry, golden dataset — then designs the testing strategy |
| Build | During implementation | Guides code generation, suggests assertions, recommends telemetry |
| Review | On an existing system | Produces an architecture score, coverage report, missing pillars, top risks, ROI improvements |

Just ask normally — it loads itself when relevant:


### Design Mode

> I'm building a RAG agent for internal docs. Design a quality strategy for it.

The skill will ask clarifying questions (LLM, orchestration framework, vector DB, tools, workflow, risks) before recommending anything.


### Build Mode

> Write tests for my AI agent.

It will never jump straight to code. It first maps your architecture (which LLM? which framework? which tools?), then generates tests with proper assertions and telemetry.


### Review Mode

> Review my AI application for production readiness.

Produces:
- Architecture score (testability, observability, confidence, production readiness, regression readiness)
- Test coverage report
- Missing pillars (Prevent / Detect / Govern / Observe / Learn)
- Top risks
- ROI improvements


---


## GitHub Copilot


### Option A — Install as a Custom Agent (recommended)


GitHub Copilot supports custom agents defined as `.agent.md` files. Install at the user level to make AQuA Architect available in all your projects, or at the repository level to share with your team.


**User level** (available everywhere on your machine):

```bash
mkdir -p ~/.copilot/agents
cp skills/aqua-architect/agent-profile.md ~/.copilot/agents/aqua-architect.agent.md
```


**Repository level** (shared with your team via source control):

```bash
mkdir -p .github/agents
cp skills/aqua-architect/agent-profile.md .github/agents/aqua-architect.agent.md
```


### Option B — Install as a Skill


Copilot auto-discovers skills from `.github/skills/<name>/SKILL.md` (project) or `~/.copilot/skills/<name>/SKILL.md` (personal). The AQuA skill uses progressive disclosure — Copilot loads the pillar files only when they are relevant.


**Personal**:

```bash
mkdir -p ~/.copilot/skills
cp -r skills/aqua-architect ~/.copilot/skills/
```


**Project** (team-shared):

```bash
mkdir -p .github/skills
cp -r skills/aqua-architect .github/skills/
```


### Usage


Invoke the agent by name in Copilot Chat or Copilot CLI:

> @aqua-architect I'm building a RAG agent. Design a testing strategy for it.

Or let it trigger naturally on quality-engineering requests:

> Write tests for my AI agent.
> Review my AI application for production readiness.

The agent applies the AQuA methodology: it asks about architecture, business, risk, telemetry, and golden dataset before recommending anything, then finishes with an AQuA Coverage Report.


---


## License


MIT
