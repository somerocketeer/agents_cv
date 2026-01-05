# Example: Playwright Verification

Example of verifying documentation against live UI.

---

## Input: Documentation

```mdx
# Creating a New Project

1. Click the **New Project** button in the top right
2. Enter a project name
3. Select a template from the dropdown
4. Click **Create**

You'll be redirected to the project dashboard.
```

---

## Execution

### Phase 1: Parse Documentation

**Agent**: playwriter-parser

```javascript
Task({
  subagent_type: "playwriter-parser",
  prompt: `
    Convert documentation to Playwright Workflow JSON:
    Input: docs/creating-project.mdx
    Output: workflow.json
  `
})
```

**Output**: `workflow.json`
```json
{
  "name": "Creating a New Project",
  "steps": [
    {
      "id": "step_1",
      "action": "click",
      "target": {
        "description": "New Project button in top right",
        "selector": null
      }
    },
    {
      "id": "step_2",
      "action": "fill",
      "target": {
        "description": "project name input",
        "selector": null
      },
      "value": "Test Project"
    },
    {
      "id": "step_3",
      "action": "click",
      "target": {
        "description": "template dropdown",
        "selector": null
      }
    },
    {
      "id": "step_4",
      "action": "click",
      "target": {
        "description": "template option",
        "selector": null
      }
    },
    {
      "id": "step_5",
      "action": "click",
      "target": {
        "description": "Create button",
        "selector": null
      }
    },
    {
      "id": "step_6",
      "action": "assert",
      "target": {
        "description": "project dashboard",
        "selector": null
      },
      "assertion": "visible"
    }
  ]
}
```

### Phase 2: Generate Selector Map

**Agent**: playwriter-map-generator

```javascript
Task({
  subagent_type: "playwriter-map-generator",
  prompt: `
    Scan: https://app.example.com/projects
    Generate selector map for workflow steps.
    Output: selector-map.json
  `
})
```

**Output**: `selector-map.json`
```json
{
  "New Project button in top right": {
    "selector": "[data-testid='new-project-btn']",
    "alternatives": [
      "button:has-text('New Project')",
      ".header-actions button.primary"
    ]
  },
  "project name input": {
    "selector": "[data-testid='project-name-input']",
    "alternatives": [
      "input[name='projectName']",
      ".project-form input:first-of-type"
    ]
  },
  "template dropdown": {
    "selector": "[data-testid='template-select']",
    "alternatives": [
      "select[name='template']",
      ".template-selector"
    ]
  },
  "Create button": {
    "selector": "[data-testid='create-project-btn']",
    "alternatives": [
      "button:has-text('Create')",
      ".project-form button[type='submit']"
    ]
  },
  "project dashboard": {
    "selector": "[data-testid='project-dashboard']",
    "alternatives": [
      ".dashboard-container",
      "main.project-view"
    ]
  }
}
```

### Phase 3: Resolve Selectors

**Agent**: playwriter-resolver-text

```javascript
Task({
  subagent_type: "playwriter-resolver-text",
  prompt: `
    Resolve stable selectors:
    Workflow: workflow.json
    Selector map: selector-map.json
    Output: workflow-resolved.json
  `
})
```

**Output**: `workflow-resolved.json`
```json
{
  "name": "Creating a New Project",
  "steps": [
    {
      "id": "step_1",
      "action": "click",
      "selector": "[data-testid='new-project-btn']"
    },
    {
      "id": "step_2",
      "action": "fill",
      "selector": "[data-testid='project-name-input']",
      "value": "Test Project"
    },
    {
      "id": "step_3",
      "action": "click",
      "selector": "[data-testid='template-select']"
    },
    {
      "id": "step_4",
      "action": "click",
      "selector": "[data-testid='template-select'] option:first-of-type"
    },
    {
      "id": "step_5",
      "action": "click",
      "selector": "[data-testid='create-project-btn']"
    },
    {
      "id": "step_6",
      "action": "assert",
      "selector": "[data-testid='project-dashboard']",
      "assertion": "visible"
    }
  ]
}
```

### Phase 4: Execute Workflow

**Orchestrator**: playwright-orchestrator

```bash
playwright-orchestrator --workflow workflow-resolved.json
```

**Run Result**:
```json
{
  "status": "FAILED",
  "steps_passed": 4,
  "steps_failed": 1,
  "failed_step": {
    "id": "step_5",
    "action": "click",
    "selector": "[data-testid='create-project-btn']",
    "error": "Element not found: [data-testid='create-project-btn']"
  },
  "screenshot": "failure-step5.png"
}
```

### Phase 5: Debug Failure

**Agent**: playwriter-debugger

```javascript
Task({
  subagent_type: "playwriter-debugger",
  prompt: `
    Analyze failure:
    Run result: run-result.json
    Screenshot: failure-step5.png

    Suggest fixes.
  `
})
```

**Output**:
```markdown
## Failure Analysis

**Step 5 Failed**: Create button not found

**Screenshot Analysis**:
The screenshot shows the form is in a modal dialog. The Create button selector
`[data-testid='create-project-btn']` is correct, but the button text has changed
from "Create" to "Create Project".

**Root Cause**:
UI was updated - button text changed from "Create" to "Create Project"

**Suggested Fix**:
1. Update documentation: "Click **Create**" → "Click **Create Project**"
2. Selector still works (data-testid unchanged)

**Alternative Selectors** (if data-testid removed):
- `button:has-text('Create Project')`
- `.modal-footer button.primary`
```

### Phase 6: Update Documentation

**Agent**: playwriter-doc-updater

```javascript
Task({
  subagent_type: "playwriter-doc-updater",
  prompt: `
    Update documentation:
    Doc: docs/creating-project.mdx
    Fix: Change "Click **Create**" to "Click **Create Project**"
    Output: docs/creating-project.mdx
  `
})
```

**Output**: Updated documentation
```mdx
# Creating a New Project

1. Click the **New Project** button in the top right
2. Enter a project name
3. Select a template from the dropdown
4. Click **Create Project**

You'll be redirected to the project dashboard.
```

### Phase 7: Re-verify

```bash
playwright-orchestrator --workflow workflow-resolved.json
```

**Result**:
```json
{
  "status": "PASSED",
  "steps_passed": 6,
  "steps_failed": 0
}
```

---

## Summary

```yaml
summary: Documentation verified and updated
verification:
  initial_status: FAILED
  final_status: PASSED
  steps_verified: 6
changes:
  - file: docs/creating-project.mdx
    change: "Updated button text from 'Create' to 'Create Project'"
```
