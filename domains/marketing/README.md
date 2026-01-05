# Marketing Domain

B2B content production at scale with evidence-backed thought leadership across 8 content types.

---

## Overview

**Orchestrators**: 8
- `blog-orchestrator` - Blog posts (800-1,400 words)
- `whitepaper-orchestrator` - Whitepapers (2,000-7,500 words)
- `datasheet-orchestrator` - Procurement-ready datasheets
- `solution-brief-orchestrator` - Executive-ready briefs
- `newsletter-orchestrator` - Insight-led newsletters
- `linkedin-orchestrator` - Hook-first LinkedIn posts
- `customer-success-orchestrator` - Rollout playbooks
- `video-integration-orchestrator` - Walkthrough videos

**Shared Subagents**: 3
- `fact_bank_builder` - Extract structured facts
- `persona_calibrator` - Map facts to personas
- `marketing_research_scout` - Fetch market intelligence

**Pipeline Subagents**: 40+ (planner, drafter, editor, publisher/finalizer per content type)

---

## Architecture

### Marketing Pipeline

```
Scratchpad → Fact Bank → Persona Brief → Market Intel → Plan → Draft → Edit → Publish
                ↓             ↓              ↓
           fact_bank_    persona_      marketing_
           builder      calibrator  research_scout
```

### Phase Breakdown

| Phase | Agent(s) | Purpose |
|-------|----------|---------|
| **Setup** | Orchestrator | Verify workspace, resolve paths |
| **Evidence** | fact_bank_builder, persona_calibrator | Extract and map facts |
| **Research** | marketing_research_scout | Gather market intelligence |
| **Plan** | *_planner | Create argument-driven outline |
| **Draft** | *_drafter | Write synthesis-first prose |
| **Edit** | *_editor | Polish + validate |
| **Publish** | *_publisher / *_finalizer | Transform to publication format |
| **Validate** | Orchestrator | Run linters, check citations |

---

## Evidence System

### Fact Bank

Facts are extracted from scratchpads with:
- Unique fact IDs (`fact_001`, `fact_002`, etc.)
- Source attribution
- Persona mappings
- Confidence tiers (T1-T4)

```markdown
## Facts

### fact_001: API Response Time
**Source:** scratchpad section 2.3
**Personas:** Architect Alice, Technical Teddy
**Tier:** T1 (measured)
Median response time of 45ms at p99 under 10k RPS load.

### fact_002: Cost Reduction
**Source:** customer interview
**Personas:** Budget-Conscious Bob
**Tier:** T2 (reported)
Customer reported 40% reduction in infrastructure costs.
```

### Confidence Tiers

| Tier | Description | Use |
|------|-------------|-----|
| **T1** | Measured/verified data | Lead with these |
| **T2** | Customer-reported | Support with context |
| **T3** | Industry benchmarks | Comparison only |
| **T4** | Estimates/projections | Hedge clearly |

### Persona Mapping

Facts are mapped to target personas:
- **Architect Alice** - Technical decision maker
- **Technical Teddy** - Hands-on implementer
- **Budget-Conscious Bob** - Cost-focused buyer
- **Executive Emma** - Strategic decision maker

---

## Shared Subagents

### fact_bank_builder

**Purpose**: Extract structured facts from scratchpads

**Model**: Gemini Flash 3 (temp 0.05)

**Input**: Raw scratchpad content

**Output**: Structured fact bank with:
- Fact ID, title, content
- Source attribution
- Persona mappings
- Confidence tier

**Example**:
```javascript
Task({
  subagent_type: "fact_bank_builder",
  prompt: `
    Extract facts from: ${scratchpadPath}

    Output: ${factBankPath}

    Rules:
    - Assign unique fact IDs (fact_001, fact_002...)
    - Preserve literal quotes
    - Tag with personas
    - Assign confidence tier
  `
})
```

---

### persona_calibrator

**Purpose**: Map facts to target personas

**Model**: Claude Sonnet 4.5 (temp 0.10)

**Input**: Fact bank

**Output**: Persona brief with:
- Per-persona fact relevance
- Messaging angles
- Pain point mappings

---

### marketing_research_scout

**Purpose**: Fetch market intelligence via web search

**Model**: Claude Sonnet 4.5 (temp 0.15)

**Input**: Topic, competitors

**Output**: Market research with:
- Competitor positioning
- Industry trends
- Supporting data points

---

## Content Type Pipelines

### Blog (800-1,400 words)

**Orchestrator**: `blog-orchestrator`

**Pipeline**:
```bash
blog-orchestrator --project my-blog-post
```

**Subagents**:
| Agent | Model | Temp | Purpose |
|-------|-------|------|---------|
| `blog_planner` | GPT-5.2 | 0.10 | Argument-driven outline |
| `blog_drafter` | Claude Sonnet 4.5 | 0.28 | Thought-leadership prose |
| `blog_editor` | Claude Sonnet 4.5 | 0.15 | Polish + de-LLM audit |
| `blog_publisher` | Gemini Flash 3 | 0.05 | Transform to publication |

**Output Structure**:
- Hook (problem statement)
- Thesis (clear position)
- 3-5 evidence-backed sections
- Counterarguments addressed
- Call to action
- Numbered references

---

### Whitepaper (2,000-7,500 words)

**Orchestrator**: `whitepaper-orchestrator`

**Pipeline**:
```bash
# Standard mode (2,000-2,600 words)
whitepaper-orchestrator --project my-whitepaper

# Extended mode (5,000-7,500 words)
whitepaper-orchestrator --project my-whitepaper --mode extended
```

**Subagents**:
| Agent | Model | Temp | Purpose |
|-------|-------|------|---------|
| `whitepaper_planner` | GPT-5.2 | 0.10 | Thesis-driven structure |
| `whitepaper_drafter` | Claude Sonnet 4.5 | 0.28 | Synthesis-first prose |
| `whitepaper_assembler` | Gemini 3 Pro | 0.20 | Stitch extended sections |
| `whitepaper_editor` | Claude Sonnet 4.5 | 0.15 | Audit and polish |
| `whitepaper_publisher` | Gemini Flash 3 | 0.05 | Final transform |

**Extended Mode**:
- Runs in section passes
- Uses `whitepaper_assembler` to stitch
- Handles 5,000-7,500 word output

---

### Datasheet

**Orchestrator**: `datasheet-orchestrator`

**Pipeline**:
```bash
datasheet-orchestrator --project my-datasheet
```

**Subagents**:
| Agent | Model | Temp | Purpose |
|-------|-------|------|---------|
| `datasheet_planner` | GPT-5.2 | 0.10 | Spec organization |
| `datasheet_drafter` | Claude Sonnet 4.5 | 0.20 | Technical specifications |
| `datasheet_editor` | Claude Sonnet 4.5 | 0.10 | Accuracy review |
| `datasheet_finalizer` | Gemini Flash 3 | 0.05 | Final spec verification |

**Output Structure**:
- Product overview
- Key specifications table
- Technical requirements
- Integration details
- Support information

---

### Solution Brief

**Orchestrator**: `solution-brief-orchestrator`

**Pipeline**:
```bash
solution-brief-orchestrator --project my-brief
```

**Subagents**:
| Agent | Model | Temp | Purpose |
|-------|-------|------|---------|
| `scratchpad_normalizer` | Gemini Flash 3 | 0.05 | Content normalization |
| `solution_brief_planner` | GPT-5.2 | 0.10 | Executive narrative |
| `solution_brief_drafter` | Claude Sonnet 4.5 | 0.28 | Executive prose |
| `solution_brief_editor` | Claude Sonnet 4.5 | 0.15 | Tone review |
| `solution_brief_finalizer` | Gemini Flash 3 | 0.05 | Release audit |

**Output Structure**:
- Executive summary
- Challenge/opportunity
- Solution overview
- Key benefits (quantified)
- Next steps

---

### Newsletter

**Orchestrator**: `newsletter-orchestrator`

**Pipeline**:
```bash
newsletter-orchestrator --project q1-update
```

**Subagents**:
| Agent | Model | Temp | Purpose |
|-------|-------|------|---------|
| `newsletter_planner` | GPT-5.2 | 0.10 | Thesis-driven structure |
| `newsletter_drafter` | Claude Sonnet 4.5 | 0.28 | Insight-led prose |
| `content_editor` | Claude Sonnet 4.5 | 0.15 | Polish + de-LLM |
| `newsletter_finalizer` | Gemini Flash 3 | 0.05 | Final QA |

**Output Structure**:
- Opening insight (not feature list)
- Architectural framing
- Feature highlights with context
- Community/ecosystem updates
- Call to action

---

### LinkedIn

**Orchestrator**: `linkedin-orchestrator`

**Pipeline**:
```bash
linkedin-orchestrator --project my-campaign
```

**Subagents**:
| Agent | Model | Temp | Purpose |
|-------|-------|------|---------|
| `linkedin_post_planner` | GPT-5.2 | 0.10 | Hook-first structure |
| `linkedin_drafter` | Claude Sonnet 4.5 | 0.30 | Insight-driven posts |
| `linkedin_editor` | Claude Sonnet 4.5 | 0.15 | Polish + de-LLM |
| `linkedin_finalizer` | Gemini Flash 3 | 0.05 | Validation |

**Output**: 1-4 posts with:
- Hook (first line grabs attention)
- Insight (not feature dump)
- Evidence (specific, quantified)
- Call to action

---

### Customer Success

**Orchestrator**: `customer-success-orchestrator`

**Pipeline**:
```bash
customer-success-orchestrator --project release-playbook
```

**Subagents**:
| Agent | Model | Temp | Purpose |
|-------|-------|------|---------|
| `customer_success_planner` | GPT-5.2 | 0.10 | Playbook structure |
| `customer_success_drafter` | Claude Sonnet 4.5 | 0.28 | Rollout guides |
| `customer_success_editor` | Claude Sonnet 4.5 | 0.15 | Customer voice |
| `customer_success_finalizer` | Gemini Flash 3 | 0.05 | Playbook audit |

**Output Structure**:
- Release overview
- Migration steps
- Rollout timeline
- Support resources
- FAQ

---

### Video Integration

**Orchestrator**: `video-integration-orchestrator`

**Pipeline**:
```bash
video-integration-orchestrator --project my-video
```

**Subagents**:
| Agent | Model | Temp | Purpose |
|-------|-------|------|---------|
| `integration_facts_builder` | Gemini Flash 3 | 0.05 | Workflow extraction |
| `video_outline_builder` | Claude Sonnet 4.5 | 0.15 | Outline creation |
| `video_validator` | Claude Sonnet 4.5 | 0.05 | Prerequisite checks |
| `video_planner` | GPT-5.2 | 0.10 | Timing + captions |
| `video_tts` | Gemini Flash 3 | 0.05 | TTS coordination |
| `video_renderer` | Gemini Flash 3 | 0.05 | FFmpeg composition |
| `video_qa` | Claude Sonnet 4.5 | 0.10 | Quality verification |

**Output**: Walkthrough video with:
- Narration (TTS)
- Screen recording references
- Captions
- Timing marks

---

## Anti-LLM Discipline

### Hard Blocks (50+ phrases)

Never use:
- "In today's fast-paced world"
- "Game-changer"
- "Revolutionary"
- "Cutting-edge"
- "Seamlessly"
- "Robust"
- "Leverage" (as verb)
- "Synergy"
- "Best-in-class"
- "World-class"

### Frequency Limits

| Element | Limit | Rationale |
|---------|-------|-----------|
| Em dashes | 1-2 per piece | LLM tell |
| Rhetorical questions | 1-2 per piece | Overused |
| Sentence starters with "And" | 1 per piece | Can feel forced |

### Structural Tells

Avoid:
- **Hedge stacking**: "might potentially possibly"
- **Filler verbs**: "helps to", "serves to", "works to"
- **Empty connectives**: "Furthermore", "Moreover", "Additionally"
- **False dichotomies**: "It's not X, it's Y" (when it's both)

### Voice Rules

- Active voice (>90%)
- Specific over vague
- Show, don't tell
- Evidence before claims
- No superlatives without proof

---

## Validation System

### Linter

```bash
marketing_lint \
  --project-dir ~/Marketing/_staging/my-project \
  --content-type blog
```

**Checks**:
- Anti-LLM phrase detection
- Frequency limit violations
- Citation completeness
- Persona name leakage
- Word count compliance

### Citation Validator

```bash
marketing_check_citations --project my-project
```

**Validates**:
- All fact IDs exist in fact bank
- Citations have synthesis (not just dropped)
- No orphaned facts

### Quality Gates

Every pipeline validates:
- Required sections present
- Citations have synthesis
- No persona names in final copy
- No LLM tells
- Word count in range

---

## Workspace Structure

```
~/Marketing/
├── _staging/
│   └── {project}/
│       ├── scratchpad.md       # Input
│       ├── fact_bank.md        # Evidence
│       ├── persona_brief.md    # Persona mapping
│       ├── market_research.md  # Market intel
│       ├── plan-{type}.md      # Outline
│       ├── draft-{type}.md     # First draft
│       ├── edited-{type}.md    # Polished
│       └── final-{type}.md     # Publication ready
├── .opencode/
│   ├── agent/
│   │   ├── blog-orchestrator.md
│   │   ├── whitepaper-orchestrator.md
│   │   ├── ... (all agents)
│   ├── context/
│   │   ├── style-guide.md
│   │   ├── personas.md
│   │   └── anti-llm-rules.md
│   └── scripts/
│       ├── marketing_lint.py
│       └── marketing_validate.py
└── published/
    └── (final outputs)
```

---

## Quick Start

### Create a Blog Post

```bash
# 1. Create project
mkdir -p ~/Marketing/_staging/my-post
cat > ~/Marketing/_staging/my-post/scratchpad.md << 'EOF'
# Topic: Why API Response Time Matters

## Key Points
- p99 latency of 45ms under 10k RPS
- 40% cost reduction reported by customers
- Comparison to industry average (200ms)

## Target Audience
Technical architects evaluating API solutions
EOF

# 2. Run pipeline
blog-orchestrator --project my-post

# 3. Check output
cat ~/Marketing/_staging/my-post/final-blog.md
```

### Create a Whitepaper

```bash
# Standard (2,000-2,600 words)
whitepaper-orchestrator --project my-whitepaper

# Extended (5,000-7,500 words)
whitepaper-orchestrator --project my-whitepaper --mode extended
```

### Run Validation

```bash
# Lint for anti-LLM issues
marketing_lint --project-dir ~/Marketing/_staging/my-post --content-type blog

# Check citations
marketing_check_citations --project my-post
```

---

## Related

- **[Patterns](../../patterns/orchestration.md)** - Phase-based orchestration
- **[Evidence Discipline](../../patterns/evidence-discipline.md)** - Fact tracking
- **[Examples](../../examples/marketing/)** - Real usage examples
- **[Agent Index](../../reference/agent-index.md)** - All agents A-Z
