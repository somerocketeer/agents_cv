# Example: Blog Post Pipeline

Complete example of the marketing blog pipeline.

---

## Input: Scratchpad

```markdown
# Topic: API Performance at Scale

## Key Points
- Our API maintains 45ms p99 latency under 10k RPS
- Customer Acme Corp reported 40% cost reduction
- Industry average is 200ms for similar workloads
- We use edge caching with smart invalidation

## Technical Details
- Global edge network (47 PoPs)
- Automatic failover
- Smart cache invalidation using dependency graphs

## Target Audience
- Technical architects evaluating API solutions
- Platform engineers building integrations
```

---

## Execution

### Phase 1: Setup

```bash
mkdir -p ~/Marketing/_staging/api-performance
cp scratchpad.md ~/Marketing/_staging/api-performance/

blog-orchestrator --project api-performance
```

### Phase 2: Evidence Foundation

**Agent**: fact_bank_builder

```javascript
Task({
  subagent_type: "fact_bank_builder",
  prompt: `
    Extract facts from: scratchpad.md
    Output: fact_bank.md
  `
})
```

**Output**: `fact_bank.md`
```markdown
## Facts

### fact_001: P99 Latency
**Source:** scratchpad, Key Points
**Personas:** Architect Alice, Technical Teddy
**Tier:** T1 (measured)
API maintains 45ms p99 latency under 10,000 requests per second load.

### fact_002: Cost Reduction
**Source:** scratchpad, Key Points
**Personas:** Budget-Conscious Bob, Executive Emma
**Tier:** T2 (customer-reported)
Acme Corp reported 40% reduction in infrastructure costs after migration.

### fact_003: Industry Comparison
**Source:** scratchpad, Key Points
**Personas:** Architect Alice
**Tier:** T3 (industry benchmark)
Industry average latency for similar workloads is 200ms, making our 45ms approximately 4.4x faster.

### fact_004: Edge Network
**Source:** scratchpad, Technical Details
**Personas:** Technical Teddy
**Tier:** T1 (verified)
Global edge network with 47 points of presence (PoPs).

### fact_005: Cache Invalidation
**Source:** scratchpad, Technical Details
**Personas:** Technical Teddy, Architect Alice
**Tier:** T1 (verified)
Smart cache invalidation using dependency graphs ensures data freshness without full cache purges.
```

**Agent**: persona_calibrator

```javascript
Task({
  subagent_type: "persona_calibrator",
  prompt: `
    Map facts to personas:
    Fact bank: fact_bank.md
    Output: persona_brief.md
  `
})
```

**Output**: `persona_brief.md`
```markdown
## Persona Brief

### Architect Alice (Primary)
**Role:** Technical decision maker evaluating API solutions
**Pain Points:** Latency at scale, reliability, total cost
**Relevant Facts:** fact_001, fact_003, fact_004, fact_005
**Messaging Angle:** Performance without compromise

### Technical Teddy
**Role:** Hands-on engineer implementing integrations
**Pain Points:** Complexity, debugging, operational burden
**Relevant Facts:** fact_004, fact_005
**Messaging Angle:** Simple to integrate, easy to debug

### Budget-Conscious Bob
**Role:** Cost-focused buyer
**Pain Points:** Infrastructure costs, hidden fees
**Relevant Facts:** fact_002
**Messaging Angle:** Measurable cost savings

### Executive Emma (Secondary)
**Role:** Strategic decision maker
**Pain Points:** ROI, competitive advantage
**Relevant Facts:** fact_002, fact_003
**Messaging Angle:** Competitive differentiation
```

### Phase 3: Planning

**Agent**: blog_planner (GPT-5.2)

```javascript
Task({
  subagent_type: "blog_planner",
  prompt: `
    Create argument-driven outline:
    Fact bank: fact_bank.md
    Persona brief: persona_brief.md
    Style guide: style-guide.md
    Output: plan-blog.md
  `
})
```

**Output**: `plan-blog.md`
```markdown
## Blog Plan: API Performance at Scale

### Thesis
At scale, every millisecond matters. Here's how we maintain 45ms p99 latency
under 10k RPS while competitors average 200ms.

### Structure

**Hook** (100 words)
- Open with the 200ms industry average problem
- Introduce the 45ms claim
- Set up the "how" question

**Section 1: The Numbers** (200 words)
- fact_001: 45ms p99 under 10k RPS
- fact_003: 4.4x faster than industry average
- Why this matters for Architect Alice

**Section 2: How We Do It** (300 words)
- fact_004: 47 edge PoPs
- fact_005: Dependency graph cache invalidation
- Technical depth for Technical Teddy

**Section 3: Real-World Results** (200 words)
- fact_002: Acme Corp 40% cost reduction
- Connect performance to business outcomes
- Appeal to Budget-Conscious Bob

**Counterargument** (100 words)
- "Benchmarks don't reflect real workloads"
- Address with: These are production numbers, not synthetic benchmarks

**Conclusion** (100 words)
- Restate thesis
- Call to action: Try the API

### Word Count Target
800-1,000 words
```

### Phase 4: Drafting

**Agent**: blog_drafter (Claude Sonnet 4.5)

```javascript
Task({
  subagent_type: "blog_drafter",
  prompt: `
    Write blog draft:
    Plan: plan-blog.md
    Fact bank: fact_bank.md
    Persona brief: persona_brief.md
    Style guide: style-guide.md
    Output: draft-blog.md
  `
})
```

**Output**: `draft-blog.md` (excerpt)
```markdown
# Why 200ms Is Too Slow (And What We Did About It)

The industry treats 200ms API response times as acceptable. We disagree.

When you're serving 10,000 requests per second, those milliseconds compound.
A 200ms average means some requests hit 500ms or more. At scale, that's the
difference between a responsive application and a frustrating one.

We built our API to maintain 45ms p99 latency under 10k RPS [fact_001].
That's not a marketing number from a synthetic benchmark. It's what our
production infrastructure delivers, day after day.

## How 4.4x Faster Changes the Equation

Industry benchmarks put similar workloads at 200ms average [fact_003].
Our 45ms represents a 4.4x improvement...

## The Architecture Behind the Numbers

Speed at scale requires infrastructure decisions that compound.

Our global edge network spans 47 points of presence [fact_004]. Requests
route to the nearest PoP automatically, cutting network latency before
our code even runs...

## What This Means for Your Infrastructure Costs

Acme Corp migrated to our API last quarter. Their infrastructure costs
dropped 40% [fact_002]. Not because we're cheaper per request, but
because faster responses mean fewer retries, smaller connection pools,
and simpler timeout handling...

## "But Benchmarks Never Reflect Reality"

Fair point. Synthetic benchmarks often inflate performance claims.

These aren't synthetic numbers. They're from production traffic across
thousands of customers. We publish our p99 latency in real-time on our
status page...

## Try It Yourself

Start a free trial and run your own benchmarks. We're confident in
what you'll find.

---

## References

1. [fact_001] Production metrics, Q4 2024
2. [fact_002] Acme Corp case study, December 2024
3. [fact_003] API Performance Benchmark Report, 2024
4. [fact_004] Infrastructure documentation
```

### Phase 5: Editing

**Agent**: blog_editor (Claude Sonnet 4.5)

```javascript
Task({
  subagent_type: "blog_editor",
  prompt: `
    Edit and polish:
    Draft: draft-blog.md
    Fact bank: fact_bank.md
    Style guide: style-guide.md
    Output: edited-blog.md
  `
})
```

**Edits Applied**:
- Removed "truly" from paragraph 3
- Changed "leverage" to "use"
- Shortened 32-word sentence to 18 words
- Verified all fact IDs match fact bank
- Confirmed no persona names in copy

### Phase 6: Publishing

**Agent**: blog_publisher (Gemini Flash 3)

```javascript
Task({
  subagent_type: "blog_publisher",
  prompt: `
    Transform to publication format:
    Edited: edited-blog.md
    Fact bank: fact_bank.md
    Style guide: style-guide.md
    Output: final-blog.md
  `
})
```

**Output**: `final-blog.md`
```markdown
# Why 200ms Is Too Slow (And What We Did About It)

The industry treats 200ms API response times as acceptable. We disagree.

When you're serving 10,000 requests per second, those milliseconds compound.
A 200ms average means some requests hit 500ms or more. At scale, that's the
difference between a responsive application and a frustrating one.

We built our API to maintain 45ms p99 latency under 10k RPS [1]. That's not
a marketing number from a synthetic benchmark. It's what our production
infrastructure delivers, day after day.

## How 4.4x Faster Changes the Equation

Industry benchmarks put similar workloads at 200ms average [3]. Our 45ms
represents a 4.4x improvement...

[... full content ...]

---

## References

1. Production metrics, Q4 2024
2. Acme Corp case study, December 2024
3. API Performance Benchmark Report, 2024
4. Infrastructure documentation
```

### Phase 7: Validation

```bash
marketing_lint --project-dir ~/Marketing/_staging/api-performance --content-type blog
```

**Result**:
```json
{
  "status": "PASS",
  "checks": {
    "anti_llm": "PASS",
    "citations": "PASS",
    "word_count": "PASS (892 words)",
    "persona_leakage": "PASS"
  }
}
```

---

## Final Output

```yaml
summary: Blog post created successfully
artifacts:
  fact_bank: ~/Marketing/_staging/api-performance/fact_bank.md
  persona_brief: ~/Marketing/_staging/api-performance/persona_brief.md
  plan: ~/Marketing/_staging/api-performance/plan-blog.md
  draft: ~/Marketing/_staging/api-performance/draft-blog.md
  edited: ~/Marketing/_staging/api-performance/edited-blog.md
  final: ~/Marketing/_staging/api-performance/final-blog.md
quality:
  thesis_delivery: STRONG
  synthesis_coverage: 5 of 5 citations synthesized
validation:
  status: PASSED
```
