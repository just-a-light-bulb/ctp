# Comic Translate Platform

A unified web-based manga translation platform built with modern web technologies. Consolidates OCR, translation management, and visual editing into a single application for efficient manga localization.

## 🎯 Features

- **Canvas Editor**: Interactive visual editor for manga pages with text element annotation
- **Multi-language Support**: Built-in i18n with English, Spanish, Thai, and Vietnamese
- **Database Management**: PostgreSQL with Drizzle ORM for robust data persistence
- **Authentication**: Kinde integration for secure user management
- **UI Components**: Pre-configured shadcn/svelte components with Tailwind CSS v4
- **Type Safety**: Full TypeScript support with strict type checking
- **Testing**: Vitest for unit tests and Playwright for E2E testing

## 🛠 Tech Stack

- **Framework**: [SvelteKit](https://kit.svelte.dev/) - Modern, powerful web framework
- **Language**: TypeScript with strict mode enabled
- **Styling**: [Tailwind CSS v4](https://tailwindcss.com/) with typography & forms plugins
- **Database**: PostgreSQL + [Drizzle ORM](https://orm.drizzle.team/)
- **UI Library**: [shadcn/svelte](https://www.shadcn-svelte.com/)
- **i18n**: [Paraglide](https://inlang.com/m/gerre34r/library-inlang-paraglideJs) for localization
- **Testing**: Vitest (unit) + Playwright (E2E)
- **Package Manager**: pnpm
- **Deployment**: Vercel adapter configured

## 🚀 Quick Start

### Prerequisites
- Node.js 18+ and pnpm

### Installation

```bash
pnpm install
```

### Development

```bash
pnpm dev              # Start dev server at http://localhost:5173
```

### Build & Deploy

```bash
pnpm build            # Production build
pnpm preview          # Preview production build locally
```

## 📋 Available Commands

### Development & Building
```bash
pnpm dev              # Start development server
pnpm build            # Create production build
pnpm check            # Type check with svelte-check
pnpm preview          # Preview production build
```

### Code Quality
```bash
pnpm lint             # Run ESLint + Prettier check
pnpm format           # Auto-format code with Prettier
```

### Testing
```bash
pnpm test             # Run all tests once
pnpm test:unit        # Run tests in watch mode
pnpm test src/demo.spec.ts  # Run specific test file
```

### Database
```bash
pnpm db:push          # Push schema changes to DB
pnpm db:generate      # Generate migrations
pnpm db:migrate       # Run migrations
pnpm db:studio        # Open Drizzle Studio GUI
```

## 📁 Project Structure

```
src/
├── lib/
│   ├── components/
│   │   ├── canvas-editor/    # Main editor component
│   │   ├── layout/           # Header, Footer, Layout components
│   │   └── ui/               # shadcn/svelte UI library
│   ├── server/
│   │   ├── auth.ts           # Kinde authentication helpers
│   │   └── db/
│   │       ├── index.ts      # Database connection
│   │       └── schema.ts      # Drizzle ORM schema
│   └── utils.ts              # Shared utilities
├── routes/
│   ├── (auth)/               # Protected routes
│   ├── api/                  # REST API endpoints
│   ├── project/[id]/         # Dynamic project routes
│   └── +layout.server.ts     # Global server-side layout
├── styles/                   # Global CSS
└── *.spec.ts                 # Test files
```

## 🎨 Code Style

This project follows consistent code standards:

- **Indentation**: Tabs (not spaces)
- **Quotes**: Single quotes
- **Print width**: 100 characters
- **Trailing commas**: None
- **Svelte 5**: Modern runes (`$state`, `$derived`, `$effect`, `$props`)
- **Imports**: Use `$lib` alias for local imports

See [AGENTS.md](./AGENTS.md) for detailed code style guidelines.

## 🔐 Security

- Never commit `.env` files or secrets
- Environment variables in `.env.local` (not tracked)
- Type-safe database operations with Drizzle ORM

## 📖 Documentation

- See [AGENTS.md](./AGENTS.md) for complete development guide
- See [`.docs/`](./.docs/) directory for additional documentation

## 🤝 Development Workflow

1. Pull latest: `git pull origin`
2. Create branch: `git checkout -b feature/your-feature`
3. Make changes and run linting: `pnpm lint`
4. Run build check: `pnpm build`
5. Commit: `git add`, `git commit`, `git push`
6. Open pull request

## 📝 License

See LICENSE file for details.
