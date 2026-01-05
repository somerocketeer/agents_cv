# Documentation Domain

End-to-end documentation lifecycle from scratchpad to verified, published docs with Playwright testing.

---

## Overview

**Orchestrators**: 3
- `draft-orchestrator` - Coordinate blueprinting and drafting
- `editor-orchestrator` - Single-pass copyedit
- `playwright-orchestrator` - Verification via Playwright

**Subagents**: 10
- `docs-architect` - Create document blueprints
- `docs-detective` - Investigate logical gaps
- `docs-writer` - Produce drafts from blueprints
- `docs-copyedit` - Single-agent audit and edit
- `docs-recovery` - Fix validation errors
- `playwriter-parser` - Convert docs to Playwright JSON
- `playwriter-resolver-text` - Resolve stable selectors
- `playwriter-debugger` - Analyze failed runs
- `playwriter-doc-updater` - Update docs to match verified workflows
- `playwriter-map-generator` - Generate selector maps

**Standalone Agents**: 6
- `atlassian` - Jira/Confluence research and editing
- `copy-editor` - Standalone polish
- `detective` - Standalone gap finder
- `docflow` - Writer-first Jira workspaces
- `reference-generator` - Generate reference tables
- `release-notes` - Extract structured release notes

---

## Architecture

### Documentation Pipeline

```
Scratchpad → EXTRACT → ANALYZE → DRAFT → INJECT → VALIDATE → PUBLISH
                ↓          ↓         ↓        ↓         ↓
           code blocks  detective  writer  inject   recovery
                       architect
```

### Phase Breakdown

| Phase | Agent(s) | Purpose |
|-------|----------|---------|
| **INIT** | Orchestrator | Resolve paths, create manifest |
| **EXTRACT** | Script | Extract code blocks from scratchpad |
| **ANALYZE** | docs-detective, docs-architect | Gap analysis + blueprint creation |
| **DRAFT** | docs-writer | Produce draft.mdx with placeholders |
| **INJECT** | Script | Replace placeholders with code |
| **VALIDATE** | Script | Run unified validation |
| **RECOVER** | docs-recovery | Fix recoverable errors |
| **REPORT** | Orchestrator | Archive input, generate report |

---

## Orchestrators

### draft-orchestrator

**Purpose**: Coordinate documentation blueprinting and drafting from scratchpad to draft.mdx

**Model**: Claude Sonnet 4.5 (temp 0.05)

**Pipeline**:
```bash
draft-orchestrator \
  --input scratchpad.md \
  --output draft.mdx
```

**Phases**:
1. Resolve workspace paths
2. Initialize manifest
3. Extract code blocks
4. Run docs-detective (non-blocking)
5. Run docs-architect (blocking)
6. Run docs-writer
7. Inject code blocks
8. Validate draft
9. Attempt recovery if needed
10. Archive input on success

**Output**: `draft.mdx` with injected code blocks

---

### editor-orchestrator

**Purpose**: Single-pass copyedit for existing documentation

**Model**: Claude Sonnet 4.5 (temp 0.05)

**Pipeline**:
```bash
editor-orchestrator \
  --target docs/guide.mdx
```

**Fixes**:
- Passive voice
- Long sentences (>25 words)
- Filler words and hedge stacking
- Code context missing
- Inconsistent terminology

---

### playwright-orchestrator

**Purpose**: Verify documentation against live UI via Playwright

**Model**: Claude Sonnet 4.5 (temp 0.05)

**Pipeline**:
```bash
playwright-orchestrator \
  --workflow workflow.json
```

**Phases**:
1. Parse documentation → Playwright Workflow JSON
2. Resolve selectors from DOM
3. Execute workflow
4. Analyze failures
5. Update documentation to match verified state

---

## Subagents

### docs-architect

**Purpose**: Analyze content and create document blueprints

**Model**: Claude Sonnet 4.5 (temp 0.0)

**Input**: Scratchpad, code manifest, questions

**Output**: Blueprint with:
- Document type classification
- Section structure
- Code block placement
- Required sections checklist

**Example**:
```javascript
Task({
  subagent_type: "docs-architect",
  prompt: `
    Create blueprint for: ${scratchpadPath}
    Code manifest: ${codeManifestPath}
    Questions: ${questionsPath}

    Output: ${blueprintPath}
  `
})
```

---

### docs-detective

**Purpose**: Investigate source material for logical gaps

**Model**: Claude Sonnet 4.5 (temp 0.15)

**Input**: Scratchpad content

**Output**: Questions file with:
- Missing context
- Ambiguous references
- Incomplete explanations
- Broken logical chains

**Example**:
```javascript
Task({
  subagent_type: "docs-detective",
  prompt: `
    Analyze: ${scratchpadPath}

    Find:
    - Missing prerequisites
    - Undefined terms
    - Incomplete code examples
    - Logical gaps

    Output: ${questionsPath}
  `
})
```

---

### docs-writer

**Purpose**: Produce draft.mdx from blueprint and scratchpad

**Model**: Claude Sonnet 4.5 (temp 0.10)

**Input**: Blueprint, scratchpad, code manifest

**Output**: draft.mdx with:
- Code placeholders (`<!-- CODE:block_id -->`)
- INPUT markers for gaps (`<!-- INPUT: question -->`)
- Proper MDX structure

**Rules**:
- Follow blueprint structure exactly
- Preserve literal quotes from scratchpad
- Use placeholders for code (don't copy)
- Mark gaps with INPUT comments

---

### docs-copyedit

**Purpose**: Single-agent audit and edit pass

**Model**: Claude Sonnet 4.5 (temp 0.05)

**Fixes**:
- Voice (active preferred)
- Sentence length
- Terminology consistency
- Style guide compliance

---

### docs-recovery

**Purpose**: Fix recoverable validation errors

**Model**: Claude Sonnet 4.5 (temp 0.0)

**Input**: Draft with validation errors

**Output**: Fixed draft

**Recoverable Errors**:
- Missing frontmatter fields
- Invalid placeholder format
- Broken internal links
- Missing required sections

---

### Playwright Subagents

| Agent | Purpose | Temp |
|-------|---------|------|
| `playwriter-parser` | Convert docs → Playwright Workflow JSON | 0.05 |
| `playwriter-resolver-text` | Resolve stable selectors from DOM | 0.05 |
| `playwriter-debugger` | Analyze failed runs, suggest fixes | 0.10 |
| `playwriter-doc-updater` | Update docs to match verified workflows | 0.10 |
| `playwriter-map-generator` | Scan websites, generate selector maps | 0.05 |

---

## Standalone Agents

### atlassian

**Purpose**: Jira and Confluence research, reporting, and editing

**Model**: Claude Sonnet 4.5 (temp 0.15)

**Capabilities**:
- Search Jira issues
- Read/update Confluence pages
- Generate reports from tickets
- Link documentation to tickets

---

### copy-editor

**Purpose**: Single-pass polish for existing documentation (no orchestrator)

**Model**: Claude Sonnet 4.5 (temp 0.05)

**Use When**: Quick polish needed without full pipeline

---

### detective

**Purpose**: Standalone gap finder (no orchestrator)

**Model**: Claude Sonnet 4.5 (temp 0.15)

**Use When**: Analyze existing docs for gaps without drafting

---

### docflow

**Purpose**: Manage writer-first Jira ticket workspaces

**Model**: Claude Sonnet 4.5 (temp 0.10)

**Features**:
- Create doc-focused Jira views
- Track documentation status
- Generate documentation tickets from code changes

---

### reference-generator

**Purpose**: Generate reference documentation tables

**Model**: Claude Sonnet 4.5 (temp 0.10)

**Output**: Structured tables for:
- API endpoints
- Configuration options
- CLI commands
- Environment variables

---

### release-notes

**Purpose**: Extract structured release notes from scratchpad

**Model**: Claude Sonnet 4.5 (temp 0.10)

**Output**: Formatted release notes with:
- Features
- Bug fixes
- Breaking changes
- Migration notes

---

## Validation System

### Validation Script

```bash
python3 validate_draft.py \
  --draft draft.mdx \
  --scratchpad scratchpad.md \
  --project-dir ./
```

### Validation Checks

| Check | Type | Description |
|-------|------|-------------|
| Frontmatter | Blocking | Required fields present |
| Placeholders | Blocking | Valid format, all resolved |
| Sections | Blocking | Required sections present |
| Links | Warning | Internal links valid |
| Code | Warning | All code blocks have language |
| Style | Warning | Follows style guide |

### Error Taxonomy

| Error | Recoverable | Action |
|-------|-------------|--------|
| `MISSING_FRONTMATTER` | Yes | Add default frontmatter |
| `INVALID_PLACEHOLDER` | Yes | Fix placeholder format |
| `MISSING_SECTION` | No | Report, stop |
| `BROKEN_LINK` | Yes | Remove or fix link |

---

## File Layout

```
~/Code/devcontainer/
├── .opencode/
│   ├── agent/
│   │   ├── draft-orchestrator.md
│   │   ├── editor-orchestrator.md
│   │   ├── playwright-orchestrator.md
│   │   ├── docs-architect.md
│   │   ├── docs-detective.md
│   │   ├── docs-writer.md
│   │   ├── docs-copyedit.md
│   │   ├── docs-recovery.md
│   │   ├── playwriter-parser.md
│   │   ├── playwriter-resolver-text.md
│   │   ├── playwriter-debugger.md
│   │   ├── playwriter-doc-updater.md
│   │   ├── playwriter-map-generator.md
│   │   ├── atlassian.md
│   │   ├── copy-editor.md
│   │   ├── detective.md
│   │   ├── docflow.md
│   │   ├── reference-generator.md
│   │   └── release-notes.md
│   └── scripts/
│       ├── orchestrator_manifest.py
│       ├── extract_code_blocks.py
│       ├── inject_code_blocks.py
│       └── validate_draft.py
└── docs/
    └── (output documentation)
```

---

## Quick Start

### Draft from Scratchpad

```bash
# Create scratchpad
echo "# My Feature\n\nRaw notes here..." > scratchpad.md

# Run draft pipeline
draft-orchestrator --input scratchpad.md --output draft.mdx
```

### Copyedit Existing Docs

```bash
editor-orchestrator --target docs/guide.mdx
```

### Verify with Playwright

```bash
# Convert docs to workflow
playwriter-parser --input docs/guide.mdx --output workflow.json

# Run verification
playwright-orchestrator --workflow workflow.json
```

---

## Related

- **[Patterns](../../patterns/orchestration.md)** - Phase-based orchestration
- **[Examples](../../examples/documentation/)** - Real usage examples
- **[Agent Index](../../reference/agent-index.md)** - All agents A-Z
