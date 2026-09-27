# Gemini Portal — 3D AI Workspace

A 3D-styled LLM portal with Google + Phone OTP + Email OTP login, MP3/MP4/photo cloud storage, and Gemini-style AI chat. Built with Next.js 16, Firebase, and a Python AI service.

![Gemini Portal](https://img.shields.io/badge/Next.js-16-black) ![Firebase](https://img.shields.io/badge/Firebase-Auth%20%2B%20Storage-orange) ![Python](https://img.shields.io/badge/Python-FastAPI-blue) ![TypeScript](https://img.shields.io/badge/TypeScript-5-blue)

## Features

- **3D UI/UX** — glassmorphic panels, tilt-on-hover cards, animated aurora background, holographic text, floating particles
- **Authentication** — Firebase Google OAuth (real popup), Firebase Phone OTP (real SMS via reCAPTCHA), Email OTP fallback (always works)
- **Country selector** — 🇮🇳+91, 🇺🇸+1, 🇬🇧+44, 🇦🇪+971, 🇸🇬+65, 🇦🇺+61
- **Privacy-first** — masked email (`r***@gmail.com`), masked phone (`•••••••43210`), OTP expiry countdown, Resend OTP button
- **Media storage** — MP3, MP4, photos — uploads to Firebase Storage with local server fallback
- **AI chat** — multi-turn context, markdown rendering, attach media to prompt, multiple LLM backends with automatic fallback
- **3D-styled** portal shell with Chat / Media / Settings tabs

## AI backend fallback chain

When the user sends a chat message, the portal tries each backend in order until one works:

1. **Gemini API** (via Python service) — real Google Gemini with your API key
2. **OpenAI-compatible API** (via Python service) — DeepSeek, OpenAI, Groq, Together AI, Mistral, etc.
3. **z-ai-web-dev-sdk** (Next.js fallback) — Gemini-equivalent that works in any region
4. **Offline reply** — if everything fails

## Tech stack

| Layer            | Technology                                       |
|------------------|--------------------------------------------------|
| Framework        | Next.js 16 (App Router) + TypeScript 5           |
| Styling          | Tailwind CSS 4 + shadcn/ui (New York)             |
| Auth             | Firebase Auth (Google + Phone) + Email OTP backend |
| Storage          | Firebase Storage + local disk fallback             |
| Database         | Prisma ORM — SQLite (dev) / PostgreSQL (Vercel)   |
| AI service       | Python FastAPI (dev) / Vercel Python function (prod) |
| State            | Zustand (client) + cookies (session)              |
| Animations       | Framer Motion                                     |

## Quick start (local dev)

```bash
# Install deps
bun install

# Set up the local DB
bun run db:push

# Start both Next.js + Python AI service together
bun run dev:all

# OR start them separately
bun run dev          # Next.js on :3000
bun run dev:python   # Python AI service on :5001
```

Open http://localhost:3000 in your browser.

## Deployment

See **[DEPLOY.md](./DEPLOY.md)** for the complete step-by-step Vercel deployment guide.

Quick summary:
1. Push to GitHub
2. Import to Vercel
3. Set env vars (DATABASE_URL, GEMINI_API_KEY, OPENAI_COMPATIBLE_API_KEY, Firebase config)
4. Deploy — `vercel.json` + `scripts/vercel-build.sh` handle the build

## Project structure

```
.
├── api/
│   └── chat-ai.py             # Vercel Python serverless function (Gemini + DeepSeek + fallback)
├── mini-services/
│   └── gemini-service/
│       ├── main.py            # Local Python FastAPI service (mirrors api/chat-ai.py)
│       └── package.json
├── prisma/
│   ├── schema.prisma          # SQLite schema (local dev)
│   └── schema.vercel.prisma   # PostgreSQL schema (Vercel)
├── scripts/
│   └── vercel-build.sh        # Build hook: swaps schema + prisma generate + db push
├── src/
│   ├── app/
│   │   ├── api/               # Next.js API routes (auth, chat, media)
│   │   ├── page.tsx           # Main entry — login or portal based on auth state
│   │   └── layout.tsx
│   ├── components/
│   │   ├── portal/            # All portal UI (auth, chat, media, shell)
│   │   └── ui/                # shadcn/ui components
│   ├── lib/
│   │   ├── firebase.ts        # Firebase client SDK init
│   │   ├── firebase-admin.ts  # Server-side ID token verification
│   │   └── portal/            # Auth + session helpers
│   ├── store/                 # Zustand stores
│   └── types/
├── public/uploads/            # Local media fallback (gitignored)
├── requirements.txt           # Vercel Python deps (google-genai, openai)
├── vercel.json                # Vercel config + build command
└── .env                       # Local env vars (gitignored)
```

## Environment variables

Copy `.env` and fill in your values. See `.env` for documentation of each var.

Required for full functionality:
- `GEMINI_API_KEY` — Google Gemini API key
- `OPENAI_COMPATIBLE_API_KEY` — DeepSeek/OpenAI-compatible key
- `OPENAI_COMPATIBLE_BASE_URL` — provider's /v1 endpoint
- `OPENAI_COMPATIBLE_MODEL` — model name
- `DATABASE_URL` — SQLite (dev) or PostgreSQL (Vercel)

Firebase config is currently hardcoded in `src/lib/firebase.ts` — for production you may want to move it to env vars (instructions in DEPLOY.md).

## Commands

```bash
bun run dev          # Start Next.js (port 3000)
bun run dev:python   # Start Python AI service (port 5001)
bun run dev:all      # Start both together
bun run lint         # ESLint check
bun run db:push      # Push Prisma schema to DB
bun run db:generate  # Generate Prisma client
```

## License

MIT — use this for your own projects.
