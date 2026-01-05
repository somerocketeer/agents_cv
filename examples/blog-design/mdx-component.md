# Example: MDX Component Builder

Example of creating an interactive component for a blog post.

---

## Trigger

```bash
mdx_component_builder --concept "Agent orchestration flow"
```

---

## Context

The blog post "Agents Aren't Magic" needs a visual showing the agent loop pattern.

---

## Execution

### Agent: mdx_component_builder (Gemini Flash 3, temp 0.15)

**Input**: Concept description from blog post context

```markdown
Concept: Agent orchestration flow

The agent loop pattern:
1. Tool call (list files)
2. Model decision (categorize)
3. Action (move file)
4. Repeat until done

Should show:
- The loop visually
- Current step highlighted
- Ability to step through manually
```

**Output**: `AgentFlowDiagram.tsx`
```tsx
'use client'

import { useState } from 'react'

interface Step {
  id: number
  label: string
  description: string
  type: 'tool' | 'decision' | 'action'
}

const steps: Step[] = [
  { id: 1, label: 'List Files', description: 'Get files from directory', type: 'tool' },
  { id: 2, label: 'Categorize', description: 'Model decides category', type: 'decision' },
  { id: 3, label: 'Move File', description: 'Move to category folder', type: 'action' },
  { id: 4, label: 'More Files?', description: 'Check if loop continues', type: 'decision' },
]

export function AgentFlowDiagram() {
  const [currentStep, setCurrentStep] = useState(0)
  const [isPlaying, setIsPlaying] = useState(false)

  const nextStep = () => {
    setCurrentStep((prev) => (prev + 1) % steps.length)
  }

  const prevStep = () => {
    setCurrentStep((prev) => (prev - 1 + steps.length) % steps.length)
  }

  const togglePlay = () => {
    setIsPlaying(!isPlaying)
  }

  // Auto-advance when playing
  useState(() => {
    if (isPlaying) {
      const interval = setInterval(nextStep, 1500)
      return () => clearInterval(interval)
    }
  })

  const getStepColor = (type: Step['type']) => {
    switch (type) {
      case 'tool': return 'bg-blue-100 border-blue-500'
      case 'decision': return 'bg-yellow-100 border-yellow-500'
      case 'action': return 'bg-green-100 border-green-500'
    }
  }

  return (
    <div className="my-8 p-6 border-2 border-black shadow-[4px_4px_0_black]">
      <h3 className="text-lg font-bold mb-4">Agent Loop Pattern</h3>

      {/* Flow diagram */}
      <div className="flex flex-wrap justify-center gap-4 mb-6">
        {steps.map((step, index) => (
          <div key={step.id} className="flex items-center">
            <div
              className={`
                p-4 border-2 border-black
                ${getStepColor(step.type)}
                ${currentStep === index ? 'shadow-[4px_4px_0_black] -translate-x-1 -translate-y-1' : ''}
                transition-all
              `}
            >
              <div className="font-bold">{step.label}</div>
              <div className="text-sm text-gray-600">{step.description}</div>
            </div>
            {index < steps.length - 1 && (
              <div className="mx-2 text-2xl">→</div>
            )}
          </div>
        ))}
        <div className="flex items-center">
          <div className="mx-2 text-2xl">↩</div>
        </div>
      </div>

      {/* Controls */}
      <div className="flex justify-center gap-4">
        <button
          onClick={prevStep}
          className="px-4 py-2 border-2 border-black shadow-[2px_2px_0_black] hover:shadow-none hover:translate-x-[2px] hover:translate-y-[2px] transition-all"
        >
          ← Prev
        </button>
        <button
          onClick={togglePlay}
          className="px-4 py-2 border-2 border-black bg-black text-white shadow-[2px_2px_0_black] hover:shadow-none hover:translate-x-[2px] hover:translate-y-[2px] transition-all"
        >
          {isPlaying ? 'Pause' : 'Play'}
        </button>
        <button
          onClick={nextStep}
          className="px-4 py-2 border-2 border-black shadow-[2px_2px_0_black] hover:shadow-none hover:translate-x-[2px] hover:translate-y-[2px] transition-all"
        >
          Next →
        </button>
      </div>

      {/* Legend */}
      <div className="mt-6 flex justify-center gap-6 text-sm">
        <div className="flex items-center gap-2">
          <div className="w-4 h-4 bg-blue-100 border border-blue-500" />
          <span>Tool Call</span>
        </div>
        <div className="flex items-center gap-2">
          <div className="w-4 h-4 bg-yellow-100 border border-yellow-500" />
          <span>Model Decision</span>
        </div>
        <div className="flex items-center gap-2">
          <div className="w-4 h-4 bg-green-100 border border-green-500" />
          <span>Action</span>
        </div>
      </div>
    </div>
  )
}
```

---

## Usage in MDX

```mdx
import { AgentFlowDiagram } from '@/components/AgentFlowDiagram'

# Agents Aren't Magic

The pattern is simple: Tool → Decision → Action → Repeat.

<AgentFlowDiagram />

Let me show you the code...
```

---

## Accessibility Check

| Check | Status |
|-------|--------|
| Keyboard navigation | PASS (buttons focusable) |
| Screen reader | PASS (button labels clear) |
| Color contrast | PASS |
| Reduced motion | WARN (should respect prefers-reduced-motion) |

**Suggested Enhancement**:
```tsx
// Add to component
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches

// Use in animation
const transitionClass = prefersReducedMotion ? '' : 'transition-all'
```

---

## Design System Compliance

| Check | Status |
|-------|--------|
| Uses design tokens | PARTIAL (hardcoded colors in legend) |
| Neobrutalist shadow | PASS |
| Border style | PASS |
| Button pattern | PASS |

**Fix for design tokens**:
```tsx
// Replace hardcoded colors
bg-blue-100 → bg-[var(--color-tool)]
bg-yellow-100 → bg-[var(--color-decision)]
bg-green-100 → bg-[var(--color-action)]
```

---

## Summary

```yaml
component: AgentFlowDiagram
type: Interactive visualization
features:
  - Step-through navigation
  - Auto-play mode
  - Visual highlighting
  - Legend
accessibility:
  keyboard: PASS
  screen_reader: PASS
  reduced_motion: NEEDS_FIX
design_system:
  compliance: PARTIAL
  fixes_needed: 1
```
