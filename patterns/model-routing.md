# Model Routing Pattern

Strategy for selecting the right model for each agent based on task characteristics.

---

## Core Principle

Match model capabilities to task requirements:
- **Reasoning depth** → Claude Opus 4.5
- **Prose quality** → Claude Sonnet 4.5
- **Logical structure** → GPT-5.2
- **Deterministic output** → Gemini Flash 3

---

## Decision Framework

```
Is this task...
├── Primary orchestration or deep analysis?
│   └── Claude Opus 4.5
├── Planning or argument structuring?
│   └── GPT-5.2
├── Deterministic/structural?
│   └── Gemini Flash 3
└── General subagent work?
    └── Claude Sonnet 4.5
```

---

## Model Profiles

### Claude Opus 4.5

**Best For**:
- Primary orchestrators requiring judgment
- Deep analysis and critical thinking
- Voice analysis and style matching
- High-stakes decisions

**Characteristics**:
- Superior reasoning
- Excellent context maintenance
- Strong voice analysis
- Higher cost, worth it for critical paths

**Agents**: silas, blog_editor, blog_designer

---

### Claude Sonnet 4.5

**Best For**:
- Most subagent work
- Prose generation
- Research and analysis
- General-purpose tasks

**Characteristics**:
- Excellent prose quality
- Good analytical depth
- Balanced cost/quality
- Reliable and consistent

**Agents**: 60+ across all domains (drafters, editors, researchers)

---

### GPT-5.2

**Best For**:
- Strategic planning
- Argument construction
- Outline generation
- Logical structuring

**Characteristics**:
- Superior logical structure
- Excellent at thesis development
- Consistent planning quality
- Strong counterargument handling

**Agents**: All *_planner subagents (blog_planner, whitepaper_planner, etc.)

---

### GPT-5.1

**Best For**:
- Publishing transforms
- Format conversion
- Deterministic output

**Characteristics**:
- Highly deterministic
- Reliable formatting
- Good for final passes

**Agents**: blog_publisher (Blog & Design)

---

### Gemini Flash 3

**Best For**:
- Fact extraction
- Final validation
- Technical operations
- High-volume tasks

**Characteristics**:
- Fast and cost-effective
- Excellent structured output
- Reliable deterministic tasks
- Good for batch operations

**Agents**: fact_bank_builder, *_finalizer subagents, video_tts, video_renderer

---

### Gemini 3 Pro

**Best For**:
- Long-form content assembly
- Extended context handling
- Section stitching

**Characteristics**:
- Handles very long contexts
- Good coherence across sections
- Cost-effective for long-form

**Agents**: whitepaper_assembler

---

## Temperature Mapping

| Task Type | Temperature | Examples |
|-----------|-------------|----------|
| Structural | 0.0-0.05 | docs-architect, docs-recovery, publishers |
| Analytical | 0.1-0.15 | planners, researchers, calibrators |
| Balanced | 0.2-0.3 | editors, meeting prep, orchestrators |
| Creative | 0.4-0.5 | drafters, communications |

---

## Cost Optimization

### High-Volume Operations

Use Gemini Flash 3 for:
- Fact extraction
- Validation passes
- Technical transforms

**Rationale**: These tasks are deterministic and high-frequency. Cost savings compound.

### Strategic Operations

Use GPT-5.2 for:
- Planning only (not execution)
- Once per pipeline

**Rationale**: Planning quality drives downstream quality. Worth the investment.

### Quality-Critical Operations

Use Claude Opus 4.5 for:
- Primary orchestration
- Final editing passes
- Voice-critical work

**Rationale**: These are the human-visible outputs. Quality matters most here.

---

## Enforcement

Model assignments are enforced via `opencode.json`, not Task prompts:

```json
{
  "agents": {
    "blog_planner": {
      "model": "openai/gpt-5.2",
      "temperature": 0.10
    },
    "blog_drafter": {
      "model": "anthropic/claude-sonnet-4-5",
      "temperature": 0.28
    },
    "blog_publisher": {
      "model": "google/gemini-flash-3",
      "temperature": 0.05
    }
  }
}
```

**Never override model in Task calls.** Use configuration.

---

## Anti-Patterns

### Wrong Model for Task

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Opus for fact extraction | Expensive, no benefit | Use Gemini Flash |
| Flash for creative writing | Prose quality suffers | Use Sonnet |
| Sonnet for planning | Structure may drift | Use GPT-5.2 |

### Temperature Mismatch

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| High temp for publishers | Inconsistent output | Use 0.0-0.05 |
| Low temp for drafters | Stilted prose | Use 0.25-0.5 |
| Variable temp per run | Unpredictable | Fix in config |

---

## Related

- **[Model Assignments](../reference/model-assignments.md)** - Full agent-model mapping
- **[Orchestration](./orchestration.md)** - Phase-based patterns
- **[Agent Index](../reference/agent-index.md)** - All agents A-Z
