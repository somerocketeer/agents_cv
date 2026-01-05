# Evidence Discipline Pattern

Systematic approach to fact tracking, citation, and evidence-backed content.

---

## Core Principle

Every claim needs a source. Every source needs synthesis. No dropped citations.

---

## Fact Bank Structure

### Fact Format

```markdown
### fact_001: Title
**Source:** Where this came from
**Personas:** Who this is relevant to
**Tier:** Confidence level
Content of the fact with preserved literals.
```

### Example

```markdown
### fact_001: P99 Latency
**Source:** scratchpad section 2.3, performance metrics
**Personas:** Architect Alice, Technical Teddy
**Tier:** T1 (measured)
API maintains 45ms p99 latency under 10,000 requests per second.
```

---

## Confidence Tiers

| Tier | Description | Evidence Type | Use |
|------|-------------|---------------|-----|
| **T1** | Measured/verified | Production metrics, benchmarks, code | Lead with these |
| **T2** | Customer-reported | Quotes, testimonials, case studies | Support with context |
| **T3** | Industry benchmark | Third-party reports, analyst data | Comparison only |
| **T4** | Estimate/projection | Calculations, forecasts | Hedge clearly |

### Usage Rules

- **T1 facts**: Can state directly without qualification
- **T2 facts**: Attribute to source ("Customer X reported...")
- **T3 facts**: Provide source ("According to [report]...")
- **T4 facts**: Qualify explicitly ("We estimate...", "Projected to...")

---

## Persona Mapping

### Why Personas Matter

Facts hit differently for different audiences. Map each fact to relevant personas:

```markdown
### fact_002: Cost Reduction
**Personas:** Budget-Conscious Bob, Executive Emma
40% infrastructure cost reduction.

### fact_003: Architecture Detail
**Personas:** Technical Teddy, Architect Alice
DAG-based dependency tracking with event-driven invalidation.
```

### Standard Personas

| Persona | Role | Cares About |
|---------|------|-------------|
| **Architect Alice** | Technical decision maker | Scalability, reliability, integration |
| **Technical Teddy** | Hands-on implementer | APIs, debugging, operational burden |
| **Budget-Conscious Bob** | Cost-focused buyer | TCO, ROI, hidden costs |
| **Executive Emma** | Strategic decision maker | Competitive advantage, risk, timeline |

---

## Citation Rules

### In Draft

Reference facts by ID:

```markdown
Our API maintains 45ms p99 latency under 10k RPS [fact_001].
```

### Synthesis Required

Never drop a citation. Every reference must include synthesis:

**Bad**:
> Performance is excellent [fact_001].

**Good**:
> Our API maintains 45ms p99 latency under 10k RPS [fact_001], which is 4.4x faster than the industry average.

### No Persona Leakage

Internal persona names never appear in final copy:

**Bad**:
> For Architect Alice, this means reduced operational burden.

**Good**:
> For technical decision makers, this means reduced operational burden.

---

## Publishing Transform

### Draft → Final

| Draft | Final |
|-------|-------|
| `[fact_001]` | `[1]` |
| `[fact_002]` | `[2]` |
| `## Editorial Notes` | (removed) |
| Persona names | Generic role names |

### References Section

```markdown
## References

1. Production metrics, Q4 2024
2. Acme Corp case study, December 2024
3. API Performance Benchmark Report, 2024
```

---

## Validation Checks

### Linter Checks

| Check | Rule | Severity |
|-------|------|----------|
| Citation exists | Every `[fact_XXX]` has matching fact | Blocking |
| Citation synthesized | No bare citations | Warning |
| No orphaned facts | Every fact used at least once | Warning |
| No persona names | No "Alice", "Teddy", etc. in final | Blocking |

### Example Output

```json
{
  "citations": {
    "status": "FAIL",
    "errors": [
      { "type": "missing_fact", "ref": "fact_099" },
      { "type": "bare_citation", "line": 45, "ref": "fact_003" }
    ],
    "warnings": [
      { "type": "orphaned_fact", "id": "fact_007" }
    ]
  }
}
```

---

## Workflow Integration

### Phase: Evidence Foundation

```javascript
// 1. Extract facts
Task({
  subagent_type: "fact_bank_builder",
  prompt: "Extract facts from scratchpad.md → fact_bank.md"
})

// 2. Map to personas
Task({
  subagent_type: "persona_calibrator",
  prompt: "Map fact_bank.md to personas → persona_brief.md"
})
```

### Phase: Drafting

```javascript
// Drafter receives:
// - fact_bank.md (the facts)
// - persona_brief.md (the mapping)
// - plan.md (which facts to use where)

Task({
  subagent_type: "blog_drafter",
  prompt: `
    Write using facts from fact_bank.md.
    Reference as [fact_XXX].
    Synthesize every citation.
  `
})
```

### Phase: Publishing

```javascript
// Publisher transforms:
// - [fact_XXX] → [N]
// - Builds References section
// - Removes Editorial Notes

Task({
  subagent_type: "blog_publisher",
  prompt: "Transform to publication format → final-blog.md"
})
```

---

## Anti-Patterns

### Dropped Citations

**Problem**: Bare fact reference without context

```markdown
This is fast [fact_001].
```

**Fix**: Always synthesize

```markdown
This maintains 45ms p99 latency [fact_001], 4.4x faster than industry average.
```

### Invented Facts

**Problem**: Claims not in fact bank

**Fix**: All claims must trace to facts. If it's not in the fact bank, either:
1. Add it to the fact bank with proper sourcing
2. Remove the claim

### Tier Mismatch

**Problem**: Stating T4 (estimate) as T1 (measured)

**Fix**: Match language to tier

- T1: "API maintains 45ms latency"
- T4: "We estimate 40% cost savings"

---

## Related

- **[Orchestration](./orchestration.md)** - Phase integration
- **[Model Routing](./model-routing.md)** - Agent selection
- **[Examples](../examples/marketing/)** - Real fact banks
