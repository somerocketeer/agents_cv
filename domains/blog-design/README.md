# Blog & Design Domain

Personal blog production with teaching-oriented voice and neobrutalist design system maintenance.

---

## Overview

**Orchestrators**: 1
- `blog_designer` - Orchestrate design work via subagents

**Blog Pipeline Agents**: 6
- `blog_interviewer` - Gather context via questions
- `blog_planner` - Create destination-first outlines
- `blog_drafter` - Write in teaching voice
- `blog_editor` - Voice check, de-LLM, polish
- `blog_publisher` - Format to MDX with frontmatter
- `voice_check_interviewer` - Ask for more context when thin

**Design System Agents**: 4
- `design_auditor` - Systematic UI/UX analysis
- `design_implementer` - Focused code changes
- `design_reviewer` - Verify a11y + design system compliance
- `blog_copywriter` - Marketing copy, landing pages, microcopy

**Utility Agents**: 1
- `mdx_component_builder` - Create interactive components

---

## Architecture

### Blog Pipeline

```
Interview → Plan → Draft → Edit → Publish
    ↓         ↓      ↓       ↓       ↓
interviewer planner drafter editor publisher
```

### Design Pipeline

```
Audit → Implement → Review
   ↓        ↓         ↓
auditor implementer reviewer
```

### Phase Breakdown

| Phase | Agent | Purpose |
|-------|-------|---------|
| **Interview** | blog_interviewer | Gather context via structured questions |
| **Plan** | blog_planner | Create destination-first outline + headlines |
| **Draft** | blog_drafter | Write in teaching voice (1,000-2,000 words) |
| **Edit** | blog_editor | 3-pass edit (voice → meat → anti-LLM) |
| **Publish** | blog_publisher | Internal linking + MDX formatting |

---

## Blog Pipeline

### blog_interviewer

**Purpose**: Gather context via structured questions

**Model**: Claude Sonnet 4.5 (temp 0.3)

**Input**: Topic or rough idea

**Output**: `interview-brief.md` with:
- Core concept
- Target audience
- Key points to cover
- Examples/stories
- Desired outcome for reader

**Example**:
```bash
blog_interviewer --topic "Understanding Agents"
```

**Interview Flow**:
1. What's the core idea?
2. Who is this for?
3. What should they be able to do after reading?
4. What examples or stories illustrate this?
5. What's the one thing they must remember?

---

### blog_planner

**Purpose**: Create destination-first outline with headline variants

**Model**: GPT-5.2 (temp 0.1)

**Input**: `interview-brief.md`

**Output**: `plan-blog.md` with:
- 5 headline variants
- Destination-first structure
- Section breakdown
- Key points per section
- Internal link opportunities

**Example**:
```bash
blog_planner --interview-brief interview-brief.md
```

**Destination-First Structure**:
```markdown
## Headline Variants
1. "Agents Aren't Magic: A Practical Guide"
2. "What I Learned Building 90 AI Agents"
3. ...

## Structure

### Opening (hook + destination)
Reader arrives at: Understanding what agents actually do

### Section 1: The Simplest Agent
Reader arrives at: "Oh, it's just a loop with tools"

### Section 2: When to Add Complexity
Reader arrives at: Knowing when orchestration helps

### Closing
Reader arrives at: Confident to build their first agent
```

---

### blog_drafter

**Purpose**: Write in Mellisa's teaching voice

**Model**: Claude Sonnet 4.5 (temp 0.28)

**Input**: `plan-blog.md`, selected headline

**Output**: `draft-blog.md` (1,000-2,000 words)

**Example**:
```bash
blog_drafter --plan plan-blog.md --headline "Agents Aren't Magic: A Practical Guide"
```

**Voice Characteristics**:
- Teaching, not lecturing
- Concrete before abstract
- "You" over "one"
- Show working (not just results)
- Admit uncertainty honestly

**Anti-Patterns**:
- No false dichotomies ("It's not X, it's Y" when it's both)
- No em dashes (0-1 max)
- No passive voice
- No hedge stacking

---

### blog_editor

**Purpose**: 3-pass edit (voice → meat → anti-LLM)

**Model**: Claude Opus 4.5 (temp 0.15)

**Input**: `draft-blog.md`

**Output**: `edited-blog.md`

**Example**:
```bash
blog_editor --draft draft-blog.md
```

**3-Pass Edit**:

| Pass | Focus | Actions |
|------|-------|---------|
| **Voice** | Authenticity | Match teaching voice, fix tone issues |
| **Meat** | Substance | Add specifics, cut fluff, strengthen examples |
| **Anti-LLM** | Detection | Remove tells, fix patterns, final polish |

**Voice Check Triggers**:
If content is thin, triggers `voice_check_interviewer` to gather more context.

---

### blog_publisher

**Purpose**: Format to MDX with frontmatter and internal links

**Model**: GPT-5.1 (temp 0.05)

**Input**: `edited-blog.md`

**Output**: `published-blog.mdx`

**Example**:
```bash
blog_publisher --edited edited-blog.md --slug understanding-agents
```

**Output Format**:
```mdx
---
title: "Agents Aren't Magic: A Practical Guide"
description: "A practical introduction to building AI agents"
date: "2025-01-04"
tags: ["agents", "ai", "practical"]
---

# Agents Aren't Magic: A Practical Guide

Content with [internal links](/related-post) and proper MDX formatting...
```

---

### voice_check_interviewer

**Purpose**: Ask for more context when content is thin

**Model**: Claude Sonnet 4.5 (temp 0.3)

**Triggered By**: `blog_editor` when draft lacks specifics

**Questions**:
- Can you give a specific example of X?
- What's the story behind Y?
- How did you discover this?
- What mistake do people commonly make here?

---

## Design System

### blog_designer

**Purpose**: Orchestrate design work via subagents

**Model**: Claude Opus 4.5 (temp 0.15)

**Capabilities**:
- Coordinate design audits
- Manage implementation tasks
- Verify compliance

**Example**:
```bash
blog_designer --task "Update button component for better a11y"
```

---

### design_auditor

**Purpose**: Systematic UI/UX analysis, finds issues

**Model**: Claude Sonnet 4.5 (temp 0.2)

**Input**: Component or page path

**Output**: Audit report with:
- Accessibility issues
- Design system violations
- UX problems
- Recommendations

**Example**:
```bash
design_auditor --component src/components/Button.tsx
```

**Audit Checklist**:
- Color contrast (WCAG AA)
- Focus states
- Keyboard navigation
- Screen reader labels
- Design token usage
- Spacing consistency
- Typography hierarchy

---

### design_implementer

**Purpose**: Focused code changes

**Model**: Claude Sonnet 4.5 (temp 0.15)

**Input**: Implementation task from audit

**Output**: Code changes

**Example**:
```bash
design_implementer --task "Add focus ring to Button component"
```

**Rules**:
- Use design tokens (not raw values)
- Follow component patterns
- Maintain backwards compatibility
- Add/update tests

---

### design_reviewer

**Purpose**: Verify a11y + design system compliance

**Model**: Claude Sonnet 4.5 (temp 0.1)

**Input**: Changed files

**Output**: Review report

**Example**:
```bash
design_reviewer --files src/components/Button.tsx
```

**Verification**:
- Design token usage
- Accessibility compliance
- Pattern consistency
- Test coverage

---

### blog_copywriter

**Purpose**: Marketing copy, landing pages, microcopy

**Model**: Claude Sonnet 4.5 (temp 0.25)

**Output Types**:
- Landing page copy
- Feature descriptions
- Call-to-action text
- Error messages
- Empty states

**Voice**: Friendly, clear, action-oriented

---

## Utility Agents

### mdx_component_builder

**Purpose**: Create interactive components for tough concepts

**Model**: Gemini Flash 3 (temp 0.15)

**Input**: Concept to visualize

**Output**: MDX component with:
- React component code
- Interactive elements
- Accessibility support

**Example**:
```bash
mdx_component_builder --concept "Agent orchestration flow"
```

**Use Cases**:
- Interactive diagrams
- Code playgrounds
- Step-by-step visualizations
- Comparison tables

---

## Design System Reference

### Neobrutalist Principles

| Principle | Implementation |
|-----------|----------------|
| **Bold borders** | 2-4px solid black |
| **Offset shadows** | 4px 4px 0 black |
| **High contrast** | Black on white, accent colors |
| **Chunky spacing** | 8px base unit |
| **Visible structure** | No hidden UI patterns |

### Design Tokens

```css
/* Colors */
--color-primary: #000000;
--color-background: #ffffff;
--color-accent: #ff6b6b;

/* Spacing */
--space-1: 8px;
--space-2: 16px;
--space-3: 24px;
--space-4: 32px;

/* Borders */
--border-width: 2px;
--border-radius: 0;

/* Shadows */
--shadow-offset: 4px 4px 0 var(--color-primary);
```

### Component Patterns

**Button**:
```tsx
<button className="
  border-2 border-black
  px-4 py-2
  shadow-[4px_4px_0_black]
  hover:shadow-[2px_2px_0_black]
  hover:translate-x-[2px]
  hover:translate-y-[2px]
  transition-all
">
  Click me
</button>
```

**Card**:
```tsx
<div className="
  border-2 border-black
  p-4
  shadow-[4px_4px_0_black]
">
  Content
</div>
```

---

## Voice Guide

### Teaching Voice Characteristics

| Do | Don't |
|----|-------|
| "Here's how I think about it..." | "It is important to note that..." |
| "You might be wondering..." | "One may observe that..." |
| "Let me show you" | "The following demonstrates" |
| "I got this wrong at first" | "Common mistakes include" |

### Anti-LLM Rules

**Hard Blocks**:
- Em dashes (max 0-1)
- "In order to" → "to"
- "It's important to note" → cut
- "Let's dive in" → cut
- Passive voice → active

**Frequency Limits**:
| Element | Limit |
|---------|-------|
| Em dashes | 0-1 |
| Rhetorical questions | 1-2 |
| "You" per paragraph | 1-2 |

---

## File Layout

```
~/Code/context-systems/
├── .opencode/
│   ├── agent/
│   │   ├── blog_interviewer.md
│   │   ├── blog_planner.md
│   │   ├── blog_drafter.md
│   │   ├── blog_editor.md
│   │   ├── blog_publisher.md
│   │   ├── voice_check_interviewer.md
│   │   ├── blog_designer.md
│   │   ├── design_auditor.md
│   │   ├── design_implementer.md
│   │   ├── design_reviewer.md
│   │   ├── blog_copywriter.md
│   │   └── mdx_component_builder.md
│   └── context/
│       ├── voice-guide.md
│       ├── design-system.md
│       └── anti-llm-rules.md
├── src/
│   ├── components/
│   └── content/
│       └── blog/
└── public/
```

---

## Quick Start

### Write a Blog Post

```bash
# 1. Interview
blog_interviewer --topic "Understanding Agents"

# 2. Plan (select headline after)
blog_planner --interview-brief interview-brief.md

# 3. Draft
blog_drafter --plan plan-blog.md --headline "Agents Aren't Magic"

# 4. Edit (3-pass)
blog_editor --draft draft-blog.md

# 5. Publish
blog_publisher --edited edited-blog.md --slug understanding-agents
```

### Design Audit

```bash
# Audit a component
design_auditor --component src/components/Button.tsx

# Implement fixes
design_implementer --task "Add focus ring to Button"

# Verify changes
design_reviewer --files src/components/Button.tsx
```

### Create Interactive Component

```bash
mdx_component_builder --concept "Agent orchestration flow"
```

---

## Related

- **[Patterns](../../patterns/orchestration.md)** - Phase-based orchestration
- **[Examples](../../examples/blog-design/)** - Real usage examples
- **[Agent Index](../../reference/agent-index.md)** - All agents A-Z
