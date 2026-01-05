# Tool Reference

Comprehensive reference for tools, sandboxes, and MCP servers across all domains.

---

## Ops Domain

### Memory Tools (MCP)

| Tool | Purpose | Parameters |
|------|---------|------------|
| `silas_time_now` | Current date/time with timezone | None |
| `silas_memory_capabilities` | Report backend versions, enabled features | None |
| `silas_memory_load` | Start session, return profile facts + context | None |
| `silas_memory_observe` | Write structured observations | `type`, `value`, `confidence?` |
| `silas_memory_person` | CRM operations | `action`, `params` |
| `silas_memory_context` | Get/set working memory context | `action`, `key?`, `value?` |
| `silas_memory_config` | Get/set/reset self-tuning config | `action`, `path?`, `value?`, `reason?` |
| `silas_memory_learn` | Trigger background learning | `action` |
| `silas_memory_ingest` | Batch ingest turns/docs/transcripts | `content`, `type` |
| `silas_memory_search` | Direct search (lexical/semantic/stats) | `query`, `mode?` |
| `silas_memory_recall` | Primary recall: facts + episodic snippets | `query` |
| `silas_memory_forget` | User-driven deletion with propagation | `target` |

### Memory Sandbox (JavaScript)

Higher-level interface wrapping MCP tools:

```javascript
// Session
memory.load()                           // Load session snapshot

// Observations
memory.observe(type, value, confidence?) // Record observation

// CRM
memory.person.get(name)                 // Get person record
memory.person.add(data)                 // Add new person
memory.person.update(name, data)        // Update person
memory.person.list()                    // List all people
memory.person.stale(days)               // People not contacted in N days
memory.person.followUps()               // People with pending follow-ups

// Context
memory.context.get(key)                 // Get context value
memory.context.set(key, value)          // Set context value
memory.context.add(key, value)          // Add to multi-value context
memory.context.remove(key, value)       // Remove from multi-value context
memory.context.clear(key)               // Clear context key
memory.context.all()                    // Get all context

// Search
memory.search(query)                    // Lexical search
memory.semantic(query)                  // Vector similarity search
memory.recall(query)                    // Hybrid recall (facts + snippets)
memory.stats()                          // Memory statistics

// Learning
memory.learn.patterns()                 // Trigger pattern analysis
memory.learn.confirm(pattern)           // Confirm a pattern

// Config
memory.config.get(path)                 // Get config value
memory.config.set(path, value, reason)  // Set config value
memory.config.all()                     // Get all config
```

### Ops Sandbox

| Function | Purpose | Example |
|----------|---------|---------|
| `ops.calendar.today()` | Get today's events | Returns events + focus blocks |
| `ops.calendar.range(start, end)` | Get events in range | Date strings YYYY-MM-DD |
| `ops.calendar.create(event)` | Create event | Requires confirmation |
| `ops.calendar.update(id, data)` | Update event | |
| `ops.calendar.delete(id)` | Delete event | Requires confirmation |
| `ops.email.inbox(limit?)` | Get recent emails | Default 20 |
| `ops.email.search(query)` | Search emails | |
| `ops.email.read(id)` | Read email content | |
| `ops.email.draft(to, subject, body)` | Draft email | |
| `ops.email.send(id)` | Send drafted email | Requires confirmation |
| `ops.todos.list()` | Get all todos | |
| `ops.todos.create(title, priority?)` | Create todo | |
| `ops.todos.complete(id)` | Mark complete | |
| `ops.todos.delete(id)` | Delete todo | |
| `ops.reminders.set(text, time)` | Set reminder | |
| `ops.reminders.list()` | Get active reminders | |
| `ops.reminders.cancel(id)` | Cancel reminder | |

### Atlassian Tool

| Function | Purpose |
|----------|---------|
| `atlassian.jira.search(jql)` | Search Jira issues |
| `atlassian.jira.get(key)` | Get issue details |
| `atlassian.jira.create(data)` | Create issue |
| `atlassian.jira.update(key, data)` | Update issue |
| `atlassian.jira.comment(key, body)` | Add comment |
| `atlassian.confluence.search(query)` | Search pages |
| `atlassian.confluence.get(id)` | Get page content |
| `atlassian.confluence.create(data)` | Create page |
| `atlassian.confluence.update(id, data)` | Update page |

---

## Documentation Domain

### Scripts

| Script | Purpose | Usage |
|--------|---------|-------|
| `orchestrator_manifest.py` | Initialize/track orchestrator state | `python3 orchestrator_manifest.py init --input-path X --output-path Y` |
| `extract_code_blocks.py` | Extract code from scratchpad | `python3 extract_code_blocks.py --path scratchpad.md` |
| `inject_code_blocks.py` | Replace placeholders with code | `python3 inject_code_blocks.py --draft draft.mdx --manifest code_blocks.json` |
| `validate_draft.py` | Run unified validation | `python3 validate_draft.py --draft draft.mdx --scratchpad scratchpad.md` |

### Validation Tool

```bash
python3 validate_draft.py \
  --draft draft.mdx \
  --scratchpad scratchpad.md \
  --project-dir ./

# Output: validation.json
{
  "status": "PASS" | "WARNINGS" | "FAIL",
  "blocking_errors": [],
  "warnings": [],
  "recoverable": boolean
}
```

### Playwright Tools

| Tool | Purpose |
|------|---------|
| `playwright.parse(doc)` | Convert documentation to workflow JSON |
| `playwright.resolve(workflow, selectors)` | Resolve selectors from map |
| `playwright.execute(workflow)` | Run workflow against live UI |
| `playwright.screenshot(step)` | Capture step screenshot |
| `playwright.analyze(failure)` | Analyze failure with screenshot |

---

## Marketing Domain

### Marketing Tools

| Tool | Purpose | Usage |
|------|---------|-------|
| `marketing_lint` | Check anti-LLM compliance | `marketing_lint --project-dir X --content-type blog` |
| `marketing_check_citations` | Validate fact references | `marketing_check_citations --project X` |
| `marketing_validate_artifact` | Validate intermediate artifacts | `marketing_validate_artifact --content-type blog --artifact-type fact_bank --file X` |

### Lint Output

```json
{
  "status": "PASS" | "FAIL",
  "checks": {
    "anti_llm": {
      "status": "PASS" | "FAIL",
      "violations": [
        { "phrase": "game-changer", "line": 15 }
      ]
    },
    "citations": {
      "status": "PASS" | "FAIL",
      "missing": [],
      "orphaned": []
    },
    "word_count": {
      "status": "PASS" | "FAIL",
      "count": 892,
      "target": "800-1400"
    },
    "persona_leakage": {
      "status": "PASS" | "FAIL",
      "found": []
    }
  }
}
```

### Style Guide Interface

Marketing agents reference the style guide:

```javascript
const style = {
  anti_llm: {
    banned_phrases: [...],      // 50+ phrases
    frequency_limits: {
      em_dashes: 2,
      rhetorical_questions: 2
    }
  },
  voice: {
    active_voice_min: 0.9,
    max_sentence_length: 25
  },
  citations: {
    synthesis_required: true,
    no_dropped_citations: true
  }
}
```

---

## Blog & Design Domain

### Voice Check Tool

```bash
interview_voiceCheck --draft draft-blog.md --check-type full

# Output
{
  "status": "PASS" | "NEEDS_WORK",
  "issues": [
    { "type": "passive_voice", "line": 12, "text": "was implemented" },
    { "type": "em_dash", "line": 23, "count": 3 }
  ],
  "suggestions": [
    "Line 12: Change to active voice",
    "Line 23: Remove 2 em dashes"
  ]
}
```

### Design System Tokens

```javascript
const tokens = {
  colors: {
    primary: 'var(--color-primary)',        // #000000
    background: 'var(--color-background)',   // #ffffff
    accent: 'var(--color-accent)'            // #ff6b6b
  },
  spacing: {
    1: 'var(--space-1)',  // 8px
    2: 'var(--space-2)',  // 16px
    3: 'var(--space-3)',  // 24px
    4: 'var(--space-4)'   // 32px
  },
  borders: {
    width: 'var(--border-width)',   // 2px
    radius: 'var(--border-radius)'  // 0
  },
  shadows: {
    offset: 'var(--shadow-offset)'  // 4px 4px 0 var(--color-primary)
  }
}
```

---

## Common Tool Patterns

### Task Delegation

All domains use the `task` tool for delegation:

```javascript
task({
  description: "Brief description",
  subagent_type: "agent-name",
  prompt: `
    1. TASK: What to do
    2. EXPECTED OUTCOME: What to return
    3. CONTEXT:
       - project: ${project}
       - paths: ${paths}
  `
})
```

### File Operations

| Tool | Purpose |
|------|---------|
| `read` | Read file contents |
| `write` | Write file contents |
| `bash` | Execute shell commands |

### User Interaction

| Tool | Purpose |
|------|---------|
| `askuser_choice` | Present choices |
| `askuser_confirm` | Get yes/no confirmation |
| `askuser_input` | Get text input |

---

## MCP Server Configuration

### Ops Memory Server

```json
{
  "mcpServers": {
    "silas-memory": {
      "command": "python3",
      "args": ["~/Ops/.opencode/memory-stack/scripts/mcp_memory_server.py"],
      "env": {
        "POSTGRES_URL": "...",
        "QDRANT_URL": "...",
        "NEO4J_URL": "..."
      }
    }
  }
}
```

### Required Services

| Service | Purpose | Port |
|---------|---------|------|
| PostgreSQL | Event log (source of truth) | 5432 |
| Qdrant | Vector store (embeddings) | 6333 |
| Neo4j | Knowledge graph (facts) | 7687 |

---

## Related

- **[Agent Index](./agent-index.md)** - All agents A-Z
- **[Model Assignments](./model-assignments.md)** - Model routing
- **[Patterns](../patterns/)** - Cross-cutting patterns
