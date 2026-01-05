# Delegation Pattern

When and how to delegate work to subagents.

---

## Core Principle

Delegate when:
- Task benefits from specialized expertise
- Work would interrupt orchestrator flow
- Parallel execution improves throughput

Don't delegate when:
- Simple tool call suffices
- Immediate response needed
- Context would be lost in handoff

---

## Decision Framework

```
Should I delegate this task?
├── Is it a simple lookup or tool call?
│   └── No → Do it yourself
├── Does it need specialized expertise?
│   └── Yes → Delegate
├── Would it interrupt your flow significantly?
│   └── Yes → Delegate (background)
├── Does it benefit from parallel execution?
│   └── Yes → Delegate multiple
└── Default: Do it yourself
```

---

## Delegation Mechanics

### Basic Pattern

```javascript
task({
  description: "Brief description (3-5 words)",
  subagent_type: "agent-name",
  prompt: `
    1. TASK: What to accomplish
    2. EXPECTED OUTCOME: What to return
    3. CONTEXT:
       - Relevant variables
       - File paths
       - Constraints
  `
})
```

### Background Execution

```javascript
// Fire and forget, check later
task({
  description: "Research X",
  subagent_type: "silas-research",
  run_in_background: true,
  prompt: "..."
})

// Continue with other work...

// Check result when needed
const result = await taskOutput(taskId)
```

### Parallel Delegation

```javascript
// Multiple subagents at once
const [facts, personas, research] = await Promise.all([
  task({ subagent_type: "fact_bank_builder", ... }),
  task({ subagent_type: "persona_calibrator", ... }),
  task({ subagent_type: "marketing_research_scout", ... })
])
```

---

## Prompt Structure

### Effective Delegation Prompt

```markdown
1. TASK: [Clear, specific goal]

2. EXPECTED OUTCOME:
   - Format of output
   - What to include
   - What NOT to include

3. REQUIRED TOOLS: [List specific tools if needed]

4. MUST DO:
   - Use absolute paths
   - Write output to specified path
   - Preserve literal quotes

5. MUST NOT DO:
   - Invent facts
   - Skip validation
   - Modify source files

6. CONTEXT:
   - project: ${project}
   - input: ${inputPath}
   - output: ${outputPath}
```

### Anti-Pattern: Vague Prompt

```javascript
// Bad
task({
  subagent_type: "silas-research",
  prompt: "Find out about competitors"
})

// Good
task({
  subagent_type: "silas-research",
  prompt: `
    Research top 3 competitors for API monitoring.

    For each competitor, find:
    - Pricing model
    - Key differentiators
    - Recent product launches (last 6 months)

    Return: Markdown table with columns for each competitor.
    Sources: Include URLs for claims.
  `
})
```

---

## Domain-Specific Patterns

### Ops (Silas)

| Subagent | When to Use |
|----------|-------------|
| `silas-research` | Multi-source lookup, deep exploration |
| `silas-drafter` | Multi-paragraph writing, complex composition |
| `silas-memory` | Learning cycles, pattern review |
| `silas-meeting-prep` | Meeting briefings, follow-ups |
| `silas-comms` | Quick voice-matched messages |

**Example**:
```javascript
// Meeting prep delegation
task({
  subagent_type: "silas-meeting-prep",
  description: "Prep for Q1 meeting",
  prompt: `
    Meeting: Q1 Planning with Sarah
    Time: Tomorrow 10am

    Gather:
    - Sarah's person record
    - Recent Jira tickets (Q1)
    - Prior meeting notes

    Return: Briefing with context + suggested agenda
  `
})
```

### Documentation

| Subagent | When to Use |
|----------|-------------|
| `docs-detective` | Gap analysis (can run parallel) |
| `docs-architect` | Blueprint creation (blocking) |
| `docs-writer` | Draft production |
| `docs-recovery` | Validation error fixes |

**Example**:
```javascript
// Parallel evidence gathering
const [detective, architect] = await Promise.all([
  task({
    subagent_type: "docs-detective",
    prompt: "Find gaps in scratchpad.md → questions.md"
  }),
  task({
    subagent_type: "docs-architect",
    prompt: "Create blueprint from scratchpad.md → blueprint.md"
  })
])
```

### Marketing

| Subagent | When to Use |
|----------|-------------|
| `fact_bank_builder` | Every pipeline (extract facts) |
| `persona_calibrator` | Every pipeline (map facts) |
| `*_planner` | Create outlines |
| `*_drafter` | Write content |
| `*_editor` | Polish + validate |
| `*_publisher` | Transform to final |

**Example**:
```javascript
// Evidence gathering phase (parallel)
await Promise.all([
  task({ subagent_type: "fact_bank_builder", ... }),
  task({ subagent_type: "persona_calibrator", ... }),
  task({ subagent_type: "marketing_research_scout", ... })
])

// Sequential pipeline
await task({ subagent_type: "blog_planner", ... })
await task({ subagent_type: "blog_drafter", ... })
await task({ subagent_type: "blog_editor", ... })
await task({ subagent_type: "blog_publisher", ... })
```

---

## Rules

### 1. Background by Default

Subagents run asynchronously. Check results when needed.

```javascript
// Good: Fire, continue, check later
const taskId = task({ run_in_background: true, ... })
// ... do other work ...
const result = await taskOutput(taskId)
```

### 2. Be Specific

Tell subagent exactly what you need back.

```javascript
// Bad
prompt: "Research the topic"

// Good
prompt: `
  Research API caching strategies.
  Return: Top 3 approaches with pros/cons.
  Format: Markdown bullet points.
`
```

### 3. Orchestrator Mediates

Subagent output comes to orchestrator, not user.

```javascript
// Subagent returns raw research
const research = await task({ subagent_type: "silas-research", ... })

// Orchestrator formats for user
const briefing = synthesize(research)
return briefing
```

### 4. Quick Lookups Stay Local

Don't delegate simple operations.

```javascript
// Don't delegate
const email = await ops.email.read(id)

// Do delegate
task({ subagent_type: "silas-research", prompt: "Analyze last 50 emails..." })
```

---

## Anti-Patterns

### Over-Delegation

**Problem**: Delegating simple tasks

```javascript
// Bad: Simple lookup
task({
  subagent_type: "silas-research",
  prompt: "Get Sarah's email address"
})

// Good: Direct call
const sarah = await memory.person.get("Sarah")
```

### Under-Specification

**Problem**: Vague prompts lead to wrong output

```javascript
// Bad
prompt: "Write something about the feature"

// Good
prompt: `
  Write 800-word blog post about smart caching.
  Use facts from fact_bank.md.
  Follow style guide in style-guide.md.
  Output: draft-blog.md
`
```

### Missing Context

**Problem**: Subagent lacks necessary information

```javascript
// Bad
prompt: "Update the document"

// Good
prompt: `
  Update: ~/docs/guide.mdx
  Change: Replace "v1.0" with "v2.0"
  Constraint: Don't change code examples
`
```

---

## Related

- **[Orchestration](./orchestration.md)** - Phase-based patterns
- **[Model Routing](./model-routing.md)** - Agent selection
- **[Agent Index](../reference/agent-index.md)** - All agents A-Z
