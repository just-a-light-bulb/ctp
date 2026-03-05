# Comic Translate Platform - Agent Guide

A unified web-based manga translation platform built with SvelteKit and Drizzle ORM. Consolidates OCR, translation management, and visual editing into a single application.

Additional documentation: `./.docs/*`

## Build, Lint & Test Commands

```bash
# Development
pnpm dev                    # Start dev server

# Build & Type Check
pnpm build                  # Production build
pnpm check                  # Type check with svelte-check

# Linting & Formatting
pnpm lint                   # Run ESLint + Prettier check
pnpm format                 # Auto-format with Prettier

# Testing
pnpm test                   # Run all tests once
pnpm test:unit              # Run tests in watch mode
pnpm test src/demo.spec.ts  # Run a single test file
pnpm test -- --run --reporter=verbose  # Run once with verbose output

# Database (Drizzle)
pnpm db:push                # Push schema changes to DB
pnpm db:generate            # Generate migrations
pnpm db:migrate             # Run migrations
pnpm db:studio              # Open Drizzle Studio
```

## Code Style Guidelines

### Formatting (Prettier)

- **Tabs**: Use tabs for indentation (`useTabs: true`)
- **Quotes**: Single quotes for strings
- **Trailing commas**: None
- **Print width**: 100 characters
- **Svelte**: Prettier auto-formats `.svelte` files

### Imports

- Use `$lib` alias for imports from `src/lib/`
- Use `$app` for SvelteKit internals (`$app/environment`, `$app/navigation`, `$app/stores`)
- Group imports: framework first, then libraries, then local modules
- Example order: SvelteKit → Drizzle → Other libs → `$lib/` → Relative

```typescript
import { fail } from '@sveltejs/kit';
import { eq, desc } from 'drizzle-orm';
import { db } from '$lib/server/db';
import { project } from '$lib/server/db/schema';
import { requireUser } from '$lib/server/auth';
import type { Actions, PageServerLoad } from './$types';
```

### TypeScript

- **Strict mode**: Enabled. All code must pass strict type checking.
- **Type imports**: Use `import type { X }` for type-only imports
- **Return types**: Explicitly type exported functions (`PageServerLoad`, `RequestHandler`)
- **Props**: Use Svelte 5 `$props()` with interface definitions

```svelte
<script lang="ts">
	interface Props {
		width?: number;
		height?: number;
	}
	let { width = 800, height = 1100 }: Props = $props();
</script>
```

### Svelte 5 Conventions

- **Runes**: Use `$state()`, `$derived()`, `$effect()`, `$props()` - no legacy reactive declarations
- **No comments in code**: Do not add code comments unless explicitly requested
- **Event handlers**: Use `onclick`, `onkeydown` (lowercase) in markup

### Naming Conventions

- **Files**: kebab-case (`canvas-editor/`, `translation-grid.svelte`)
- **Components**: PascalCase exports (`Canvas.svelte`, `TopToolbar.svelte`)
- **Variables**: camelCase (`gridExpanded`, `canvasWidth`)
- **Constants**: SCREAMING_SNAKE_CASE for true constants (`ITEMS_PER_PAGE`)
- **Database tables**: snake_case in schema (`text_element`, `account_profile`)
- **Server routes**: Use `+page.server.ts`, `+server.ts`, `+layout.server.ts`

### Error Handling

- **Server actions**: Use `fail()` from `@sveltejs/kit` for form validation errors
- **API endpoints**: Return `json({ error: 'message' }, { status: 400 })`
- **Auth errors**: Throw `error(401, 'Unauthorized')` via SvelteKit's error helper
- **Validation**: Validate early, return early pattern

```typescript
if (!name) {
	return fail(400, { error: 'Project name is required.' });
}
```

### Database (Drizzle ORM)

- Schema defined in `src/lib/server/db/schema.ts`
- Use typed query builders: `db.select().from(table).where(eq(table.id, id))`
- Use `$onUpdate(() => new Date())` for auto-updating timestamps
- Foreign keys should use `references(() => table.id, { onDelete: 'cascade' })`

### UI Components

- Use `pnpm dlx shadcn-svelte@latest add <component>` to add shadcn/svelte components
- UI components live in `src/lib/components/ui/`
- Check shadcn/svelte docs before building UI from scratch
- Tailwind CSS v4 with `@tailwindcss/vite` plugin

## Project Structure

```
src/
├── lib/
│   ├── components/
│   │   ├── canvas-editor/    # Main editor feature
│   │   ├── layout/           # Header, Footer
│   │   └── ui/               # shadcn/svelte components
│   ├── server/
│   │   ├── auth.ts           # Kinde auth helpers
│   │   └── db/               # Drizzle schema & connection
│   └── utils.ts              # Shared utilities
├── routes/
│   ├── (auth)/               # Auth-gated routes
│   ├── api/                  # REST endpoints
│   ├── project/[id]/         # Project pages
│   └── +layout.server.ts     # Global auth guard
└── *.spec.ts                 # Test files
```

## Development Workflow

1. **Before starting**: Run `git pull origin` and create a new branch if on main
2. **After changes**: Run `pnpm lint` - fix all errors before finishing
3. **Build check**: Run `pnpm build` to ensure clean builds
4. **Commit**: Use `git add`, `git commit`, `git push` after task completion
5. **Security**: Never commit secrets (`.env`, credentials)
6. **PR**: Open a pull request after successful task completion

## Skills & MCP Tools

Use available skills for improved performance:

- **brainstorming** - Before creative work or new features
- **svelte5-best-practices** / **svelte-code-writer** - When writing Svelte code
- **frontend-design** / **ui-ux-pro-max** - When building UI components
- **writing-plans** / **executing-plans** - For multi-step tasks

Svelte MCP Tools (use for Svelte/SvelteKit questions):

1. `list-sections` - First, to discover documentation sections
2. `get-documentation` - Fetch relevant Svelte docs
3. `svelte-autofixer` - Validate Svelte code before sending to user
4. `playground-link` - Generate playground link (only if user requests, never for project files)