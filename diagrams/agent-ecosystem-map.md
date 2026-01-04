# Agent Ecosystem Map

Visual representation of all agents and their relationships across domains.

---

## Complete Ecosystem Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          AGENT ECOSYSTEM (~90 agents)                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                ┌─────────────────────┼─────────────────────┐
                │                     │                     │
                ▼                     ▼                     ▼
        ┌───────────────┐     ┌───────────────┐     ┌───────────────┐
        │   OPS (8)     │     │  DOCS (19)    │     │ MARKETING(60+)│
        │   Silas       │     │  Pipeline     │     │  Content      │
        └───────────────┘     └───────────────┘     └───────────────┘
                │                     │                     │
                ▼                     ▼                     ▼
        ┌───────────────┐     ┌───────────────┐     ┌───────────────┐
        │ BLOG+DESIGN   │     │               │     │               │
        │    (12)       │     │               │     │               │
        └───────────────┘     └───────────────┘     └───────────────┘
```

---

## Ops Domain - Silas Executive Assistant

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              SILAS (Primary)                                 │
│                         Claude Opus 4.5 | temp 0.2                          │
│                                                                              │
│  Capabilities: Memory, Calendar, Email, Todos, Reminders, Jira/Confluence   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ delegates to
                                      │
        ┌─────────────┬───────────────┼───────────────┬─────────────┐
        │             │               │               │             │
        ▼             ▼               ▼               ▼             ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   research   │ │   drafter    │ │    memory    │ │ meeting-prep │ │    comms     │
│  Sonnet 4.5  │ │  Sonnet 4.5  │ │  Sonnet 4.5  │ │  Sonnet 4.5  │ │  Sonnet 4.5  │
│   temp 0.3   │ │   temp 0.5   │ │   temp 0.15  │ │   temp 0.2   │ │   temp 0.4   │
│              │ │              │ │              │ │              │ │              │
│ Deep         │ │ Long-form    │ │ Learning     │ │ Briefings    │ │ Quick        │
│ exploration  │ │ writing      │ │ cycles       │ │ Follow-ups   │ │ messages     │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
                                      │
                                      │ uses
                                      ▼
                        ┌──────────────────────────────┐
                        │      MEMORY STACK            │
                        │                              │
                        │  ┌──────────┐ ┌──────────┐  │
                        │  │ Postgres │ │  Qdrant  │  │
                        │  │Event Log │ │  Vector  │  │
                        │  └──────────┘ └──────────┘  │
                        │  ┌──────────┐               │
                        │  │  Neo4j   │               │
                        │  │Knowledge │               │
                        │  │  Graph   │               │
                        │  └──────────┘               │
                        └──────────────────────────────┘
```

---

## Documentation Domain

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DOCUMENTATION PIPELINE                               │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                ┌─────────────────────┼─────────────────────┐
                │                     │                     │
                ▼                     ▼                     ▼
        ┌───────────────┐     ┌───────────────┐     ┌───────────────┐
        │    DRAFT      │     │    EDITOR     │     │  PLAYWRIGHT   │
        │ ORCHESTRATOR  │     │ ORCHESTRATOR  │     │ ORCHESTRATOR  │
        │  Sonnet 4.5   │     │  Sonnet 4.5   │     │  Sonnet 4.5   │
        │   temp 0.05   │     │   temp 0.05   │     │   temp 0.05   │
        └───────────────┘     └───────────────┘     └───────────────┘
                │                     │                     │
                │                     │                     │
        ┌───────┴───────┐     ┌───────┴───────┐     ┌───────┴───────┐
        │               │     │               │     │               │
        ▼               ▼     ▼               ▼     ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  detective   │ │  architect   │ │  copyedit    │ │    parser    │ │   resolver   │
│  Sonnet 4.5  │ │  Sonnet 4.5  │ │  Sonnet 4.5  │ │  Sonnet 4.5  │ │  Sonnet 4.5  │
│   temp 0.15  │ │   temp 0.0   │ │   temp 0.05  │ │   temp 0.05  │ │   temp 0.05  │
│              │ │              │ │              │ │              │ │              │
│ Gap analysis │ │ Blueprints   │ │ Style audit  │ │ Docs→JSON    │ │ Selectors    │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
        │               │                                 │               │
        ▼               ▼                                 ▼               ▼
┌──────────────┐ ┌──────────────┐                 ┌──────────────┐ ┌──────────────┐
│    writer    │ │   recovery   │                 │   debugger   │ │ doc-updater  │
│  Sonnet 4.5  │ │  Sonnet 4.5  │                 │  Sonnet 4.5  │ │  Sonnet 4.5  │
│   temp 0.10  │ │   temp 0.0   │                 │   temp 0.10  │ │   temp 0.10  │
│              │ │              │                 │              │ │              │
│ Draft prose  │ │ Fix errors   │                 │ Analyze fail │ │ Sync docs    │
└──────────────┘ └──────────────┘                 └──────────────┘ └──────────────┘

        ┌───────────────────────────────────────────────────────────┐
        │                  STANDALONE AGENTS                         │
        ├───────────────┬───────────────┬───────────────┬───────────┤
        │  copy-editor  │   detective   │  reference-   │  docflow  │
        │               │  (standalone) │   generator   │           │
        │  Sonnet 4.5   │  Sonnet 4.5   │  Sonnet 4.5   │ Sonnet4.5 │
        │   temp 0.05   │   temp 0.15   │   temp 0.10   │ temp 0.10 │
        └───────────────┴───────────────┴───────────────┴───────────┘
```

---

## Marketing Domain

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MARKETING CONTENT PRODUCTION                         │
│                              8 Orchestrators                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
        ┌─────────────┬───────────────┼───────────────┬─────────────┐
        │             │               │               │             │
        ▼             ▼               ▼               ▼             ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│     BLOG     │ │  WHITEPAPER  │ │  DATASHEET   │ │   SOLUTION   │ │  NEWSLETTER  │
│ ORCHESTRATOR │ │ ORCHESTRATOR │ │ ORCHESTRATOR │ │    BRIEF     │ │ ORCHESTRATOR │
│  Sonnet 4.5  │ │  Sonnet 4.5  │ │  Sonnet 4.5  │ │ ORCHESTRATOR │ │  Sonnet 4.5  │
│   temp 0.05  │ │   temp 0.05  │ │   temp 0.05  │ │  Sonnet 4.5  │ │   temp 0.05  │
│              │ │              │ │              │ │   temp 0.05  │ │              │
│ 800-1,400    │ │ 2,000-7,500  │ │ Procurement  │ │  Executive   │ │  Insight-led │
│    words     │ │    words     │ │    ready     │ │    ready     │ │   updates    │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘

        ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
        │   LINKEDIN   │ │   CUSTOMER   │ │     VIDEO    │
        │ ORCHESTRATOR │ │    SUCCESS   │ │ INTEGRATION  │
        │  Sonnet 4.5  │ │ ORCHESTRATOR │ │ ORCHESTRATOR │
        │   temp 0.05  │ │  Sonnet 4.5  │ │  Sonnet 4.5  │
        │              │ │   temp 0.05  │ │   temp 0.05  │
        │ 1-4 posts    │ │   Playbooks  │ │  Walkthrough │
        └──────────────┘ └──────────────┘ └──────────────┘
                                      │
                                      │ all use
                                      ▼
        ┌─────────────────────────────────────────────────────────┐
        │              SHARED SUBAGENTS (Universal)                │
        ├───────────────┬───────────────┬───────────────────────┐ │
        │ fact_bank_    │  persona_     │ marketing_research_   │ │
        │   builder     │  calibrator   │       scout           │ │
        │ Gemini Flash  │  Sonnet 4.5   │    Sonnet 4.5         │ │
        │   temp 0.05   │   temp 0.10   │     temp 0.15         │ │
        │               │               │                       │ │
        │ Extract facts │ Map to        │ Web search            │ │
        │ with IDs      │ personas      │ intelligence          │ │
        └───────────────┴───────────────┴───────────────────────┘ │
        └─────────────────────────────────────────────────────────┘
                                      │
                                      │ then
                                      ▼
        ┌─────────────────────────────────────────────────────────┐
        │           PIPELINE-SPECIFIC SUBAGENTS (per type)         │
        │                                                          │
        │  planner → drafter → editor → publisher/finalizer        │
        │  GPT-5.2   Sonnet    Sonnet   Gemini Flash              │
        │  temp 0.1  temp 0.28 temp 0.15 temp 0.05                │
        │                                                          │
        │  Argument   Synthesis  Polish +   Transform to          │
        │  structure  first      validate   publication           │
        └─────────────────────────────────────────────────────────┘
```

---

## Blog & Design Domain

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         BLOG & DESIGN SYSTEM                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                ┌─────────────────────┼─────────────────────┐
                │                     │                     │
                ▼                     ▼                     ▼
        ┌───────────────┐     ┌───────────────┐     ┌───────────────┐
        │     BLOG      │     │     BLOG      │     │     DESIGN    │
        │   PIPELINE    │     │  COPYWRITER   │     │  ORCHESTRATOR │
        │   (5 stages)  │     │  Sonnet 4.5   │     │   Opus 4.5    │
        │               │     │   temp 0.25   │     │   temp 0.15   │
        └───────────────┘     └───────────────┘     └───────────────┘
                │                                            │
                │                                            │
        ┌───────┴───────┬───────────────┬───────────────┐   │
        │               │               │               │   │
        ▼               ▼               ▼               ▼   ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ interviewer  │ │   planner    │ │   drafter    │ │    editor    │ │  publisher   │
│  Sonnet 4.5  │ │   GPT-5.2    │ │  Sonnet 4.5  │ │   Opus 4.5   │ │   GPT-5.1    │
│   temp 0.3   │ │   temp 0.1   │ │   temp 0.28  │ │   temp 0.15  │ │   temp 0.05  │
│              │ │              │ │              │ │              │ │              │
│ Context      │ │ Destination  │ │ Teaching     │ │ 3-pass edit  │ │ MDX format   │
│ gathering    │ │ first        │ │ voice        │ │ voice/meat/  │ │ + linking    │
│              │ │ outline      │ │              │ │ anti-LLM     │ │              │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
        │                                 │               │
        │                                 │               │
        ▼                                 ▼               ▼
┌──────────────┐                 ┌──────────────┐ ┌──────────────┐
│    voice     │                 │     mdx      │ │   design     │
│    check     │                 │  component   │ │   auditor    │
│ interviewer  │                 │   builder    │ │  Sonnet 4.5  │
│  Sonnet 4.5  │                 │ Gemini Flash │ │   temp 0.2   │
│   temp 0.3   │                 │   temp 0.15  │ │              │
│              │                 │              │ │ UI/UX        │
│ Gap filling  │                 │ Interactive  │ │ analysis     │
└──────────────┘                 │ components   │ └──────────────┘
                                 └──────────────┘         │
                                                          │
                                         ┌────────────────┴────────────────┐
                                         │                                 │
                                         ▼                                 ▼
                                 ┌──────────────┐                 ┌──────────────┐
                                 │   design     │                 │   design     │
                                 │ implementer  │                 │   reviewer   │
                                 │  Sonnet 4.5  │                 │  Sonnet 4.5  │
                                 │   temp 0.15  │                 │   temp 0.1   │
                                 │              │                 │              │
                                 │ Code changes │                 │ Verify a11y  │
                                 └──────────────┘                 └──────────────┘
```

---

## Cross-Domain Patterns

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         UNIVERSAL PATTERNS                                   │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│  PHASE-BASED ORCHESTRATION                                               │
│                                                                          │
│  Setup → Evidence → Planning → Execution → Validation → Publishing      │
│                                                                          │
│  - Sequential execution                                                  │
│  - Fail-fast behavior                                                    │
│  - Validation gates                                                      │
│  - Recovery attempts                                                     │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│  MODEL ROUTING                                                           │
│                                                                          │
│  GPT-5.2      → Planning (temp 0.1)                                      │
│  Claude Opus  → Primary orchestration (temp 0.15-0.2)                    │
│  Claude Sonnet→ Most subagents (temp 0.05-0.5)                           │
│  Gemini Flash → Deterministic ops (temp 0.05)                            │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│  EVIDENCE DISCIPLINE                                                     │
│                                                                          │
│  Fact IDs → Citation synthesis → Persona transformation                 │
│                                                                          │
│  - Unique identifiers for all claims                                    │
│  - Never cite without explaining                                        │
│  - Internal facts → reader-facing prose                                 │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│  ANTI-LLM DETECTION                                                      │
│                                                                          │
│  - Hard blocks: 50+ banned phrases                                      │
│  - Frequency limits: Em dashes (1-2), rhetorical questions (1-2)        │
│  - Structural tells: No hedge stacking, filler verbs                    │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Agent Count by Type

```
┌─────────────────────────────────────────────────────────────────┐
│  AGENT DISTRIBUTION                                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Orchestrators:  11  ████████████                              │
│  Subagents:      73  ████████████████████████████████████████  │
│  Standalone:      6  ██████                                    │
│                                                                 │
│  TOTAL:          90                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Model Distribution

```
┌─────────────────────────────────────────────────────────────────┐
│  MODEL USAGE                                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Claude Sonnet 4.5:  60  ████████████████████████████████████  │
│  Claude Opus 4.5:    15  ███████████████                       │
│  GPT-5.2:            10  ██████████                            │
│  Gemini Flash 3:     10  ██████████                            │
│  GPT-5.1:             2  ██                                    │
│  Gemini 3 Pro:        1  █                                     │
│                                                                 │
│  TOTAL:              90                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## Temperature Distribution

```
┌─────────────────────────────────────────────────────────────────┐
│  TEMPERATURE RANGES                                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  0.0-0.05 (Deterministic):  25  █████████████████████████      │
│  0.1-0.15 (Precise):        30  ██████████████████████████████ │
│  0.2-0.3 (Balanced):        25  █████████████████████████      │
│  0.4-0.5 (Creative):        10  ██████████                     │
│                                                                 │
│  TOTAL:                     90                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Related

- **[Agent Index](../reference/agent-index.md)** - Alphabetical agent list
- **[Model Assignments](../reference/model-assignments.md)** - Model routing reference
- **[Patterns](../patterns/)** - Cross-cutting patterns
- **[Examples](../examples/)** - Real usage examples
