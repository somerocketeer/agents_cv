# Ops Domain - Silas Executive Assistant

Silas is an AI-powered executive assistant with persistent memory, autonomous task management, and proactive suggestions.

---

## Overview

**Primary Agent**: `silas` (Claude Opus 4.5, temp 0.2)

**Agents**: 8 total
- `silas` - Primary orchestrator (Claude Opus 4.5, temp 0.2)
- `silas-research` - Deep exploration (Claude Sonnet 4.5, temp 0.3)
- `silas-drafter` - Long-form writing (Gemini 3 Pro, temp 0.4)
- `silas-memory` - Memory curation (Claude Sonnet 4.5, temp 0.2)
- `silas-meeting-prep` - Meeting briefings (Claude Sonnet 4.5, temp 0.3)
- `silas-comms` - Quick messages (Claude Sonnet 4.5, temp 0.5)
- `suggestions` - Morning/calendar/proactive (Claude Sonnet 4.5, temp 0.3)
- `weekly-review` - Weekly planning (Claude Sonnet 4.5, temp 0.3)

**Infrastructure**:
- Memory Stack (Postgres + Qdrant + Neo4j)
- MCP Server (12 tools)
- Ops Sandbox (calendar, email, todos, reminders)
- Scheduled automation (daily briefings, proactive checks)

---

## Architecture

### Memory Stack

```
┌─────────────────────────────────────────────────────────────┐
│             MCP Server (mcp_memory_server.py)               │
│   silas_memory_* tools exposed to agents via MCP protocol   │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│   Postgres    │     │    Qdrant     │     │    Neo4j      │
│  Event Log    │     │ Vector Store  │     │ Knowledge     │
│  (source of   │     │ (embeddings)  │     │ Graph (facts) │
│   truth)      │     │               │     │               │
└───────────────┘     └───────────────┘     └───────────────┘
        │                     ▲                     ▲
        │                     │                     │
        └─────────────────────┴─────────────────────┘
                    Background Workers
              (embed_worker.py, kg_worker.py)
```

### Data Stores

| Store | Purpose | Data |
|-------|---------|------|
| **Postgres** | Event log (source of truth) | All events, deletions, thread state, retrieval audits |
| **Qdrant** | Vector search | Embeddings for semantic retrieval |
| **Neo4j** | Knowledge graph | Entities, relationships, facts with temporal validity |

---

## MCP Tools (12 total)

| Tool | Purpose |
|------|---------|
| `silas_time_now` | Current date/time with timezone |
| `silas_memory_capabilities` | Report backend versions, enabled features |
| `silas_memory_load` | Start session, return profile facts + context |
| `silas_memory_observe` | Write structured observations (type, subtype, value, confidence) |
| `silas_memory_person` | CRM operations (get/add/list/stale/follow_ups) |
| `silas_memory_context` | Get/set working memory context (thread state) |
| `silas_memory_config` | Get/set/reset self-tuning config |
| `silas_memory_learn` | Trigger background learning (runs workers) |
| `silas_memory_ingest` | Batch ingest turns/docs/transcripts |
| `silas_memory_search` | Direct search (lexical/semantic/stats) |
| `silas_memory_recall` | Primary recall: facts + episodic snippets |
| `silas_memory_forget` | User-driven deletion with propagation |

---

## Knowledge Graph Ontology

### Entities

```
User:       singleton_id: "user:primary"
Person:     id_prefix: "person:"
Project:    id_prefix: "project:"
Thread:     id_prefix: "thread:"
Dependency: id_prefix: "dep:"
```

### Predicates (Sample)

| Predicate | Subject | Cardinality | Policy | Type |
|-----------|---------|-------------|--------|------|
| `user.timezone` | User | single | supersede | string |
| `user.work_hours` | User | single | supersede | json |
| `pref.communication.length` | User | single | supersede | string |
| `pref.communication.format` | User | multi | set_union | string |
| `pref.communication.tone` | User | single | supersede | string |
| `pref.time.focus_time` | User | multi | set_union | json |
| `pref.writing.avoid_em_dashes` | User | single | supersede | boolean |
| `person.relationship_to_user` | Person | single | supersede | string |
| `person.communication_style` | Person | single | supersede | string |
| `person.contact_priority` | Person | single | supersede | string |
| `person.next_action` | Person | single | supersede | string |
| `person.topics` | Person | multi | set_union | string |

### Cardinality Policies

- **single + supersede**: New value replaces old (closes previous assertion)
- **multi + set_union**: Values accumulate (multiple assertions valid)

---

## Session Protocol

### On Every Session Start

1. **Load memory (single call)**:
```javascript
const state = await memory.load()
return {
  time: state.current_time,
  focus: state.memory?.current_focus ?? {},
  stats: state.stats ? { people: state.stats.people, patterns: state.stats.patterns } : {},
}
```

2. **Orient silently** - Don't narrate what you're loading
3. **Be ready** - You now know who you're working with

### During Session

**Observe via `memory.observe(type, value, confidence?)`**:

| Signal | Type | Value Fields |
|--------|------|--------------|
| User corrects you | `correction` | `{what_was_wrong, what_was_expected, category}` |
| Time preference | `time_preference` | `{time_of_day, day_of_week, preference: prefer/avoid}` |
| Communication style | `communication` | `{preference, example, context}` |
| Person mentioned | `person_interaction` | `{person_name, relationship, context, sentiment}` |
| Meeting notes ingested | `meeting_notes` | `{participants, topics, action_items_count, summary}` |

**Correction observations are HIGHEST signal.**

### On Session End

1. Update context if things changed: `memory.context.set(...)`
2. Record any notable observations not yet captured

---

## Capabilities

| Domain | Tools | Autonomy Level |
|--------|-------|----------------|
| **Todos** | `ops` | High - create/update freely |
| **Reminders** | `ops` | High - set without asking |
| **Calendar** | `ops` | Read-first - create/delete with confirmation |
| **Email** | `ops` | Medium - read freely, draft with intent (send only with confirmation) |
| **Memory** | `memory` | Internal - user shouldn't see these calls |
| **Atlassian (Jira/Confluence)** | `atlassian` | Medium - read freely, write with context |

---

## Permission Scoping (op1-policy.json)

Silas operates under a permission policy that gates sensitive operations:

| Scope | Permission | Notes |
|-------|------------|-------|
| `todos:read` | ✓ Allowed | |
| `todos:write` | ✓ Allowed | |
| `reminders:read` | ✓ Allowed | |
| `reminders:write` | ✓ Allowed | |
| `email:read` | ✓ Allowed | |
| `email:draft` | ✓ Allowed | |
| `email:send` | ✗ Gated | Requires user confirmation |
| `calendar:read` | ✓ Allowed | |
| `calendar:write` | ✗ Gated | Requires user confirmation |
| `memory:read` | ✓ Allowed | |
| `memory:write` | ✓ Allowed | |
| `exec:shell` | ✗ Denied | Shell execution blocked |
| `network:unrestricted` | ✗ Denied | Network access restricted |

This policy ensures Silas can work autonomously on low-risk tasks while requiring confirmation for actions that could have unintended consequences.

---

## Delegation (Subagents)

### When to Delegate

| Subagent | Trigger | Example |
|----------|---------|---------|
| **silas-research** | Deep exploration, multi-source lookup | "Find out about...", "What's the best way to...", "Research X" |
| **silas-drafter** | Long-form writing, complex composition | Multi-paragraph emails, documents, sensitive communications |
| **silas-memory** | Deep memory operations | Learning cycles, pattern review, memory health audits |
| **silas-meeting-prep** | Meeting briefings, follow-ups | "Prep me for X meeting", post-meeting action items |
| **silas-comms** | Quick voice-matched messages | Slack replies, short emails, status updates |

### How to Delegate

```javascript
task({
  description: "Research X for user",
  subagent_type: "silas-research",
  prompt: "Find information about X. User needs to decide Y. Return synthesized findings with confidence level."
})
```

### Delegation Rules

1. **Background by default** - Subagents run while you continue
2. **Be specific** - Tell the subagent exactly what you need back
3. **You mediate** - Subagent output comes to you, you present to user
4. **Quick lookups stay local** - Single Jira query? Do it yourself

---

## Learning System

### Hybrid Recall

`silas_memory_recall` implements hybrid retrieval:

1. **Facts** (from Neo4j): Current assertions for entity
2. **Snippets** (from Qdrant + Postgres): Vector similarity + lexical search

```javascript
RecallResult:
  facts:       list[dict]  # From Neo4j current_facts
  snippets:    list[dict]  # Merged vector + lexical hits
  used:        dict        # Debug: entity_ids, point_ids, assertion_ids
  latency_ms:  int
```

### Background Workers

| Worker | Purpose |
|--------|---------|
| `embed_worker.py` | Generates embeddings for unprocessed events → Qdrant |
| `kg_worker.py` | Maps observations/person_updates → Neo4j facts |

### Learning Rules

1. **Infer, don't ask** - Figure it out from context when possible
2. **Corrections are gold** - Record type=correction immediately
3. **Confirm periodically** - Use `memory.learn.patterns()` in weekly reviews
4. **Self-tune when prompted** - If user says you're too X or not enough Y, adjust config

---

## Self-Tuning Config

| Path | What it controls |
|------|------------------|
| `memory.half_life_days` | How fast memories decay (default: 14) |
| `learning.min_pattern_confidence` | Threshold to surface patterns (default: 0.3) |
| `behavior.ask_clarifying_questions` | Whether to ask vs infer (default: true) |
| `behavior.proactive_suggestions` | Whether to volunteer info (default: true) |
| `behavior.auto_observe_corrections` | Auto-record corrections when detected (default: true) |

**When to self-tune**:
- If user says "you forget too fast" → increase `memory.half_life_days`
- If patterns are too noisy → increase `learning.min_pattern_confidence`
- If user wants less questioning → set `behavior.ask_clarifying_questions` to false

Always include a `reason` when setting config - it's logged for audit.

---

## People Memory (CRM)

### CRM Fields

| Field | Purpose | When to Update |
|-------|---------|----------------|
| `last_contact` | Last real interaction | After meeting/call/email exchange |
| `contact_priority` | high/medium/low | Based on relationship importance |
| `next_action` | What to do next | After conversations with follow-ups |
| `follow_up_date` | When to reach out | When user says "remind me to..." |
| `topics` | Common discussion topics | Accumulated from interactions |

### CRM Actions

```javascript
const stale = await memory.person.stale(14)
const followUps = await memory.person.followUps()
await memory.person.add({ name: "Sarah", topics: ["project X", "budget"] })
return { staleCount: stale.length, followUpsCount: followUps.length }
```

---

## Energy-Type Task Prioritization

When presenting todos, group by energy type:

| Type | Description | Task Signals |
|------|-------------|--------------|
| **Quick Win** | < 15 min, low cognitive load | reply, approve, review, update, send |
| **Creative** | Open-ended, needs focus | design, draft, write, brainstorm, plan |
| **Deep Work** | Complex, uninterrupted time | debug, implement, architect, refactor |
| **Admin** | Routine but necessary | schedule, file, organize, expense |

Consider time of day and calendar density when suggesting focus:
- Morning with focus time → suggest deep work
- Fragmented day → suggest quick wins between meetings
- End of week → good for admin backlog

---

## Commands

| Command | Agent | Purpose |
|---------|-------|---------|
| `/morning` | suggestions | Daily briefing (calendar, energy-typed todos, urgent items) |
| `/silas` | silas | Start conversation with Silas |
| `/todos` | silas | Energy-typed task list |
| `/inbox` | silas | Check and triage email |
| `/calendar` | suggestions | Today's calendar overview |
| `/remind` | silas | Set a reminder |
| `/prep "topic"` | silas | Meeting preparation (attendees, context, agenda) |
| `/meeting-notes` | silas | Process notes (extract people, actions, context) |
| `/wrap-up` | silas | End of day sync and capture |
| `/weekly-review` | weekly-review | Pattern confirmation and learning cycle |
| `/status` | silas | System health check (jobs, memory stats) |
| `/proactive-check` | suggestions | Background proactive suggestions |
| `/project` | silas | Project context management |
| `/tui` | — | Launch Silas dashboard GUI |

---

## Scheduled Automation

| Job | Schedule | Purpose |
|-----|----------|---------|
| Daily briefing | 8:30 AM weekdays | Trigger `/morning` reminder |
| Proactive check | Every 2 hours | Background suggestions |
| Nightly maintenance | 2:00 AM | Memory health, cleanup |
| Meeting prep emailer | 30 min before meetings | Auto-send prep briefs |

---

## CLI Commands

```bash
# Bring up stack (Postgres + Qdrant + Neo4j)
python3 scripts/memory_stack.py up

# Check backend health
python3 scripts/memory_stack.py doctor

# Run schema migrations
python3 scripts/memory_stack.py upgrade

# Rebuild derived indexes
python3 scripts/memory_stack.py reindex

# Import legacy SQLite memory
python3 scripts/memory_stack.py import-legacy

# Destroy volumes (destructive)
python3 scripts/memory_stack.py reset --yes
```

---

## Disabled Features (oh-my-opencode.json)

The Ops workspace intentionally disables many standard OpenCode features for a focused implementation:

**Disabled Agents**: sisyphus, oracle, librarian, explore, frontend-ui-ux-engineer, document-writer, multimodal-looker

**Disabled MCPs**: websearch_exa, context7, grep_app

**Disabled Commands**: init-deep

**Disabled Skills**: playwright

**Disabled Hooks**: 50+ hooks including todo-continuation-enforcer, context-window-monitor, session-recovery, thinking-block-validator, etc.

**Claude Code Features**: All disabled (mcp: false, commands: false, skills: false, agents: false, hooks: false, plugins: false)

This configuration creates a streamlined environment where Silas operates without interference from generic agent behaviors.

---

## File Layout

```
Ops/
├── .opencode/
│   ├── agent/                    # Agent definitions (8 agents)
│   │   ├── silas.md
│   │   ├── silas-research.md
│   │   ├── silas-drafter.md
│   │   ├── silas-memory.md
│   │   ├── silas-meeting-prep.md
│   │   ├── silas-comms.md
│   │   ├── suggestions.md
│   │   └── weekly-review.md
│   ├── persona/
│   │   └── silas.md              # Voice and identity guidelines
│   ├── tool/                     # Sandbox implementations
│   │   ├── ops-sandbox.ts        # Calendar, email, todos, reminders, notes
│   │   ├── memory-sandbox.ts     # Wraps silas_memory_* MCP tools
│   │   ├── atlassian-sandbox.ts  # Jira + Confluence
│   │   ├── granola-sandbox.ts    # Meeting transcripts
│   │   ├── askuser.ts            # Interactive prompts
│   │   └── lib/                  # Supporting libraries
│   ├── command/                  # Slash commands (13 commands)
│   │   ├── morning.md
│   │   ├── silas.md
│   │   ├── todos.md
│   │   ├── inbox.md
│   │   ├── calendar.md
│   │   ├── remind.md
│   │   ├── prep.md
│   │   ├── meeting-notes.md
│   │   ├── wrap-up.md
│   │   ├── weekly-review.md
│   │   ├── status.md
│   │   ├── proactive-check.md
│   │   └── project.md
│   ├── memory-stack/             # Memory system implementation
│   │   ├── scripts/
│   │   │   ├── mcp_memory_server.py
│   │   │   ├── memory_stack.py   # CLI orchestrator
│   │   │   ├── store/            # Postgres, Qdrant, Neo4j clients
│   │   │   ├── retrieval/        # Hybrid recall controller
│   │   │   └── workers/          # embed_worker.py, kg_worker.py
│   │   └── memory/
│   │       ├── ontology.yaml     # KG schema
│   │       ├── config.json       # Self-tuning config
│   │       └── conventions.md    # User preferences
│   ├── opencode.json             # Agent/tool configuration
│   ├── op1-policy.json           # Permission scoping
│   └── oh-my-opencode.json       # Disabled features
└── .beads/
    └── README.md
```

---

## Next Steps

- **[Agents](./agents/)** - Detailed agent documentation
- **[Memory Stack](./memory-stack/)** - Deep dive into memory architecture
- **[Workflows](./workflows/)** - Common workflow examples
- **[Examples](../../examples/ops/)** - Real usage examples
