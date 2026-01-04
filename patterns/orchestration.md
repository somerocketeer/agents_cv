# Phase-Based Orchestration Pattern

All orchestrators across domains follow a consistent phase-based execution model with fail-fast behavior.

---

## Core Pattern

```
Setup → Evidence → Planning → Execution → Validation → Publishing
```

### Characteristics

1. **Sequential Execution** - Phases run in order, no skipping
2. **Fail-Fast** - Blocking errors stop immediately
3. **Validation Gates** - Each phase validates before proceeding
4. **Recovery Attempts** - Retry once on recoverable errors
5. **Final Report** - Always output results, even on failure

---

## Phase Breakdown

### Phase 1: Setup

**Purpose**: Validate inputs, resolve paths, create workspace

**Common Operations**:
- Resolve `${HOME}` / `${PWD}` placeholders
- Verify required files exist
- Create working directories
- Initialize manifest/state

**Failure Behavior**: STOP immediately, report missing files

**Example** (Documentation):
```bash
# Resolve workspace
repo_root=$(pwd)
while [ ! -d "${repo_root}/.opencode" ] && [ "${repo_root}" != "/" ]; do
  repo_root=$(dirname "${repo_root}")
done

# Initialize manifest
python3 "${repo_root}/.opencode/scripts/orchestrator_manifest.py" init \
  --input-path "${input_path}" \
  --output-path "${output_path}"
```

**Example** (Marketing):
```javascript
// Verify workspace
const workspace = `${HOME}/Marketing/_staging`
const projectDir = `${workspace}/${project}`

// Verify required files
const required = [
  `${projectDir}/scratchpad.md`,
  `${styleGuidePath}`,
  `${personaPath}`
]

for (const file of required) {
  if (!exists(file)) {
    return { error: "REQUIRED_FILE_NOT_FOUND", file }
  }
}
```

---

### Phase 2: Evidence Foundation

**Purpose**: Extract facts, build context, gather intelligence

**Common Operations**:
- Extract structured facts from source material
- Map facts to target personas
- Gather market intelligence (if applicable)
- Build code block manifests (docs)

**Failure Behavior**: Retry once, then STOP

**Parallel Execution**: Evidence-gathering tasks can run in parallel

**Example** (Marketing):
```javascript
// Parallel evidence gathering
Task({ subagent: "fact_bank_builder", ... })
Task({ subagent: "persona_calibrator", ... })
Task({ subagent: "marketing_research_scout", ... })

// Wait for all to complete
await Promise.all([factBank, personaBrief, marketIntel])
```

**Example** (Documentation):
```bash
# Extract code blocks
python3 extract_code_blocks.py --path "${input_path}"

# Run detective (non-blocking)
task({ subagent: "docs-detective", ... })

# Run architect (blocking)
task({ subagent: "docs-architect", ... })
```

---

### Phase 3: Planning

**Purpose**: Create structured outline or blueprint

**Common Operations**:
- Generate argument-driven outlines
- Create document blueprints
- Define section structure
- Identify code block placement

**Failure Behavior**: Retry once with explicit instruction, then STOP

**Validation Gates**:
- Required sections present
- Thesis defined (marketing)
- Counterarguments present (marketing)
- Code block placement (docs)

**Example** (Marketing):
```javascript
// Invoke planner
Task({
  subagent: "blog_planner",
  inputs: {
    fact_bank_path,
    persona_brief_path,
    market_research_path,
    style_path
  }
})

// Validate plan
if (!plan.thesis) {
  // Retry with explicit instruction
  Task({
    subagent: "blog_planner",
    prompt: "Create outline with EXPLICIT thesis statement..."
  })
}
```

**Example** (Documentation):
```javascript
// Invoke architect
Task({
  subagent: "docs-architect",
  inputs: {
    input_path,
    draft_dir,
    questions_path,
    code_manifest_path
  }
})

// Store docType for later
const docType = blueprint.docType
```

---

### Phase 4: Execution (Drafting)

**Purpose**: Produce first draft following plan/blueprint

**Common Operations**:
- Write prose following outline
- Insert code placeholders (docs)
- Add INPUT markers for gaps (docs)
- Synthesize citations (marketing)

**Failure Behavior**: STOP if draft not created

**Validation Gates**:
- Draft file exists
- Required sections present
- Placeholders/citations valid

**Example** (Marketing):
```javascript
// Invoke drafter
Task({
  subagent: "blog_drafter",
  inputs: {
    fact_bank_path,
    persona_brief_path,
    plan_path,
    style_path,
    market_research_path
  }
})

// Verify draft exists
if (!exists(draft_path)) {
  return { error: "DRAFT_NOT_CREATED" }
}
```

**Example** (Documentation):
```javascript
// Invoke writer with inline code manifest
Task({
  subagent: "docs-writer",
  inputs: {
    output_path,
    blueprint_path,
    input_path,
    questions_path,
    code_manifest_content: JSON.stringify(codeManifest)
  }
})

// Verify draft
if (!exists(output_path)) {
  return { error: "DRAFT_NOT_CREATED" }
}
```

---

### Phase 5: Validation

**Purpose**: Check quality, compliance, completeness

**Common Operations**:
- Run linters
- Validate citations
- Check style compliance
- Verify placeholders (docs)
- Check frontmatter (docs)

**Failure Behavior**: Attempt recovery, then report

**Validation Types**:
- **Blocking**: Must pass to proceed
- **Warnings**: Log and continue
- **Recoverable**: Attempt fix

**Example** (Marketing):
```javascript
// Run linter
const lintResult = marketing_lint({
  project_dir: projectDir,
  content_type: "blog"
})

// Validate citations
const citationResult = marketing_check_citations({
  project: project
})

// Check for blocking errors
if (lintResult.status === "FAIL") {
  return { error: "VALIDATION_FAILED", details: lintResult }
}
```

**Example** (Documentation):
```bash
# Run unified validation
python3 validate_draft.py \
  --draft "${output_path}" \
  --scratchpad "${input_path}" \
  --project-dir "${draft_dir}"

# Parse results
validation_status=$(jq -r '.status' validation.json)
blocking_errors=$(jq -r '.blocking_errors' validation.json)

# Check status
if [ "$validation_status" == "FAIL" ]; then
  # Attempt recovery
  task({ subagent: "docs-recovery", ... })
fi
```

---

### Phase 6: Publishing

**Purpose**: Transform to final publication format

**Common Operations**:
- Convert internal format → reader format
- Transform fact IDs → numbered references
- Add frontmatter (docs)
- Clean internal sections
- Final formatting

**Failure Behavior**: Report errors, do not retry

**Validation Gates**:
- No internal markers remaining
- No persona names in final copy
- Clean formatting
- Valid frontmatter (docs)

**Example** (Marketing):
```javascript
// Invoke publisher
Task({
  subagent: "blog_publisher",
  inputs: {
    edited_path,
    fact_bank_path,
    style_path
  }
})

// Final checks
const finalChecks = [
  !contains(final, "[fact_"),
  !contains(final, "Architect Alice"),
  contains(final, "## References"),
  !contains(final, "## Editorial Notes")
]

if (!finalChecks.every(Boolean)) {
  return { error: "PUBLISHING_FAILED" }
}
```

**Example** (Documentation):
```bash
# Inject code blocks
python3 inject_code_blocks.py \
  --draft "${output_path}" \
  --manifest "${code_blocks_path}"

# Archive input on success
if [ "$validation_status" == "PASS" ]; then
  mv "${input_path}" "${draft_dir}/$(basename ${input_path}).archived"
fi
```

---

## Delegation Patterns

### When to Delegate

| Scenario | Action |
|----------|--------|
| **Complex multi-step task** | Delegate to subagent |
| **Specialized expertise needed** | Delegate to subagent |
| **Simple tool call** | Execute locally |
| **Immediate response needed** | Execute locally |

### How to Delegate

```javascript
// Standard delegation
Task({
  description: "Brief action description",
  subagent_type: "subagent_name",
  prompt: `
    1. TASK: Specific goal
    2. EXPECTED OUTCOME: Deliverables
    3. REQUIRED SKILLS: If any
    4. REQUIRED TOOLS: read, write
    5. MUST DO:
       - Use absolute paths
       - Write output to specified path
       - Preserve literal quotes
    6. MUST NOT DO:
       - Invent facts
       - Skip validation
    7. CONTEXT:
       - project: ${project}
       - paths: ${paths}
  `
})
```

### Delegation Rules

1. **Background by default** - Subagents run while orchestrator continues
2. **Be specific** - Tell subagent exactly what to return
3. **Orchestrator mediates** - Subagent output goes to orchestrator, not user
4. **Quick lookups stay local** - Don't delegate simple operations

---

## Error Handling

### Error Taxonomy

| Code | Source | Meaning | Action |
|------|--------|---------|--------|
| `WORKSPACE_NOT_FOUND` | Orchestrator | Staging directory missing | STOP |
| `PROJECT_NOT_FOUND` | Orchestrator | Project directory missing | STOP |
| `REQUIRED_FILE_NOT_FOUND` | Orchestrator | Input file missing | STOP |
| `FACT_BANK_INVALID` | Subagent | Empty or malformed fact bank | Retry once, then STOP |
| `PLAN_INCOMPLETE` | Subagent | Missing thesis or sections | Retry once, then STOP |
| `DRAFT_NOT_CREATED` | Subagent | Draft file not written | STOP |
| `VALIDATION_FAILED` | Orchestrator | Blocking validation errors | Attempt recovery, then STOP |
| `PUBLISHING_FAILED` | Subagent | Transformation errors | STOP |

### Retry Strategy

```javascript
// Retry pattern
let attempts = 0
const maxAttempts = 2

while (attempts < maxAttempts) {
  const result = Task({ subagent: "...", ... })
  
  if (result.status === "success") {
    break
  }
  
  attempts++
  
  if (attempts >= maxAttempts) {
    return { error: "MAX_RETRIES_EXCEEDED", stage: "..." }
  }
  
  // Truncate output file before retry
  truncate(output_path)
}
```

### Recovery Pattern

```javascript
// Recovery attempt
if (validation.status === "FAIL" && validation.recoverable) {
  Task({
    subagent: "docs-recovery",
    inputs: {
      output_path,
      validation_errors: validation.blocking_errors
    }
  })
  
  // Re-validate
  const revalidation = validate(output_path)
  
  if (revalidation.status === "PASS") {
    // Continue to publishing
  } else {
    // Report partial recovery
    return { status: "PARTIAL_RECOVERY", unfixed: revalidation.errors }
  }
}
```

---

## Anti-Loop Rules

1. **Never re-invoke a subagent with identical inputs** after failure
2. **Never re-read files already produced** by subagents
3. **Stop and report** if you hit 8 Task calls without progress
4. **Truncate output files** before retry to avoid appending

---

## Response Format

All orchestrators return structured results:

```yaml
summary: <1-2 sentence outcome>
artifacts:
  fact_bank: <absolute path>
  plan: <absolute path>
  draft: <absolute path>
  edited: <absolute path>
  final: <absolute path>
quality:
  thesis_delivery: <STRONG | ADEQUATE | WEAK>
  synthesis_coverage: <X of Y citations>
error:
  code: <error code or `none`>
  stage: <stage name or `none`>
  message: "<description or `none`>"
validation:
  status: <PASSED | WARNINGS | FAILED>
  errors: []
  warnings: []
next_actions:
  - <suggested follow-up if incomplete>
```

---

## Examples by Domain

### Ops (Silas)

```
Load Memory → Orient → Execute Task → Observe → Update Context
```

**Phases**:
1. **Load**: `memory.load()` - Get session snapshot
2. **Orient**: Understand context silently
3. **Execute**: Handle task or delegate to subagent
4. **Observe**: Record high-signal observations
5. **Update**: Update context if changed

### Documentation

```
INIT → EXTRACT → ANALYZE → DRAFT → INJECT → VALIDATE → RECOVER → REPORT
```

**Phases**:
1. **INIT**: Resolve paths, create manifest
2. **EXTRACT**: Extract code blocks
3. **ANALYZE**: Run detective + architect
4. **DRAFT**: Produce draft with placeholders
5. **INJECT**: Replace placeholders with code
6. **VALIDATE**: Run unified validation
7. **RECOVER**: Fix recoverable errors (optional)
8. **REPORT**: Archive input, generate report

### Marketing

```
Setup → Evidence → Research → Plan → Draft → Edit → Publish → Validate
```

**Phases**:
1. **Setup**: Verify workspace, resolve paths
2. **Evidence**: fact_bank_builder + persona_calibrator
3. **Research**: marketing_research_scout (optional)
4. **Plan**: *_planner (argument-driven outline)
5. **Draft**: *_drafter (synthesis-first prose)
6. **Edit**: *_editor (polish + validate)
7. **Publish**: *_publisher (transform to publication format)
8. **Validate**: Run linters, check citations

### Blog & Design

```
Interview → Plan → Draft → Edit → Publish
```

**Phases**:
1. **Interview**: Gather context via questions
2. **Plan**: Create destination-first outline + headline variants
3. **Draft**: Write in teaching voice
4. **Edit**: 3-pass (voice → meat → anti-LLM)
5. **Publish**: Internal linking + MDX formatting

---

## Related

- **[Model Routing](./model-routing.md)** - Model selection patterns
- **[Evidence Discipline](./evidence-discipline.md)** - Fact tracking patterns
- **[Delegation](./delegation.md)** - Subagent patterns
- **[Examples](../examples/)** - Real usage examples
