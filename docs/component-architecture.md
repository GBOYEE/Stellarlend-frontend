# Component Architecture

This document defines the component layering model, dependency rules, and folder responsibilities for the StellarLend frontend.

## Layer Overview

The project uses a hybrid atomic-design + feature-based architecture. Components are organized into six layers with strict import-direction rules.

## Layers

### 1. `components/atoms/` — Primitive UI Elements

The smallest building blocks. No business logic, no side effects.

**Responsibilities:**
- Buttons, inputs, labels, badges, tooltips
- Pure presentational elements
- Highly reusable across the entire app

**Import rules:**
- ✅ May import: `shared/ui` primitives, Tailwind utilities
- ❌ Must NOT import: `features`, `molecules`, `organisms`, `shared/layout`

**Examples:** `Button`, `IconButton`, `Tooltip`, `ScrollCues`

### 2. `components/molecules/` — Composed UI

Combinations of atoms that form a meaningful unit.

**Responsibilities:**
- Search bars, form groups, card headers
- Composed of 2-5 atoms
- May have local state (e.g., toggle, focus)

**Import rules:**
- ✅ May import: `atoms`, `shared/ui`
- ❌ Must NOT import: `features`, `organisms`, `shared/layout`

**Examples:** `SearchBar`

### 3. `components/organisms/` — Complex UI Sections

Large, self-contained UI sections.

**Responsibilities:**
- Headers, footers, sidebars, navigation bars
- May compose atoms + molecules
- May have internal state and side effects

**Import rules:**
- ✅ May import: `atoms`, `molecules`, `shared/ui`, `shared/layout`
- ❌ Must NOT import: `features`

**Examples:** `Header`

### 4. `components/features/` — Feature-Specific Components

Components tied to a specific domain or page.

**Responsibilities:**
- Lending forms, dashboard widgets, account management
- Contain business logic and data fetching
- May use hooks from `hooks/` and API clients from `lib/`

**Import rules:**
- ✅ May import: ALL layers (`atoms`, `molecules`, `organisms`, `shared`)
- ❌ Must NOT be imported by: `atoms`, `molecules`, `organisms`

**Sub-folders:**
- `features/lending/` — Lending page components
- `features/dashboard/` — Dashboard widgets
- `features/account/` — Account management

### 5. `components/shared/` — Shared Utilities

Cross-cutting UI, layout, and common components.

**Sub-folders:**
- `shared/ui/` — Shared UI primitives and icon sets
- `shared/layout/` — Layout shells, page wrappers
- `shared/common/` — Shared utilities used across features

**Import rules:**
- ✅ May import: `atoms`
- ❌ Must NOT import: `features`, `molecules`, `organisms`

### 6. `components/` (root) — Top-Level Components

Legacy top-level components that haven't been migrated to a specific layer yet.

**Known debt:**
- Root-level `Button/` duplicates `atoms/Button/` — prefer `atoms/Button`
- Root-level `Input/`, `Modal/`, `NavLink/`, `Pagination/` should be migrated to appropriate layers

## Dependency Direction

```
features → organisms → molecules → atoms → shared/ui
                     ↘ shared/layout ↗
```

**Golden rule:** Import direction flows downward. A layer may only import from layers below it in the diagram above.

## Where Does My Component Go?

| Component type | Layer |
|---------------|-------|
| Button, input, badge | `atoms/` |
| Search bar, form group | `molecules/` |
| Header, footer, sidebar | `organisms/` |
| Lending form, dashboard widget | `features/{domain}/` |
| Layout shell, page wrapper | `shared/layout/` |
| Shared icon set | `shared/ui/` |

## Known Technical Debt

1. **Duplicate Button**: Both `components/Button/` and `components/atoms/Button/` exist. New code should use `atoms/Button`.
2. **Root-level components**: `Input`, `Modal`, `NavLink`, `Pagination` at root should be migrated to `atoms/` or `molecules/`.
3. **SearchBar duplication**: `components/SearchBar/` and `components/molecules/SearchBar/` exist. Use `molecules/SearchBar`.

## Cross-references

- [CONTRIBUTING.md](../../CONTRIBUTING.md) — General contribution guidelines
- [COMPONENT-CHECKLIST.md](../../COMPONENT-CHECKLIST.md) — Component quality checklist
- [PERFORMANCE_BUDGETS.md](../../docs/PERFORMANCE_BUDGETS.md) — Performance budgets
