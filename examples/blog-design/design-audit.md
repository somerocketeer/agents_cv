# Example: Design Audit

Example of the design system audit workflow.

---

## Trigger

```bash
design_auditor --component src/components/Button.tsx
```

---

## Input: Button Component

```tsx
// src/components/Button.tsx
import { ReactNode } from 'react'

interface ButtonProps {
  children: ReactNode
  onClick?: () => void
  variant?: 'primary' | 'secondary'
  disabled?: boolean
}

export function Button({ children, onClick, variant = 'primary', disabled }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`
        px-4 py-2 font-medium
        ${variant === 'primary' ? 'bg-black text-white' : 'bg-white text-black border border-black'}
        ${disabled ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'}
      `}
    >
      {children}
    </button>
  )
}
```

---

## Execution

### Agent: design_auditor (Claude Sonnet 4.5, temp 0.2)

**Audit Checklist**:

| Category | Check | Status |
|----------|-------|--------|
| **Accessibility** | | |
| | Focus states visible | FAIL |
| | Keyboard navigation | PASS |
| | ARIA labels | WARN |
| | Color contrast | PASS |
| **Design System** | | |
| | Uses design tokens | FAIL |
| | Follows component patterns | WARN |
| | Neobrutalist style | PARTIAL |
| **UX** | | |
| | Hover states | FAIL |
| | Active states | FAIL |
| | Loading state | MISSING |
| | Disabled state | PASS |

**Output**: `audit-report.md`
```markdown
## Design Audit: Button Component

### Critical Issues

#### 1. No Focus Ring (Accessibility - Critical)
**Location:** Button element
**Issue:** No visible focus indicator for keyboard navigation
**Impact:** Users navigating via keyboard cannot see which element is focused
**Fix:** Add focus ring using design tokens

```tsx
// Add to className
focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-black
```

#### 2. Hardcoded Colors (Design System - High)
**Location:** className
**Issue:** Using hardcoded values (`bg-black`, `text-white`) instead of design tokens
**Impact:** Inconsistent theming, harder to maintain
**Fix:** Use CSS variables

```tsx
// Instead of
bg-black text-white

// Use
bg-[var(--color-primary)] text-[var(--color-background)]
```

### Medium Issues

#### 3. Missing Hover/Active States (UX - Medium)
**Location:** className
**Issue:** No visual feedback on hover or click
**Impact:** Reduced interactivity feedback
**Fix:** Add neobrutalist hover effect

```tsx
// Add to className
hover:shadow-[2px_2px_0_black] hover:-translate-x-[2px] hover:-translate-y-[2px]
active:shadow-none active:translate-x-0 active:translate-y-0
transition-all
```

#### 4. Missing Shadow (Design System - Medium)
**Location:** className
**Issue:** Neobrutalist components should have offset shadow
**Impact:** Inconsistent with design system
**Fix:** Add default shadow

```tsx
shadow-[4px_4px_0_black]
```

### Low Issues

#### 5. No Loading State (UX - Low)
**Location:** Props interface
**Issue:** No way to indicate loading
**Impact:** Poor UX during async operations
**Fix:** Add `loading` prop with spinner

#### 6. Missing aria-label for Icon Buttons (Accessibility - Low)
**Location:** Props interface
**Issue:** If used with just an icon, no accessible name
**Impact:** Screen readers can't announce button purpose
**Fix:** Add optional `aria-label` prop

### Summary

| Severity | Count |
|----------|-------|
| Critical | 1 |
| High | 1 |
| Medium | 2 |
| Low | 2 |

**Recommendation:** Fix critical and high issues before shipping.
```

---

## Phase 2: Implementation

**Agent**: design_implementer (Claude Sonnet 4.5, temp 0.15)

```bash
design_implementer --task "Fix Button audit issues (critical and high)"
```

**Output**: Updated `Button.tsx`
```tsx
// src/components/Button.tsx
import { ReactNode } from 'react'

interface ButtonProps {
  children: ReactNode
  onClick?: () => void
  variant?: 'primary' | 'secondary'
  disabled?: boolean
  loading?: boolean
  'aria-label'?: string
}

export function Button({
  children,
  onClick,
  variant = 'primary',
  disabled,
  loading,
  'aria-label': ariaLabel
}: ButtonProps) {
  const baseStyles = `
    px-4 py-2 font-medium
    border-2 border-[var(--color-primary)]
    shadow-[4px_4px_0_var(--color-primary)]
    transition-all
    focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-[var(--color-primary)]
  `

  const variantStyles = {
    primary: 'bg-[var(--color-primary)] text-[var(--color-background)]',
    secondary: 'bg-[var(--color-background)] text-[var(--color-primary)]'
  }

  const interactiveStyles = disabled || loading
    ? 'opacity-50 cursor-not-allowed'
    : `
      cursor-pointer
      hover:shadow-[2px_2px_0_var(--color-primary)]
      hover:-translate-x-[2px] hover:-translate-y-[2px]
      active:shadow-none active:translate-x-0 active:translate-y-0
    `

  return (
    <button
      onClick={onClick}
      disabled={disabled || loading}
      aria-label={ariaLabel}
      aria-busy={loading}
      className={`${baseStyles} ${variantStyles[variant]} ${interactiveStyles}`}
    >
      {loading ? (
        <span className="flex items-center gap-2">
          <svg className="animate-spin h-4 w-4" viewBox="0 0 24 24">
            <circle
              className="opacity-25"
              cx="12" cy="12" r="10"
              stroke="currentColor"
              strokeWidth="4"
              fill="none"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"
            />
          </svg>
          {children}
        </span>
      ) : children}
    </button>
  )
}
```

---

## Phase 3: Review

**Agent**: design_reviewer (Claude Sonnet 4.5, temp 0.1)

```bash
design_reviewer --files src/components/Button.tsx
```

**Output**: `review-report.md`
```markdown
## Design Review: Button Component

### Checklist

| Check | Status |
|-------|--------|
| Uses design tokens | PASS |
| Focus states visible | PASS |
| Hover/active states | PASS |
| Neobrutalist shadow | PASS |
| Loading state | PASS |
| ARIA support | PASS |
| Keyboard accessible | PASS |

### Notes

- All critical and high issues resolved
- Loading state added with accessible busy indicator
- Design tokens used throughout
- Neobrutalist interaction pattern implemented correctly

### Remaining Suggestions

1. Consider adding `size` prop (sm, md, lg) for flexibility
2. Add unit tests for accessibility

### Verdict: APPROVED
```

---

## Summary

```yaml
audit:
  critical_issues: 1 → 0
  high_issues: 1 → 0
  medium_issues: 2 → 0
  low_issues: 2 → 1 (size prop deferred)
implementation:
  files_changed: 1
  lines_added: 42
  lines_removed: 12
review:
  status: APPROVED
```
