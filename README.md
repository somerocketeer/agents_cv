#Agent CV

## Memory Stack — Next-Gen Memory System

**Purpose**: Scalable, distributed memory system replacing the SQLite-backed implementation. Designed for 1M+ items with temporal knowledge graph, hybrid search, and automatic consolidation.

---

## Architecture

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

---

## Data Stores

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

```yaml
User:       singleton_id: "user:primary"
Person:     id_prefix: "person:"
Project:    id_prefix: "project:"
Thread:     id_prefix: "thread:"
Dependency: id_prefix: "dep:"
```

### Predicates

| Predicate | Subject | Cardinality | Policy | Type |
|-----------|---------|-------------|--------|------|
| `user.timezone` | User | single | supersede | string |
| `user.work_hours` | User | single | supersede | json |
| `pref.communication.length` | User | single | supersede | string |
| `pref.communication.format` | User | multi | set_union | string |
| `pref.communication.tone` | User | single | supersede | string |
| `pref.communication.channel` | User | multi | set_union | string |
| `pref.time.focus_time` | User | multi | set_union | json |
| `pref.time.meetings` | User | multi | set_union | json |
| `pref.writing.avoid_em_dashes` | User | single | supersede | boolean |
| `person.relationship_to_user` | Person | single | supersede | string |
| `person.communication_style` | Person | single | supersede | string |
| `person.contact_priority` | Person | single | supersede | string |
| `person.next_action` | Person | single | supersede | string |
| `person.topics` | Person | multi | set_union | string |

### Cardinality Policies

- `single` + `supersede`: New value replaces old (closes previous assertion)
- `multi` + `set_union`: Values accumulate (multiple assertions valid)

---

## Hybrid Recall System

`silas_memory_recall` implements hybrid retrieval:

1. **Facts** (from Neo4j): Current assertions for entity
2. **Snippets** (from Qdrant + Postgres): Vector similarity + lexical search

```python
RecallResult:
  facts:       list[dict]  # From Neo4j current_facts
  snippets:    list[dict]  # Merged vector + lexical hits
  used:        dict        # Debug: entity_ids, point_ids, assertion_ids
  latency_ms:  int
```

---

## Background Workers

| Worker | Purpose |
|--------|---------|
| `embed_worker.py` | Generates embeddings for unprocessed events → Qdrant |
| `kg_worker.py` | Maps observations/person_updates → Neo4j facts |

### KG Worker Logic

- `observation` events → User preference facts (e.g., `pref.communication.*`)
- `person_update` events → Person entities + relationship facts

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

## File Layout

```
memory-stack/
├── README.md
├── ARCHITECTURE.md          # This file
├── requirements.txt
├── memory/
│   ├── stack/               # docker-compose.yml + manifest.json
│   ├── ontology.yaml        # KG predicate definitions
│   ├── config.json          # Self-tuning config
│   └── conventions.md       # User preferences (loaded at session start)
├── scripts/
│   ├── mcp_memory_server.py # MCP tools implementation
│   ├── memory_stack.py      # CLI orchestrator
│   ├── stack_env.py         # Environment config loader
│   ├── ontology.py          # Ontology parser
│   ├── config_store.py      # Config get/set
│   ├── embedding_client.py  # Embeddings API client
│   ├── import_legacy_sqlite.py
│   ├── store/
│   │   ├── postgres.py      # Event log, thread state, audits
│   │   ├── qdrant.py        # Vector store client
│   │   └── neo4j.py         # Knowledge graph client
│   ├── retrieval/
│   │   └── controller.py    # Hybrid recall implementation
│   ├── workers/
│   │   ├── embed_worker.py  # Background embedding
│   │   └── kg_worker.py     # Background KG extraction
│   ├── migrations/
│   │   └── neo4j/           # Cypher schema migrations
│   └── bench/
│       ├── seed_memory.py   # Test data seeding
│       └── smoke_test.py    # Health check
└── agent/                   # (placeholder)
```

---

## Features Status

| Feature | Status |
|---------|--------|
| `vector_search` | Enabled (Qdrant + embeddings) |
| `lexical_search` | Enabled (Postgres) |
| `knowledge_graph` | Enabled (Neo4j) |
| `hybrid_recall` | Enabled (all backends) |
| `rerank` | Not implemented |
| `privacy_redaction` | Not implemented |

---

## 2. Documentation Pipeline

**Purpose**: End-to-end documentation lifecycle management.

### Orchestrators

| Orchestrator | Pipeline |
|--------------|----------|
| **draft-orchestrator** | Scratchpad → EXTRACT → ANALYZE → DRAFT → INJECT → VALIDATE |
| **editor-orchestrator** | Single-pass copyedit with remediation |
| **playwright-orchestrator** | Docs → executable tests → auto-update docs |

### Documentation Subagents

| Agent | Purpose |
|-------|---------|
| **docs-detective** | Gap analyzer (finds contradictions, missing prereqs, dead ends) |
| **docs-architect** | Blueprint creator (docType, section plan, code block placement) |
| **docs-writer** | Draft producer with `{/* CODE: CB### */}` placeholders |
| **docs-copyedit** | Style audit and anti-LLM cleanup |
| **docs-recovery** | Fix recoverable validation errors |
| **docflow** | JJ + Jira + GitLab automation for ticket workflows |

### Playwright Subagents

| Agent | Purpose |
|-------|---------|
| **playwriter-parser** | Convert docs to Workflow JSON |
| **playwriter-resolver-text** | Find stable selectors via DOM querying |
| **playwriter-debugger** | Analyze failures, suggest fixes |
| **playwriter-doc-updater** | Sync docs with verified workflow |
| **playwriter-map-generator** | Generate selector_map.json |

### Standalone Agents

| Agent | Purpose |
|-------|---------|
| **copy-editor** | Single-pass polish for existing docs |
| **detective** | Standalone gap finder |
| **reference-generator** | Generate reference tables from UI scanning |
| **release-notes** | Extract structured release notes from scratchpad |
| **atlassian** | Jira + Confluence research and editing |

### DocFlow Tools

| Tool | Purpose |
|------|---------|
| `docflow_new` | Create new ticket |
| `docflow_start` | Start workspace |
| `docflow_sync` | Sync/rebase |
| `docflow_publish` | Create MR |
| `docflow_finish` | Complete ticket |
| `docflow_close` | Close ticket |
| `docflow_status` | Show status |
| `docflow_pause/resume` | Time tracking |

---

## 3. Marketing Content Production

**Purpose**: Evidence-backed B2B thought leadership at scale.

### Content Pipelines (7 Orchestrators)

| Orchestrator | Output | Philosophy |
|--------------|--------|------------|
| **blog-orchestrator** | 800-1,400 word posts | Thought leadership, not documentation |
| **whitepaper-orchestrator** | 1,600-7,500 words | Authoritative, thesis-driven |
| **newsletter-orchestrator** | Insight-led updates | "Feature announcements → architectural insights" |
| **linkedin-orchestrator** | 1-4 post campaigns | "Vendor noise → peer signal" |
| **datasheet-orchestrator** | Procurement-ready specs | Technical accuracy with Architect Alice persona |
| **solution-brief-orchestrator** | Executive summaries | Business-outcome focused |
| **customer-success-orchestrator** | Rollout playbooks | Operations-focused |

### Video Pipeline

**video-integration-orchestrator** — 7-stage pipeline:
1. Setup & Validation
2. Outline Gate
3. Planning & Asset Authoring
4. TTS Synthesis (ElevenLabs)
5. Rendering (FFmpeg: 16:9, 1:1, 9:16)
6. Quality Assurance
7. Response Assembly

### Universal Subagents

| Agent | Model | Purpose |
|-------|-------|---------|
| **fact_bank_builder** | Gemini Flash | Extract citable facts with unique IDs |
| **persona_calibrator** | Claude Sonnet | Map facts to personas |
| **marketing_research_scout** | Claude Sonnet | Tiered web research (T1-T4 confidence) |

### Pipeline-Specific Subagents

Each pipeline has:
- `*_planner` — Argument-driven outlines (GPT-5.2)
- `*_drafter` — First-draft prose (Claude Sonnet)
- `*_editor` — Style audit + thesis validation (Claude Sonnet)
- `*_publisher/*_finalizer` — Transform to publication format (Gemini Flash)

### Personas

| Persona | Role |
|---------|------|
| **Architect Alice** | Technical decision-makers |
| **Technical Teddy** | Engineering leads |
| **Operations Owen** | Operations teams |

### Quality Enforcement Skills

| Skill | Purpose |
|-------|---------|
| **evidence** | No fabrication, fact IDs sacred, synthesis after every citation |
| **anti-llm** | 50+ banned phrases, frequency limits, sentence variance |
| **model-routing** | Authoritative model assignments across orchestrators |
| **shared-subagents** | Universal agent invocation patterns |

### Anti-LLM Discipline

**Hard blocks** (never use):
- Transition padding: "Moreover", "Furthermore", "Additionally"
- Hedge stacking: "It's important to note that"
- Filler verbs: "serves to", "helps to", "works to"
- Empty connectives: "In terms of", "When it comes to"

**Frequency limits**:
- Em dashes: 1-2 max per article
- Rhetorical questions: 1-2 max per article
- "leverage", "robust", "seamless": 2-3 uses max

---

## 4. Blog + Design System

**Purpose**: Personal blog production and neobrutalist design maintenance.

### Blog Pipeline (5 Stages)

| Stage | Agent | Model | Purpose |
|-------|-------|-------|---------|
| 1 | **blog_interviewer** | Claude Sonnet 4.5 | Context gathering with "meat" validation |
| 2 | **blog_planner** | GPT-5.2 | Destination-first outlines, 5 headline variants |
| 3 | **blog_drafter** | Claude Sonnet 4.5 | Teaching voice, analogy-first, 1,000-2,000 words |
| 4 | **blog_editor** | Claude Opus 4.5 | 3-pass: voice → meat → anti-LLM audit |
| 5 | **blog_publisher** | GPT-5.1 | Internal linking + MDX formatting |

### Support Agent

| Agent | Model | Purpose |
|-------|-------|---------|
| **mdx_component_builder** | Gemini Flash 3 | Interactive React components for tough concepts |

### Design System Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| **design_auditor** | Claude Sonnet 4.5 | Systematic UI/UX analysis |
| **design_implementer** | Claude Sonnet 4.5 | Focused code changes (no scope creep) |
| **design_reviewer** | Claude Sonnet 4.5 | Verification against design tokens |

### Design Orchestrator

**blog_designer** (Claude Opus 4.5) — Coordinates:
- Research → Audit → Decision → Implementation → Verification

### Design System Constraints

| Property | Value |
|----------|-------|
| Shadows | 4px offset, solid #3d405b |
| Borders | 2-4px, #3d405b |
| Radius | 5px or 0 only |
| Font weights | 900 (headings), 600 (body) |
| Focus states | outline-2 outline-[#3d405b] outline-offset-2 |
| Touch targets | minimum 44x44px |

### Voice Profile (Blog)

- Conversational, teaching-oriented
- Analogy-first approach ("Think of it like...")
- Over-explains rather than under-explains
- Task-oriented, no gatekeeping

---

## Cross-System Patterns

| Pattern | Where Used |
|---------|------------|
| **Phase-based orchestration** | All orchestrators (draft, editor, playwright, blog, marketing) |
| **Temperature control** | 0.0-0.05 deterministic, 0.25-0.30 creative |
| **Evidence discipline** | Marketing (fact IDs), Docs (code placeholders), Silas (observations) |
| **Anti-AI detection** | Marketing + context-systems (banned phrases, frequency limits) |
| **Memory/learning** | Silas (observations → patterns), Marketing (persona calibration) |
| **Delegation** | Silas → 5 subagents, Orchestrators → workers |
| **Validation gates** | All pipelines have explicit validation + recovery phases |
| **Model routing** | GPT for planning, Claude for prose, Gemini for deterministic |
| **File budgets** | Subagents limited to specific file read/write counts |
| **Retry limits** | Max retries per stage (typically 1-2) |
| **Anti-loop rules** | Never re-invoke with identical inputs |

---

## Summary Stats

| Domain | Location | Agents | Primary Use |
|--------|----------|--------|-------------|
| Ops (Silas) | `~/Ops` | 8 | Executive assistance with persistent memory |
| Devcontainer | `~/Code/devcontainer` | 19 | Documentation lifecycle + Playwright verification |
| Marketing | `~/Marketing` | 50+ | B2B content production |
| Context-systems | `~/Code/context-systems` | 12 | Blog + design system |
| **Total** | | **~90** | |

---

## Model Distribution

| Model | Use Cases |
|-------|-----------|
| **Claude Opus 4.5** | Silas primary, blog editing, design orchestration |
| **Claude Sonnet 4.5** | Most subagents, research, drafting, analysis |
| **GPT-5.2** | Planning, argument-driven outlines |
| **GPT-5.1** | Publishing, deterministic transforms |
| **Gemini Flash 3** | Deterministic operations, fact extraction, video rendering |
| **Gemini 3 Pro** | Long-form drafting |

---

## Temperature Guidelines

| Range | Use |
|-------|-----|
| 0.0 | Deterministic (architect, recovery) |
| 0.05 | Precise, structural (copyedit, publisher) |
| 0.1-0.15 | Careful analysis (research, planner) |
| 0.2-0.3 | Balanced (editor, meeting-prep) |
| 0.4-0.5 | Creative (drafter, comms) |

