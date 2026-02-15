# GitHub Copilot Instructions: Thorin Design System

Self-contained instructions for GitHub Copilot. This file does NOT reference external files.

---

## Project Overview

Thorin - A web3-native React component library for ECNS (Ethereum Classic Name Service). Built with TypeScript 5.7.2, React 18, and vanilla-extract CSS-in-JS.

**Package:** `@ensdomains/thorin` (beta)
**Repository:** github.com/ecnsdomains/thorin
**Status:** Alpha (v1.0.0-beta.26)

---

## Tech Stack (LTS Versions - 2026-02-12)

| Technology | Version | Notes |
|------------|---------|-------|
| Node.js | 24.x | LTS (requires >=20) |
| TypeScript | 5.7.2 | Strict mode |
| React | 18.3.1 | Peer dependency |
| pnpm | 9.4.0 | Package manager |
| Vite | 6.x | Build tool |
| Vitest | 3.x | Testing |
| ESLint | 9.x | Flat config |

### Core Dependencies
- `@vanilla-extract/css` 1.x - CSS-in-JS
- `@vanilla-extract/sprinkles` 1.x - Utility styles
- `@vanilla-extract/recipes` 0.5.x - Variant styles
- `@testing-library/react` 16.x - Component testing
- `ts-pattern` 5.x - Pattern matching
- `clsx` 2.x - Class name utility

---

## Project Structure

```
components/
├── src/
│   ├── components/
│   │   ├── atoms/         # Basic components (Button, Input, Avatar, Card, Tag)
│   │   ├── molecules/     # Compound components (Field, Modal, Toast, SearchInput)
│   │   └── organisms/     # Complex compositions (Forms, Navigation, Dialogs)
│   ├── tokens/            # Design tokens (colors, spacing, typography, shadows)
│   ├── css/               # Vanilla-extract utilities
│   ├── hooks/             # React hooks
│   ├── utils/             # Helper functions
│   └── icons.tsx          # Icon components
├── package.json
└── vite.config.ts
```

**Atomic Design Hierarchy:**
1. **Atoms** - Foundational elements (Button, Input, Avatar)
2. **Molecules** - Combinations of atoms (Field = Label + Input + Helper)
3. **Organisms** - Complex UI sections (Forms, Navigation)

---

## Commands

```bash
# Build
pnpm build:components        # Build library
pnpm build:watch            # Watch mode

# Test
pnpm test                   # Run Vitest
pnpm test:watch             # Watch mode
pnpm test:coverage          # Coverage report

# Lint
pnpm lint                   # ESLint
pnpm lint:styles            # Stylelint
pnpm lint:types             # TypeScript check

# Size
pnpm size                   # Check bundle size (< 150 KB limit)

# Publish
pnpm prepublish            # Clean + build
pnpm publish:local         # Publish to yalc
```

---

## Code Patterns

### Component Structure

Every component follows this pattern:

```
ComponentName/
├── ComponentName.tsx       # React component
├── ComponentName.css.ts    # Vanilla-extract styles
├── ComponentName.test.tsx  # Vitest tests
└── utils/                  # Helper functions (optional)
```

### Vanilla-Extract Recipe Pattern

```typescript
// Button.css.ts
import { recipe } from '@vanilla-extract/css'
import { tokens } from '@/src/tokens'

export const button = recipe({
  base: {
    fontFamily: tokens.fonts.sans,
    borderRadius: tokens.radii.large,
    transition: `all ${tokens.transitionDuration['150']} ${tokens.transitionTimingFunction.inOut}`,
  },
  variants: {
    size: {
      small: { padding: tokens.space['2'], fontSize: tokens.fontSizes.small },
      medium: { padding: tokens.space['3'], fontSize: tokens.fontSizes.base },
    },
    variant: {
      primary: {
        backgroundColor: tokens.colors.light.blue,
        color: tokens.colors.light.background,
      },
      secondary: {
        backgroundColor: tokens.colors.light.greyLight,
        color: tokens.colors.light.text,
      },
    },
  },
  defaultVariants: {
    size: 'medium',
    variant: 'primary',
  },
})
```

### Component API Pattern

```typescript
import * as React from 'react'
import { match } from 'ts-pattern'
import { clsx } from 'clsx'
import * as styles from './Button.css'

type Size = 'small' | 'medium' | 'flexible'

type BaseProps = {
  as?: 'a' | 'button'          // Polymorphic rendering
  children: React.ReactNode
  disabled?: boolean
  loading?: boolean
  prefix?: React.ReactNode      // Icon before text
  suffix?: React.ReactNode      // Icon after text
  size?: Size
  shape?: 'rectangle' | 'square' | 'rounded' | 'circle'
  pressed?: boolean             // Toggle state
  shadow?: boolean
}

export const Button = React.forwardRef<HTMLButtonElement, BaseProps>(
  ({ size = 'medium', variant = 'primary', children, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={clsx(styles.button({ size, variant }))}
        {...props}
      >
        {children}
      </button>
    )
  }
)

Button.displayName = 'Button'
```

### ts-pattern for Conditionals

```typescript
import { match } from 'ts-pattern'

const iconSize = match(size)
  .with('small', () => '3.5')
  .with('medium', () => '4')
  .with('flexible', () => undefined)
  .exhaustive()
```

### Testing Pattern

```typescript
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { describe, it, expect, vi } from 'vitest'
import { Button } from './Button'

describe('Button', () => {
  it('renders children', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument()
  })

  it('handles click events', async () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>Click</Button>)
    await userEvent.click(screen.getByRole('button'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('supports keyboard navigation', async () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>Submit</Button>)
    await userEvent.tab()
    expect(screen.getByRole('button')).toHaveFocus()
    await userEvent.keyboard('{Enter}')
    expect(handleClick).toHaveBeenCalled()
  })

  it('disables interaction when disabled', () => {
    render(<Button disabled>Submit</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

---

## Design Tokens

Located in `components/src/tokens/`:

```typescript
import { tokens } from '@/src/tokens'

// Colors (light/dark modes)
tokens.colors.light.blue
tokens.colors.dark.blue
tokens.colors.light.background
tokens.colors.light.text

// Spacing (0.25 = 1px, 4 = 16px, 128 = 512px)
tokens.space['1']   // 4px
tokens.space['2']   // 8px
tokens.space['4']   // 16px

// Typography
tokens.fonts.sans
tokens.fontSizes.small
tokens.fontSizes.base
tokens.fontSizes.large
tokens.fontWeights.medium
tokens.lineHeights.normal

// Borders
tokens.radii.large
tokens.radii.full
tokens.borderWidths['0.375']

// Shadows
tokens.shadows['0.02']

// Transitions
tokens.transitionDuration['150']
tokens.transitionDuration['200']
tokens.transitionTimingFunction.inOut

// Breakpoints (px)
tokens.breakpoints.xs   // 360
tokens.breakpoints.sm   // 640
tokens.breakpoints.md   // 768
tokens.breakpoints.lg   // 1024
tokens.breakpoints.xl   // 1280
```

**Rule:** Always use tokens. Never hardcode colors, spacing, or typography.

---

## Accessibility Requirements (WCAG 2.1)

### Keyboard Navigation
- All interactive elements accessible via Tab
- Enter/Space activate buttons
- Arrow keys navigate lists/menus
- Visible focus indicators required
- Logical tab order

### ARIA Attributes
```typescript
// Prefer semantic HTML
<button>Click</button>

// Use ARIA when needed
<button aria-label="Close modal" aria-pressed={pressed}>
  <Icon />
</button>
```

### Color Contrast
- Text: 4.5:1 minimum (WCAG AA)
- Large text (18pt+): 3:1 minimum
- UI components: 3:1 minimum

### Screen Reader Support
- Meaningful labels for all inputs
- Alt text for images
- Announce dynamic content changes

### Testing Checklist
- [ ] Keyboard navigation works
- [ ] Focus states visible
- [ ] Screen reader labels present
- [ ] Color contrast meets WCAG AA
- [ ] Works without mouse

---

## Bundle Size

**Target:** < 150 KB total bundle (defined in `.size-limit.json`)

Check before committing:
```bash
pnpm size
```

Optimization strategies:
- Tree-shakable exports
- Lazy-load large icons
- Minimal dependencies
- Use native APIs

---

## Rules

### Always Do
- Write Vitest tests (render, interaction, accessibility, edge cases)
- Use TypeScript strict mode
- Use design tokens (no magic values)
- Include ARIA attributes
- Follow atomic design hierarchy
- Run `pnpm lint:types && pnpm test && pnpm size` before committing
- Export from group index, then main index
- Set `displayName` on components
- Use `React.forwardRef` when refs needed

### Ask First
- Adding new dependencies
- Changing design token values
- Modifying component APIs (breaking changes)
- Renaming exported components

### Never Do
- Hardcode colors, spacing, typography (use tokens)
- Skip accessibility
- Commit without tests
- Use inline styles (use vanilla-extract)
- Exceed 150 KB bundle size
- Import from `react-dom` (use `@testing-library/react`)
- Use `styled-components` or `emotion` (this project uses vanilla-extract)

---

## Protected Files

Do not modify without explicit request:
- `package.json` (version, dependencies)
- `vite.config.ts` (build config)
- `.size-limit.json` (bundle limits)
- `components/src/tokens/` (design tokens)

---

## Validation

Before committing:

```bash
pnpm lint:types && pnpm test && pnpm size
```

Expected output:
- TypeScript: No errors
- Vitest: All tests pass
- Size: < 150 KB

---

## Code Style

### TypeScript
- Strict mode enabled
- Explicit return types for public APIs
- Use `type` over `interface` for props
- Prefer `React.ReactNode` over `JSX.Element`

### React
- Function components
- Use `React.forwardRef` for components accepting refs
- Set `displayName` for debugging
- Hooks at top level (no conditionals)
- Destructure props with defaults

### Vanilla-Extract
- Use `recipe()` for variant styles
- Use `sprinkles` for utility styles
- Export styles as named exports
- File naming: `ComponentName.css.ts`

### Testing
- One `describe` per component
- Test user behavior, not implementation
- Use `screen.getByRole` over `getByTestId`
- Mock with `vi.fn()`

---

## Example: Complete Component

```typescript
// components/src/components/atoms/Badge/Badge.tsx
import * as React from 'react'
import { clsx } from 'clsx'
import * as styles from './Badge.css'

type BadgeProps = {
  children: React.ReactNode
  variant?: 'primary' | 'secondary' | 'success' | 'error'
  size?: 'small' | 'medium'
}

export const Badge = ({ children, variant = 'primary', size = 'medium' }: BadgeProps) => {
  return (
    <span className={clsx(styles.badge({ variant, size }))}>
      {children}
    </span>
  )
}
```

```typescript
// components/src/components/atoms/Badge/Badge.css.ts
import { recipe } from '@vanilla-extract/css'
import { tokens } from '@/src/tokens'

export const badge = recipe({
  base: {
    display: 'inline-flex',
    alignItems: 'center',
    fontFamily: tokens.fonts.sans,
    fontWeight: tokens.fontWeights.medium,
    borderRadius: tokens.radii.full,
  },
  variants: {
    size: {
      small: {
        padding: `${tokens.space['1']} ${tokens.space['2']}`,
        fontSize: tokens.fontSizes.small,
      },
      medium: {
        padding: `${tokens.space['2']} ${tokens.space['3']}`,
        fontSize: tokens.fontSizes.base,
      },
    },
    variant: {
      primary: {
        backgroundColor: tokens.colors.light.blue,
        color: tokens.colors.light.background,
      },
      secondary: {
        backgroundColor: tokens.colors.light.greyLight,
        color: tokens.colors.light.text,
      },
      success: {
        backgroundColor: tokens.colors.light.green,
        color: tokens.colors.light.background,
      },
      error: {
        backgroundColor: tokens.colors.light.red,
        color: tokens.colors.light.background,
      },
    },
  },
  defaultVariants: {
    size: 'medium',
    variant: 'primary',
  },
})
```

```typescript
// components/src/components/atoms/Badge/Badge.test.tsx
import { render, screen } from '@testing-library/react'
import { describe, it, expect } from 'vitest'
import { Badge } from './Badge'

describe('Badge', () => {
  it('renders children', () => {
    render(<Badge>New</Badge>)
    expect(screen.getByText('New')).toBeInTheDocument()
  })

  it('applies variant styles', () => {
    const { container } = render(<Badge variant="success">Success</Badge>)
    expect(container.firstChild).toHaveClass(/success/)
  })

  it('applies size styles', () => {
    const { container } = render(<Badge size="small">Small</Badge>)
    expect(container.firstChild).toHaveClass(/small/)
  })
})
```

Export from `components/src/components/atoms/index.ts`:
```typescript
export { Badge } from './Badge/Badge'
```

Export from `components/src/index.ts`:
```typescript
export { Badge } from './components/atoms'
```

---

## Response Style

- Direct, concise answers
- Code first, explanations only if needed
- Bullet points over paragraphs
- No pleasantries
