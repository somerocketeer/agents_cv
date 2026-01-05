# Example: Whitepaper Extended Mode

Example of the extended whitepaper pipeline (5,000-7,500 words).

---

## Input

```bash
whitepaper-orchestrator --project api-architecture --mode extended
```

---

## Key Differences from Standard Mode

| Aspect | Standard | Extended |
|--------|----------|----------|
| Word count | 2,000-2,600 | 5,000-7,500 |
| Sections | 4-6 | 8-12 |
| Drafting | Single pass | Section-by-section |
| Assembly | None | whitepaper_assembler stitches |
| Model for assembly | N/A | Gemini 3 Pro |

---

## Execution

### Phase 1-3: Same as Standard

Evidence gathering, persona mapping, and planning proceed identically.

**Plan Output** (extended):
```markdown
## Extended Whitepaper Plan

### Thesis
Modern API architecture requires rethinking traditional patterns.
This paper presents a comprehensive approach to building APIs
that scale from startup to enterprise.

### Sections (8)

1. **Executive Summary** (500 words)
2. **The Problem with Traditional APIs** (800 words)
3. **Architectural Principles** (1,000 words)
4. **Edge-First Design** (800 words)
5. **Cache Invalidation Strategies** (700 words)
6. **Observability at Scale** (600 words)
7. **Case Study: Acme Corp** (800 words)
8. **Implementation Roadmap** (500 words)
9. **Conclusion** (300 words)

### Total Target: 6,000 words
```

### Phase 4: Section-by-Section Drafting

**Agent**: whitepaper_drafter (per section)

```javascript
// Sections drafted individually
for (const section of plan.sections) {
  Task({
    subagent_type: "whitepaper_drafter",
    prompt: `
      Write section: ${section.title}
      Target: ${section.wordCount} words
      Previous sections: ${previousContent}
      Fact bank: fact_bank.md
      Output: section-${section.id}.md
    `
  })
}
```

**Section Outputs**:
- `section-01-executive-summary.md` (512 words)
- `section-02-problem.md` (823 words)
- `section-03-principles.md` (1,047 words)
- `section-04-edge.md` (791 words)
- `section-05-cache.md` (688 words)
- `section-06-observability.md` (612 words)
- `section-07-case-study.md` (834 words)
- `section-08-roadmap.md` (498 words)
- `section-09-conclusion.md` (287 words)

### Phase 5: Assembly

**Agent**: whitepaper_assembler (Gemini 3 Pro)

```javascript
Task({
  subagent_type: "whitepaper_assembler",
  prompt: `
    Stitch sections into cohesive whitepaper:
    Sections: section-*.md
    Plan: plan-whitepaper.md

    Ensure:
    - Smooth transitions between sections
    - Consistent terminology
    - No redundancy
    - Unified narrative arc

    Output: draft-whitepaper.md
  `
})
```

**Assembly Actions**:
1. Merge all sections
2. Add transition paragraphs
3. Unify terminology (e.g., "PoP" vs "point of presence")
4. Remove redundant explanations
5. Ensure thesis threads through all sections

**Output**: `draft-whitepaper.md` (6,092 words)

### Phase 6: Editing

**Agent**: whitepaper_editor

```javascript
Task({
  subagent_type: "whitepaper_editor",
  prompt: `
    Audit and edit:
    Draft: draft-whitepaper.md
    Fact bank: fact_bank.md
    Style guide: style-guide.md
    Output: edited-whitepaper.md
  `
})
```

**Edit Passes**:
1. **Structural**: Verify thesis delivery per section
2. **Evidence**: Check all citations synthesized
3. **Voice**: Remove LLM tells
4. **Flow**: Smooth transitions

### Phase 7: Publishing

**Agent**: whitepaper_publisher

```javascript
Task({
  subagent_type: "whitepaper_publisher",
  prompt: `
    Transform to publication format:
    Edited: edited-whitepaper.md
    Fact bank: fact_bank.md
    Output: final-whitepaper.md
  `
})
```

**Transformations**:
- `[fact_001]` → `[1]`
- Remove `## Editorial Notes`
- Add title page metadata
- Format references section

---

## Output Structure

```markdown
# Modern API Architecture: A Comprehensive Guide

**Abstract**
[Executive summary content]

---

## Table of Contents

1. Executive Summary
2. The Problem with Traditional APIs
3. Architectural Principles
4. Edge-First Design
5. Cache Invalidation Strategies
6. Observability at Scale
7. Case Study: Acme Corp
8. Implementation Roadmap
9. Conclusion

---

## 1. Executive Summary

[512 words]

---

## 2. The Problem with Traditional APIs

[823 words with transitions]

---

[... remaining sections ...]

---

## References

1. Production metrics, Q4 2024
2. Acme Corp case study, December 2024
3. API Performance Benchmark Report, 2024
[... additional references ...]
```

---

## Quality Report

```yaml
summary: Extended whitepaper completed (6,092 words)
artifacts:
  sections:
    - section-01-executive-summary.md
    - section-02-problem.md
    - section-03-principles.md
    - section-04-edge.md
    - section-05-cache.md
    - section-06-observability.md
    - section-07-case-study.md
    - section-08-roadmap.md
    - section-09-conclusion.md
  draft: draft-whitepaper.md
  edited: edited-whitepaper.md
  final: final-whitepaper.md
quality:
  thesis_delivery: STRONG
  section_coherence: STRONG
  synthesis_coverage: 18 of 18 citations synthesized
  word_count: 6,092 (target: 5,000-7,500)
validation:
  status: PASSED
```

---

## Notes

- Extended mode uses Gemini 3 Pro for assembly due to long context handling
- Sections are drafted individually to maintain quality
- Assembly pass ensures coherent narrative across sections
- Same evidence discipline as standard mode
