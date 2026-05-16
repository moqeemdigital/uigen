# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — Start dev server (Next.js 15 + Turbopack at localhost:3000)
- `npm run build` — Production build
- `npm run test` — Run all tests with Vitest
- `npm run test -- -t "test name"` — Run a single test by name pattern
- `npm run setup` — Install deps + generate Prisma client + run migrations
- `npm run db:reset` — Reset SQLite database and re-run all migrations

## Formatting

- Heading fonts should be bold and large
- Use regular fonts for regular text.
- Use Color Green and Blue fo rtext and for background light yellow

## Code Style

- **Comments**: Use sparingly. Only comment non-obvious code — hidden constraints, subtle invariants, or workarounds. Never comment what the code does (well-named identifiers are enough) or reference the current task/callers.

## Prisma Database Schema

The database schema is defined in `prisma/schema.prisma` — reference this file whenever working with the database schema.

## Architecture Overview

**UI Generator** — an AI-powered React component generator with live preview. Users describe components in a chat, and Claude generates them in a virtual file system that renders live.

### Project Structure

```
src/
├── app/
│   ├── layout.tsx              # Root layout (Geist fonts, global CSS)
│   ├── page.tsx                # Home — redirects authed users to project, shows MainContent for anon
│   ├── [projectId]/page.tsx    # Project page — loads project by ID for authed users
│   ├── main-content.tsx        # Core layout: resizable chat (left) + preview/code (right)
│   ├── api/chat/route.ts       # POST /api/chat — AI SDK streaming endpoint
│   └── globals.css
├── components/
│   ├── chat/                   # ChatInterface, MessageList, MessageInput, MarkdownRenderer
│   ├── editor/                 # FileTree (recursive tree view), CodeEditor (Monaco)
│   ├── preview/                # PreviewFrame (iframe with import-map-based rendering)
│   └── auth/                   # AuthDialog, SignInForm, SignUpForm
├── lib/
│   ├── contexts/
│   │   ├── chat-context.tsx     # Wraps @ai-sdk/react useChat, sends files + projectId
│   │   └── file-system-context.tsx  # React context around VirtualFileSystem
│   ├── file-system.ts           # VirtualFileSystem class (in-memory, no disk writes)
│   ├── provider.ts              # Language model provider (Anthropic or mock fallback)
│   ├── prompts/generation.tsx   # System prompt for Claude
│   ├── tools/
│   │   ├── str-replace.ts       # str_replace_editor tool (view/create/replace/insert)
│   │   └── file-manager.ts      # file_manager tool (rename/delete)
│   ├── transform/jsx-transformer.ts  # Babel transform + import map generation + preview HTML
│   ├── auth.ts                  # JWT session management (jose library)
│   └── anon-work-tracker.ts     # localStorage tracker for anonymous user work
├── actions/
│   ├── index.ts                 # Server actions: signUp, signIn, signOut, getUser
│   ├── create-project.ts        # Create new project in DB
│   ├── get-project.ts           # Load single project
│   └── get-projects.ts          # List user's projects
└── middleware.ts                # Auth middleware (protects /api/projects, /api/filesystem)
prisma/
└── schema.prisma                # SQLite: User + Project models
```

### Key Design Decisions

- **Virtual File System**: All generated files live in-memory (`VirtualFileSystem` class), never written to disk. The file system is serialized/deserialized when saving/loading projects.
- **Preview System**: Files are Babel-transformed in the browser, converted to blob URLs, then loaded via import maps in an iframe with Tailwind CDN. No bundler — just esm.sh for React and Babel-standalone for JSX transform.
- **AI Tool Integration**: The chat endpoint exposes two tools to Claude — `str_replace_editor` (view/create/replace/insert in files) and `file_manager` (rename/delete). Tool calls are handled server-side on the VirtualFileSystem, and results stream back to the client.
- **Auth**: JWT-based sessions stored in httpOnly cookies. Middleware protects API routes. Uses bcrypt for passwords and Prisma/SQLite for persistence.
- **Mock Provider**: If `ANTHROPIC_API_KEY` is unset or placeholder, a `MockLanguageModel` returns canned components to let users try the UI without an API key.
- **Anonymous Work**: Unauthenticated users' files and messages are tracked in localStorage so they can sign up without losing work.
