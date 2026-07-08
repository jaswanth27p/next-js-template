# Next.js Full-Stack Template — Design

Date: 2026-07-08

## Goal
Reusable starter template for future projects: shadcn/ui (sonner, sidebar, command), theming, fixed sidebar app layout, dashboard, simple email/password auth, zustand, TanStack Query, Prisma+Postgres, Zod, Lucide icons.

## Stack
- Next.js 16 App Router (existing repo base), TypeScript, Tailwind v4, pnpm
- shadcn/ui (New York style) — components: sidebar, sonner, command, dropdown-menu, button, input, label, card, avatar, form
- next-themes for light/dark/system
- zustand for client state
- @tanstack/react-query for server-state fetching
- Prisma + PostgreSQL (local via docker-compose)
- Zod for validation
- lucide-react icons (bundled via shadcn)
- Custom auth: bcryptjs password hashing + jose-signed JWT in httpOnly cookie (portable — verifiable later from a separate NestJS/Python backend via shared secret, unlike Auth.js's JWE session format)

## Auth flow
- `/register`, `/login` pages, zod-validated forms, outside sidebar layout
- Server actions: `register()`, `login()`, `logout()`
  - `register`: validate → hash pw (bcryptjs) → create User → sign JWT (jose, HS256) → set httpOnly cookie
  - `login`: validate → verify pw → sign JWT → set cookie
  - `logout`: clear cookie
- `middleware.ts` protects `(app)` route group — missing/invalid cookie → redirect `/login`
- `lib/session.ts`: `getSession()` / `requireUser()` helpers for server components
- Generic "invalid credentials" error on login failure (no user-enumeration)

## Layout
- `(app)/layout.tsx`: fixed shadcn sidebar (sidebar-07 pattern) + `SidebarInset` main content
  - Sidebar top: logo/app name
  - Sidebar middle: nav — Dashboard, Settings, Profile
  - Sidebar bottom: theme toggle + user dropdown (avatar/email/logout)
- `⌘K` command palette (shadcn `command`) — jumps to nav routes
- `(auth)/login`, `(auth)/register`: centered card, no sidebar

## Data / state
- Prisma schema: `User { id, email (unique), passwordHash, name?, createdAt }`
- `docker-compose.yml`: local postgres service; `.env.example` with `DATABASE_URL`, `JWT_SECRET`
- `lib/prisma.ts`: singleton PrismaClient
- TanStack Query: `Providers` wrapper w/ QueryClientProvider; example `useProfile()` hook on Dashboard demonstrating loading/error state
- Zustand: example `use-ui-store.ts` (sidebar-collapsed preference) demonstrating pattern

## Folder structure
```
app/
  (auth)/login/page.tsx
  (auth)/register/page.tsx
  (app)/layout.tsx
  (app)/dashboard/page.tsx
  (app)/settings/page.tsx
  (app)/profile/page.tsx
components/
  ui/...              (shadcn generated)
  app-sidebar.tsx
  nav-user.tsx
  theme-toggle.tsx
  theme-provider.tsx
  providers.tsx        (React Query provider)
lib/
  prisma.ts
  session.ts
  actions/auth.ts       (server actions)
  validations/auth.ts   (zod schemas)
  utils.ts
store/
  use-ui-store.ts
prisma/schema.prisma
middleware.ts
docker-compose.yml
.env.example
```

## Error handling
- zod `safeParse` → inline field errors on forms
- Server action failures → sonner toast
- Login failure → generic message, no leak of which field was wrong

## Out of scope (YAGNI)
No OAuth, no email verification, no password reset flow, no automated tests — kept minimal; add per-project as needed.
