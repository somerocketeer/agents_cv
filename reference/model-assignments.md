# Model Assignments Reference

Comprehensive model routing strategy across all agents and domains.

---

## Model Distribution

| Model | Use Cases | Agent Count |
|-------|-----------|-------------|
| **Claude Opus 4.5** | Primary orchestration, deep analysis, editing | ~15 |
| **Claude Sonnet 4.5** | Most subagents, prose quality, research | ~60 |
| **GPT-5.2** | Strategic planning, argument structure | ~10 |
| **GPT-5.1** | Publishing, deterministic transforms | ~2 |
| **Gemini Flash 3** | Deterministic operations, fact extraction | ~10 |
| **Gemini 3 Pro** | Long-form drafting (extended whitepapers) | ~1 |

---

## Temperature Guidelines

| Range | Use | Examples |
|-------|-----|----------|
| **0.0** | Deterministic (architecture, recovery) | docs-architect, docs-recovery |
| **0.05** | Precise, structural (copyedit, publisher) | draft-orchestrator, blog_publisher |
| **0.1-0.15** | Careful analysis (research, planner) | persona_calibrator, marketing_research_scout |
| **0.2-0.3** | Balanced (editor, meeting-prep) | silas, blog_editor |
| **0.4-0.5** | Creative (drafter, comms) | silas-drafter, silas-comms |

---

## By Domain

### Ops (Silas)

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `silas` | Claude Opus 4.5 | 0.2 | Primary orchestration, balanced decision-making |
| `silas-research` | Claude Sonnet 4.5 | 0.3 | Exploratory research, synthesis |
| `silas-drafter` | Gemini 3 Pro | 0.4 | Creative long-form writing (long context) |
| `silas-memory` | Claude Sonnet 4.5 | 0.15 | Pattern analysis, learning cycles |
| `silas-meeting-prep` | Claude Sonnet 4.5 | 0.2 | Structured briefings |
| `silas-comms` | Claude Sonnet 4.5 | 0.4 | Voice-matched quick messages |
| `suggestions` | Claude Sonnet 4.5 | 0.3 | Proactive recommendations |
| `weekly-review` | Claude Sonnet 4.5 | 0.3 | Weekly planning and patterns |

### Documentation

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `draft-orchestrator` | Claude Sonnet 4.5 | 0.05 | Precise phase coordination |
| `editor-orchestrator` | Claude Sonnet 4.5 | 0.05 | Deterministic editing workflow |
| `playwright-orchestrator` | Claude Sonnet 4.5 | 0.05 | Test execution coordination |
| `docs-architect` | Claude Sonnet 4.5 | 0.0 | Structural blueprint creation |
| `docs-detective` | Claude Sonnet 4.5 | 0.15 | Gap analysis, critical thinking |
| `docs-writer` | Claude Sonnet 4.5 | 0.10 | Controlled prose generation |
| `docs-copyedit` | Claude Sonnet 4.5 | 0.05 | Style compliance |
| `docs-recovery` | Claude Sonnet 4.5 | 0.0 | Deterministic error fixing |
| `copy-editor` | Claude Sonnet 4.5 | 0.05 | Standalone polish |
| `detective` | Claude Sonnet 4.5 | 0.15 | Standalone gap finding |
| `reference-generator` | Claude Sonnet 4.5 | 0.10 | Table generation |
| `release-notes` | Claude Sonnet 4.5 | 0.10 | Structured extraction |
| `atlassian` | Claude Sonnet 4.5 | 0.15 | Jira/Confluence operations |
| `docflow` | Claude Sonnet 4.5 | 0.10 | Workflow automation |
| `playwriter-parser` | Claude Sonnet 4.5 | 0.05 | JSON generation |
| `playwriter-resolver-text` | Claude Sonnet 4.5 | 0.05 | Selector resolution |
| `playwriter-debugger` | Claude Sonnet 4.5 | 0.10 | Failure analysis |
| `playwriter-doc-updater` | Claude Sonnet 4.5 | 0.10 | Doc synchronization |
| `playwriter-map-generator` | Claude Sonnet 4.5 | 0.05 | Selector mapping |

### Marketing

#### Orchestrators

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `blog-orchestrator` | Claude Sonnet 4.5 | 0.05 | Pipeline coordination |
| `whitepaper-orchestrator` | Claude Sonnet 4.5 | 0.05 | Pipeline coordination |
| `datasheet-orchestrator` | Claude Sonnet 4.5 | 0.05 | Pipeline coordination |
| `solution-brief-orchestrator` | Claude Sonnet 4.5 | 0.05 | Pipeline coordination |
| `newsletter-orchestrator` | Claude Sonnet 4.5 | 0.05 | Pipeline coordination |
| `linkedin-orchestrator` | Claude Sonnet 4.5 | 0.05 | Pipeline coordination |
| `customer-success-orchestrator` | Claude Sonnet 4.5 | 0.05 | Pipeline coordination |
| `video-integration-orchestrator` | Claude Sonnet 4.5 | 0.05 | Pipeline coordination |

#### Shared Subagents

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `fact_bank_builder` | Gemini Flash 3 | 0.05 | Structured fact extraction |
| `persona_calibrator` | Claude Sonnet 4.5 | 0.10 | Persona-to-fact mapping |
| `marketing_research_scout` | Claude Sonnet 4.5 | 0.15 | Web search synthesis |

#### Blog Pipeline

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `blog_planner` | GPT-5.2 | 0.10 | Argument structure |
| `blog_drafter` | Claude Sonnet 4.5 | 0.28 | Prose quality |
| `blog_editor` | Claude Sonnet 4.5 | 0.15 | Style analysis + polish |
| `blog_publisher` | Gemini Flash 3 | 0.05 | Deterministic transform |

#### Whitepaper Pipeline

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `whitepaper_planner` | GPT-5.2 | 0.10 | Argument structure |
| `whitepaper_drafter` | Claude Sonnet 4.5 | 0.28 | Synthesis-first prose |
| `whitepaper_assembler` | Gemini 3 Pro | 0.20 | Long-form stitching |
| `whitepaper_editor` | Claude Sonnet 4.5 | 0.15 | Audit and polish |
| `whitepaper_publisher` | Gemini Flash 3 | 0.05 | Deterministic transform |

#### Datasheet Pipeline

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `datasheet_planner` | GPT-5.2 | 0.10 | Spec organization |
| `datasheet_drafter` | Claude Sonnet 4.5 | 0.20 | Technical writing |
| `datasheet_editor` | Claude Sonnet 4.5 | 0.10 | Accuracy review |
| `datasheet_finalizer` | Gemini Flash 3 | 0.05 | Spec verification |

#### Solution Brief Pipeline

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `scratchpad_normalizer` | Gemini Flash 3 | 0.05 | Content normalization |
| `solution_brief_planner` | GPT-5.2 | 0.10 | Executive narrative |
| `solution_brief_drafter` | Claude Sonnet 4.5 | 0.28 | Executive prose |
| `solution_brief_editor` | Claude Sonnet 4.5 | 0.15 | Tone review |
| `solution_brief_finalizer` | Gemini Flash 3 | 0.05 | Release audit |

#### Newsletter Pipeline

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `newsletter_planner` | GPT-5.2 | 0.10 | Thesis-driven structure |
| `newsletter_drafter` | Claude Sonnet 4.5 | 0.28 | Insight-led prose |
| `content_editor` | Claude Sonnet 4.5 | 0.15 | Polish + de-LLM |
| `newsletter_finalizer` | Gemini Flash 3 | 0.05 | Final QA |

#### LinkedIn Pipeline

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `linkedin_post_planner` | GPT-5.2 | 0.10 | Hook-first structure |
| `linkedin_drafter` | Claude Sonnet 4.5 | 0.30 | Insight-driven prose |
| `linkedin_editor` | Claude Sonnet 4.5 | 0.15 | Polish + de-LLM |
| `linkedin_finalizer` | Gemini Flash 3 | 0.05 | Validation |

#### Customer Success Pipeline

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `customer_success_planner` | GPT-5.2 | 0.10 | Playbook structure |
| `customer_success_drafter` | Claude Sonnet 4.5 | 0.28 | Rollout guides |
| `customer_success_editor` | Claude Sonnet 4.5 | 0.15 | Customer voice |
| `customer_success_finalizer` | Gemini Flash 3 | 0.05 | Playbook audit |

#### Video Pipeline

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `integration_facts_builder` | Gemini Flash 3 | 0.05 | Workflow extraction |
| `video_outline_builder` | Claude Sonnet 4.5 | 0.15 | Outline creation |
| `video_validator` | Claude Sonnet 4.5 | 0.05 | Prerequisite checks |
| `video_planner` | GPT-5.2 | 0.10 | Timing + captions |
| `video_tts` | Gemini Flash 3 | 0.05 | TTS coordination |
| `video_renderer` | Gemini Flash 3 | 0.05 | FFmpeg composition |
| `video_qa` | Claude Sonnet 4.5 | 0.10 | Quality verification |

### Blog & Design

| Agent | Model | Temp | Rationale |
|-------|-------|------|-----------|
| `blog_interviewer` | Claude Sonnet 4.5 | 0.3 | Conversational, adaptive questioning |
| `blog_planner` | GPT-5.2 | 0.1 | Strategic structure, outline building |
| `blog_drafter` | Claude Sonnet 4.5 | 0.28 | Prose quality, voice authenticity |
| `blog_editor` | Claude Opus 4.5 | 0.15 | Deep voice analysis, anti-LLM check |
| `blog_publisher` | GPT-5.1 | 0.05 | Deterministic MDX formatting |
| `voice_check_interviewer` | Claude Sonnet 4.5 | 0.3 | Gap-filling, asks for specifics |
| `mdx_component_builder` | Gemini Flash 3 | 0.15 | Interactive components, visualizations |
| `blog_designer` | Claude Opus 4.5 | 0.15 | Primary orchestrator for design work |
| `design_auditor` | Claude Sonnet 4.5 | 0.2 | Systematic UI/UX analysis, finds issues |
| `design_implementer` | Claude Sonnet 4.5 | 0.15 | Focused code changes |
| `design_reviewer` | Claude Sonnet 4.5 | 0.1 | Verifies a11y + design system compliance |
| `blog_copywriter` | Claude Sonnet 4.5 | 0.25 | Marketing copy, landing pages, microcopy |

---

## Model Selection Rationale

### Claude Opus 4.5
**Use for**: Primary orchestration, deep analysis, critical editing

**Strengths**:
- Superior reasoning and planning
- Excellent at maintaining context
- Strong voice analysis
- Best for high-stakes decisions

**Agents**: silas, blog_editor, blog_designer

### Claude Sonnet 4.5
**Use for**: Most subagents, prose quality, research

**Strengths**:
- Excellent prose generation
- Strong analytical capabilities
- Good balance of quality and speed
- Reliable for structured tasks

**Agents**: 60+ across all domains

### GPT-5.2
**Use for**: Strategic planning, argument structure

**Strengths**:
- Superior logical structure
- Excellent at argument construction
- Strong outline generation
- Consistent planning quality

**Agents**: All *_planner subagents

### GPT-5.1
**Use for**: Publishing, deterministic transforms

**Strengths**:
- Highly deterministic
- Excellent at format conversion
- Reliable for final publishing

**Agents**: blog_publisher (Blog & Design)

### Gemini Flash 3
**Use for**: Deterministic operations, fact extraction

**Strengths**:
- Fast and cost-effective
- Excellent at structured extraction
- Reliable for deterministic tasks
- Good for technical operations

**Agents**: fact_bank_builder, all *_finalizer subagents, video_tts, video_renderer

### Gemini 3 Pro
**Use for**: Long-form drafting (extended whitepapers)

**Strengths**:
- Handles very long contexts
- Good for extended content stitching
- Cost-effective for long-form

**Agents**: whitepaper_assembler

---

## Temperature Strategy

### Deterministic (0.0-0.05)
**Purpose**: Structural tasks, error recovery, publishing

**Characteristics**:
- Minimal variation
- Predictable output
- Format compliance
- Error-free execution

**Examples**: docs-architect (0.0), docs-recovery (0.0), blog_publisher (0.05)

### Precise (0.1-0.15)
**Purpose**: Analysis, planning, research

**Characteristics**:
- Controlled creativity
- Analytical depth
- Structured thinking
- Reliable patterns

**Examples**: persona_calibrator (0.10), marketing_research_scout (0.15), blog_editor (0.15)

### Balanced (0.2-0.3)
**Purpose**: Editing, meeting prep, general orchestration

**Characteristics**:
- Natural variation
- Contextual adaptation
- Balanced creativity
- Appropriate tone

**Examples**: silas (0.2), blog_drafter (0.28), blog_interviewer (0.3)

### Creative (0.4-0.5)
**Purpose**: Drafting, communications, creative writing

**Characteristics**:
- High variation
- Natural prose
- Voice authenticity
- Engaging content

**Examples**: silas-comms (0.4), silas-drafter (0.5)

---

## Cost Optimization

### High-Volume Operations
Use Gemini Flash 3 for:
- Fact extraction (fact_bank_builder)
- Final validation (*_finalizer)
- Technical operations (video_tts, video_renderer)

### Strategic Operations
Use GPT-5.2 for:
- Planning (*_planner)
- Argument structure
- Outline generation

### Quality-Critical Operations
Use Claude Opus 4.5 for:
- Primary orchestration (silas, blog_designer)
- Deep editing (blog_editor)
- Critical analysis

### General Operations
Use Claude Sonnet 4.5 for:
- Most subagents
- Prose generation
- Research and analysis

---

## Model Routing Enforcement

Model assignments are enforced via `opencode.json` configuration, not in Task prompts.

**Example**:
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
    }
  }
}
```

**Do not override model in Task calls** - use `opencode.json` configuration.

---

## Related

- **[Agent Index](./agent-index.md)** - Alphabetical agent list
- **[Patterns](../patterns/model-routing.md)** - Model routing patterns
- **[Examples](../examples/)** - Real usage examples
