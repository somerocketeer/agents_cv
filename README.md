# Agent CV

A comprehensive portfolio of AI agent systems and pipelines for executive assistance, documentation, marketing content production, and blog publishing.

## Quick Stats

- **Total Agents**: 90 across 4 domains
- **Orchestrators**: 14 (1 ops, 3 docs, 9 marketing, 1 blog & design)
- **Subagents**: 70 specialized workers
- **Standalone**: 6 utility agents
- **Infrastructure**: Memory systems, MCP tools, sandboxes, style guides

---

## Navigation

### By Domain
- **[Ops (Silas)](./domains/ops/)** - Executive assistant with persistent memory
- **[Documentation](./domains/documentation/)** - End-to-end docs lifecycle
- **[Marketing](./domains/marketing/)** - B2B content production at scale
- **[Blog & Design](./domains/blog-design/)** - Personal blog + neobrutalist design system

### By Topic
- **[Patterns](./patterns/)** - Cross-cutting architectural patterns
- **[Examples](./examples/)** - Real usage examples
- **[Reference](./reference/)** - Quick lookup guides

---

## Quick Start

### Ops (Silas)
```bash
# Morning briefing
/morning

# Meeting preparation
/prep "Meeting with Sarah about Q1 planning"

# Memory operations
memory.observe("correction", {what_was_wrong: "...", what_was_expected: "..."})
```

### Documentation
```bash
# Draft from scratchpad
draft-orchestrator --input scratchpad.md --output draft.mdx

# Copyedit existing docs
editor-orchestrator --target docs/guide.mdx

# Verify with Playwright
playwright-orchestrator --workflow workflow.json
```

### Marketing
```bash
# Blog post (800-1,400 words)
blog-orchestrator --project tokenization1

# Whitepaper (2,000-7,500 words)
whitepaper-orchestrator --project TDP --mode extended

# LinkedIn campaign
linkedin-orchestrator --project signing
```

### Blog & Design
```bash
# Full blog pipeline
blog_interviewer → blog_planner → blog_drafter → blog_editor → blog_publisher

# Design audit
design_auditor --component Button.tsx
```

---

## Architecture Overview

### Memory Stack (Ops)
```
┌─────────────────────────────────────────────────────────────┐
│             MCP Server (mcp_memory_server.py)               │
│   silas_memory_* tools exposed to agents via MCP protocol   │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│   Postgres    │     │    Qdrant     │     │    Neo4j      │
│  Event Log    │     │ Vector Store  │     │ Knowledge     │
│  (source of   │     │ (embeddings)  │     │ Graph (facts) │
│   truth)      │     │               │     │               │
└───────────────┘     └───────────────┘     └───────────────┘
```

### Documentation Pipeline
```
Scratchpad → EXTRACT → ANALYZE → DRAFT → INJECT → VALIDATE → PUBLISH
                ↓          ↓         ↓        ↓         ↓
           code blocks  detective  writer  inject   recovery
                       architect
```

### Marketing Pipeline
```
Scratchpad → Fact Bank → Persona Brief → Market Intel → Plan → Draft → Edit → Publish
                ↓             ↓              ↓
           fact_bank_    persona_      marketing_
           builder      calibrator  research_scout
```

### Blog Pipeline
```
Interview → Plan → Draft → Edit → Publish
    ↓         ↓      ↓       ↓       ↓
interviewer planner drafter editor publisher
```

---

## Key Patterns

### Phase-Based Orchestration
All orchestrators follow sequential phase execution with fail-fast behavior:
1. Setup/validation
2. Evidence gathering
3. Planning
4. Execution
5. Quality gates
6. Publishing

### Model Routing
- **GPT-5.2**: Strategic planning, argument structure
- **Claude Opus 4.5**: Primary orchestration, deep analysis
- **Claude Sonnet 4.5**: Most subagents, prose quality
- **Gemini Flash 3**: Deterministic operations, fact extraction

### Temperature Control
- **0.0-0.05**: Deterministic (architecture, recovery, publishing)
- **0.1-0.15**: Precise analysis (planning, research)
- **0.2-0.3**: Balanced (editing, meeting prep)
- **0.4-0.5**: Creative (drafting, communications)

### Evidence Discipline
- **Fact IDs**: Unique identifiers for all claims
- **Citation synthesis**: Never cite without explaining
- **Persona transformation**: Internal facts → reader-facing prose

### Anti-LLM Detection
- **Hard blocks**: 50+ banned phrases
- **Frequency limits**: Em dashes (1-2), rhetorical questions (1-2)
- **Structural tells**: No hedge stacking, filler verbs, empty connectives

---

## Domain Summaries

### Ops (Silas) - 8 agents
Executive assistant with persistent memory, CRM, and autonomous task management. Runs scheduled briefings, meeting prep, and proactive suggestions.

**Key Features**:
- Hybrid recall (Postgres + Qdrant + Neo4j)
- Self-tuning configuration
- Energy-type task prioritization
- 7 specialized subagents (research, drafter, memory, meeting-prep, comms, suggestions, weekly-review)

### Documentation - 19 agents
End-to-end documentation lifecycle from scratchpad to verified, published docs with Playwright testing.

**Key Features**:
- Blueprint-driven drafting
- Gap detection (docs-detective)
- Code placeholder injection
- Playwright workflow verification

### Marketing - 51 agents
B2B content production with evidence-backed thought leadership across 8 content types.

**Key Features**:
- 9 orchestrators (blog, whitepaper, datasheet, solution-brief, newsletter, linkedin, customer-success, video-integration, integration-outline)
- Shared subagents (fact_bank_builder, persona_calibrator, marketing_research_scout)
- Tiered research (T1-T4 confidence)
- Strict anti-LLM discipline

### Blog & Design - 12 agents
Personal blog production with teaching-oriented voice and neobrutalist design system maintenance.

**Key Features**:
- Interview-driven content gathering
- Destination-first structure
- MDX component builder
- Design system compliance verification

---

## Quick Links

- [Agent Index](./reference/agent-index.md) - Alphabetical agent list
- [Model Assignments](./reference/model-assignments.md) - Model routing reference
- [Tool Reference](./reference/tool-reference.md) - MCP tools & sandboxes
- [Patterns](./patterns/) - Cross-cutting patterns

---

## Documentation Structure

```
agents_cv/
├── domains/           # Domain-specific documentation
│   ├── ops/
│   ├── documentation/
│   ├── marketing/
│   └── blog-design/
├── patterns/          # Cross-cutting patterns
├── examples/          # Real usage examples
├── diagrams/          # Visual workflows
└── reference/         # Quick lookup guides
```

---

## Contributing

This is a living document. Agent definitions and patterns evolve based on real-world usage.

---

## License

MIT
