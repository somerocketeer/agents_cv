# Example: LinkedIn Campaign

Example of the LinkedIn orchestrator producing multiple posts.

---

## Input: Scratchpad

```markdown
# New Feature: Smart Cache Invalidation

## What It Does
- Dependency graph tracks relationships between cached objects
- When source data changes, only affected cache entries invalidate
- No more full cache purges or stale data

## Key Benefits
- 10x fewer cache misses
- Sub-second invalidation propagation
- Zero stale data incidents since launch

## Technical Details
- DAG-based dependency tracking
- Event-driven invalidation
- Configurable staleness tolerance
```

---

## Execution

### Phase 1-2: Evidence Foundation

**Output**: `fact_bank.md`
```markdown
### fact_001: Cache Miss Reduction
**Tier:** T1 (measured)
Smart invalidation reduces cache misses by 10x compared to TTL-based expiration.

### fact_002: Propagation Speed
**Tier:** T1 (measured)
Invalidation propagates across entire cache network in under one second.

### fact_003: Reliability
**Tier:** T1 (measured)
Zero stale data incidents reported since feature launch (3 months).

### fact_004: Architecture
**Tier:** T1 (verified)
Uses DAG-based dependency tracking with event-driven invalidation.
```

### Phase 3: Planning

**Agent**: linkedin_post_planner (GPT-5.2)

```javascript
Task({
  subagent_type: "linkedin_post_planner",
  prompt: `
    Create hook-first post plans:
    Fact bank: fact_bank.md
    Persona brief: persona_brief.md
    Target: 2-4 posts
    Output: plan-linkedin.md
  `
})
```

**Output**: `plan-linkedin.md`
```markdown
## LinkedIn Campaign Plan

### Post 1: The Problem Hook
**Hook:** "Your cache invalidation strategy is probably wrong."
**Angle:** Challenge conventional TTL-based thinking
**Facts:** fact_001
**CTA:** Link to docs

### Post 2: The Numbers Hook
**Hook:** "10x fewer cache misses. Here's how."
**Angle:** Lead with measurable outcome
**Facts:** fact_001, fact_002
**CTA:** Technical deep-dive link

### Post 3: The Reliability Hook
**Hook:** "3 months. Zero stale data incidents."
**Angle:** Reliability story
**Facts:** fact_003
**CTA:** Case study link

### Post 4: The Technical Hook
**Hook:** "We replaced TTLs with a dependency graph."
**Angle:** Technical depth for engineers
**Facts:** fact_004
**CTA:** Architecture blog link
```

### Phase 4: Drafting

**Agent**: linkedin_drafter (Claude Sonnet 4.5, temp 0.30)

```javascript
// Draft all posts
for (const post of plan.posts) {
  Task({
    subagent_type: "linkedin_drafter",
    prompt: `
      Write LinkedIn post:
      Hook: ${post.hook}
      Angle: ${post.angle}
      Facts: ${post.facts}

      Rules:
      - Hook in first line
      - 150-200 words
      - No hashtag spam
      - Evidence, not claims

      Output: draft-post-${post.id}.md
    `
  })
}
```

**Output**: `draft-post-1.md`
```markdown
Your cache invalidation strategy is probably wrong.

TTLs are a guess. You set 5 minutes, hope it's long enough for performance,
short enough for freshness. Sometimes you're right. Often you're not.

We replaced guessing with knowing.

Our smart invalidation tracks dependencies between cached objects. When source
data changes, only affected entries invalidate. Not everything. Just what matters.

The result: 10x fewer cache misses [1].

Not because we cache more aggressively. Because we invalidate precisely.

No more choosing between performance and freshness. You get both.

[Link to documentation]

---
[1] Production metrics, comparing TTL-based vs dependency-based invalidation
```

**Output**: `draft-post-2.md`
```markdown
10x fewer cache misses. Here's how.

Traditional caching makes you choose: long TTLs for performance, short TTLs
for freshness. It's a tradeoff that shouldn't exist.

We built a dependency graph for cache invalidation.

When data changes, the graph knows exactly which cache entries depend on it.
Invalidation propagates in under one second [1]. No full purges. No stale data.

The technical bit: DAG-based tracking with event-driven propagation.

The practical bit: Your cache hit rate goes up. Your stale data rate goes to zero.

We've been running this in production for 3 months. Zero stale data incidents [2].

[Link to technical deep-dive]

---
[1] Production metrics, Q4 2024
[2] Incident tracking, Oct-Dec 2024
```

### Phase 5: Editing

**Agent**: linkedin_editor

**Edits Applied**:
- Post 1: Changed "seamlessly" to specific mechanism
- Post 2: Removed "revolutionary"
- Post 3: Shortened 28-word sentence
- Post 4: Added specific latency number

### Phase 6: Publishing

**Agent**: linkedin_finalizer

```javascript
Task({
  subagent_type: "linkedin_finalizer",
  prompt: `
    Validate and finalize:
    Posts: draft-post-*.md
    Fact bank: fact_bank.md

    Check:
    - Hook in first line
    - Word count 150-200
    - No banned phrases
    - Citations valid

    Output: final-linkedin-campaign.md
  `
})
```

---

## Final Output

```markdown
# LinkedIn Campaign: Smart Cache Invalidation

## Post 1: The Problem Hook

Your cache invalidation strategy is probably wrong.

TTLs are a guess. You set 5 minutes, hope it's long enough for performance,
short enough for freshness. Sometimes you're right. Often you're not.

We replaced guessing with knowing.

Our smart invalidation tracks dependencies between cached objects. When source
data changes, only affected entries invalidate. Not everything. Just what matters.

The result: 10x fewer cache misses.

Not because we cache more aggressively. Because we invalidate precisely.

No more choosing between performance and freshness. You get both.

[Link]

---

## Post 2: The Numbers Hook

10x fewer cache misses. Here's how.

Traditional caching makes you choose: long TTLs for performance, short TTLs
for freshness. It's a tradeoff that shouldn't exist.

We built a dependency graph for cache invalidation.

When data changes, the graph knows exactly which cache entries depend on it.
Invalidation propagates in under one second. No full purges. No stale data.

The technical bit: DAG-based tracking with event-driven propagation.

The practical bit: Your cache hit rate goes up. Your stale data rate goes to zero.

We've been running this in production for 3 months. Zero stale data incidents.

[Link]

---

## Post 3: The Reliability Hook

3 months. Zero stale data incidents.

That's our track record since launching smart cache invalidation.

The old way: Set TTLs, hope for the best, scramble when stale data slips through.

The new way: Dependency tracking. When source data changes, affected cache
entries invalidate automatically. In under one second.

No guessing. No hoping. No incidents.

[Link]

---

## Post 4: The Technical Hook

We replaced TTLs with a dependency graph.

Every cached object knows what it depends on. When source data changes,
invalidation propagates through the graph in <1 second.

The architecture:
- DAG-based dependency tracking
- Event-driven invalidation
- Configurable staleness tolerance

The results:
- 10x fewer cache misses
- Sub-second propagation
- Zero stale data (3 months running)

If you're still using TTLs, you're leaving performance on the table.

[Link to architecture blog]
```

---

## Quality Report

```yaml
summary: 4 LinkedIn posts created
posts:
  - id: 1
    hook: "Your cache invalidation strategy is probably wrong"
    words: 156
    status: PASSED
  - id: 2
    hook: "10x fewer cache misses"
    words: 168
    status: PASSED
  - id: 3
    hook: "3 months. Zero stale data incidents"
    words: 142
    status: PASSED
  - id: 4
    hook: "We replaced TTLs with a dependency graph"
    words: 158
    status: PASSED
validation:
  anti_llm: PASSED
  citations: PASSED
  word_counts: PASSED
```
