# Quick Start Guide

Get up and running with the agent ecosystem in 5 minutes.

---

## Prerequisites

- **For Ops (Silas)**: Access to `~/Ops` directory, MCP server running
- **For Documentation**: Access to `~/Code/devcontainer/.opencode`
- **For Marketing**: Access to `~/Marketing` directory
- **For Blog**: Access to `~/Code/context-systems`

---

## 1. Ops (Silas) - Executive Assistant

### Start a Session

```javascript
// Silas loads memory automatically at session start
const state = await memory.load()
```

### Common Operations

```javascript
// Record an observation
memory.observe("correction", {
  what_was_wrong: "You said the meeting was at 2pm",
  what_was_expected: "It's at 3pm",
  category: "calendar"
})

// Check CRM
const person = await memory.person.get("Sarah")
const stale = await memory.person.stale(14) // People not contacted in 14 days

// Search memory
const results = await memory.recall("What do I usually do on Fridays?")

// Update context
memory.context.set("primary_project", "Q1 Planning")
```

### Commands

```bash
/morning              # Daily briefing
/prep "Meeting topic" # Meeting preparation
/todos                # Energy-typed task list
/weekly-review        # Pattern confirmation
```

### Delegation

```javascript
// Delegate to subagents
task({
  subagent_type: "silas-research",
  description: "Research X",
  prompt: "Find information about X. Return synthesized findings."
})
```

---

## 2. Documentation Pipeline

### Draft from Scratchpad

```bash
# Input: scratchpad.md
# Output: draft.mdx with code placeholders

draft-orchestrator \
  --input scratchpad.md \
  --output draft.mdx
```

**Pipeline**: EXTRACT → ANALYZE → DRAFT → INJECT → VALIDATE

### Copyedit Existing Docs

```bash
# Single-pass polish

editor-orchestrator \
  --target docs/guide.mdx
```

**Fixes**: Passive voice, long sentences, filler words, code context

### Verify with Playwright

```bash
# Convert docs → workflow → test → update docs

playwright-orchestrator \
  --workflow workflow.json
```

---

## 3. Marketing Content Production

### Blog Post (800-1,400 words)

```bash
# Create project directory
mkdir -p ~/Marketing/_staging/my-blog-post
echo "# Topic\n\nRaw notes here..." > ~/Marketing/_staging/my-blog-post/scratchpad.md

# Run pipeline
blog-orchestrator --project my-blog-post
```

**Output**: `final-blog.md` with numbered references

### Whitepaper (2,000-7,500 words)

```bash
# Standard mode (2,000-2,600 words)
whitepaper-orchestrator --project my-whitepaper

# Extended mode (5,000-7,500 words)
whitepaper-orchestrator --project my-whitepaper --mode extended
```

### LinkedIn Campaign

```bash
linkedin-orchestrator --project my-campaign
```

**Output**: 1-4 posts with hook-first structure

### Newsletter

```bash
newsletter-orchestrator --project q1-update
```

**Output**: Insight-led newsletter with architectural framing

---

## 4. Blog & Design System

### Full Blog Pipeline

```bash
# Interactive interview
blog_interviewer --topic "Understanding Agents"

# Generate outline + 5 headline variants
blog_planner --interview-brief interview-brief.md

# Draft (1,000-2,000 words)
blog_drafter --plan plan-blog.md --headline "Selected Headline"

# 3-pass edit (voice → meat → anti-LLM)
blog_editor --draft draft-blog.md

# Publish to MDX
blog_publisher --edited edited-blog.md --slug understanding-agents
```

### Design Audit

```bash
# Systematic UI/UX analysis
design_auditor --component src/components/Button.tsx
```

### Design Implementation

```bash
# Focused code changes
design_implementer --task "Update button focus states"

# Verify compliance
design_reviewer --files src/components/Button.tsx
```

---

## Common Patterns

### 1. Phase-Based Execution

All orchestrators follow:
```
Setup → Evidence → Planning → Execution → Validation → Publishing
```

### 2. Fail-Fast Behavior

- **Blocking errors**: Stop immediately, report
- **Recoverable errors**: Retry once, then stop
- **Warnings**: Log and continue

### 3. Evidence Discipline

```markdown
## Facts

### fact_001: Title
**Source:** scratchpad section
**Personas:** Architect Alice, Technical Teddy
[Fact content with preserved literals]
```

### 4. Model Routing

- **Planning**: GPT-5.2 (temp 0.1)
- **Drafting**: Claude Sonnet 4.5 (temp 0.28)
- **Editing**: Claude Sonnet 4.5 (temp 0.15)
- **Publishing**: Gemini Flash 3 (temp 0.05)

### 5. Quality Gates

Every pipeline validates:
- ✅ Required sections present
- ✅ Citations have synthesis
- ✅ No persona names in final copy
- ✅ No LLM tells (banned phrases, hedge stacking)

---

## Troubleshooting

### Silas Not Remembering

```javascript
// Check memory stats
const stats = await memory.stats()

// Increase retention
memory.config.set("memory.half_life_days", 30, "User forgets too fast")

// Run learning cycle
memory.learn.patterns()
```

### Documentation Validation Failing

```bash
# Check validation errors
python3 .opencode/scripts/validate_draft.py --draft draft.mdx

# Run recovery
docs-recovery --draft draft.mdx --errors validation-errors.json
```

### Marketing Citations Invalid

```bash
# Validate fact bank
marketing_validate_artifact \
  --content-type blog \
  --artifact-type fact_bank \
  --file fact_bank.md

# Check citations
marketing_check_citations --project my-project
```

### Blog Voice Check Failing

```bash
# Run voice check
interview_voiceCheck --draft draft-blog.md --check-type full

# Common issues:
# - False dichotomies ("not X, it's Y")
# - Em dashes (max 0-1)
# - Passive voice (zero tolerance)
```

---

## Next Steps

- **[Domain Guides](./domains/)** - Deep dive into each system
- **[Patterns](./patterns/)** - Learn architectural patterns
- **[Examples](./examples/)** - See real usage examples
- **[Reference](./reference/)** - Quick lookup guides

---

## Getting Help

- **Agent Index**: [reference/agent-index.md](./reference/agent-index.md)
- **Tool Reference**: [reference/tool-reference.md](./reference/tool-reference.md)
- **Model Assignments**: [reference/model-assignments.md](./reference/model-assignments.md)
