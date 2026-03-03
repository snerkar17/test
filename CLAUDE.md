# Vocab Memory App — Claude Instructions

## Project Overview
A vocabulary learning web app that encodes words through personal anchors (memories, people, scenes) instead of definitions. See `brainstorm.md` for full product spec and architectural decisions.

## Tech Stack
- **Frontend**: React + Vite 5 + TypeScript
- **Styling**: Tailwind CSS (via PostCSS plugin — no separate build step)
- **State**: Zustand
- **Animations**: Framer Motion
- **Routing**: React Router v6
- **Auth / DB / Storage**: Supabase (client SDK + RLS, no custom backend)
- **Analytics**: PostHog
- **Deploy**: Vercel

## Commands

### Dev & Build
```bash
npm run dev        # start dev server (localhost:5173)
npm run build      # TypeScript compile + Vite build
npm run preview    # preview production build locally
```

### Type Checking & Lint
```bash
npm run typecheck  # tsc --noEmit, type-check without building
npm run lint       # eslint . --ext ts,tsx
```

### Supabase
```bash
npx supabase start                                                                 # start local stack (Docker)
npx supabase db push                                                               # push migrations to remote
npx supabase gen types typescript --project-id <id> > src/types/supabase.ts       # generate DB types
npx supabase db diff                                                               # diff local vs remote schema
```

### Vercel
```bash
vercel dev         # local dev with Vercel edge config
vercel --prod      # manual deploy to production
```

### package.json scripts
```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "typecheck": "tsc --noEmit",
    "lint": "eslint . --ext ts,tsx"
  }
}
```

## Color Palette
Always use these tokens — never hardcode hex values in components.

| Token | Hex | Role |
|---|---|---|
| `text-ink` / `bg-ink` | `#0e2e0f` | All body text, headings, labels |
| `bg-canvas` | `#d6edc7` | App background |
| `bg-primary` / `text-primary` | `#ce4527` | CTAs, buttons, destructive actions |
| `bg-secondary` | `#fff6dc` | Card backgrounds, surfaces |
| `bg-accent` / `text-accent` | `#67c190` | Highlights, success states, active indicators |

Defined via `@theme` in `src/index.css` using Tailwind v4 CSS variables.

## Key Conventions
- All Supabase calls go through `src/lib/supabase.ts` — never import the client directly in components
- Generated DB types live in `src/types/supabase.ts` — regenerate after any schema change
- Never use inline styles — always use Tailwind classes
- Never use `any` — run `npm run typecheck` before considering a task done
