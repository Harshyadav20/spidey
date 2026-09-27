# Deploying Gemini Portal to Vercel

This guide walks you through deploying the Gemini Portal (Next.js + Python serverless + Firebase) to Vercel.

## Architecture on Vercel

| Component            | Local dev                            | Vercel production                          |
|----------------------|--------------------------------------|--------------------------------------------|
| Next.js app          | `next dev -p 3000`                   | Vercel Next.js serverless                  |
| Python AI service    | `mini-services/gemini-service/` on :5001 | `api/chat-ai.py` Vercel Python serverless |
| Database             | SQLite (file:./db/custom.db)         | PostgreSQL (Neon / Vercel Postgres)        |
| Media storage        | Local `/public/uploads` (fallback)   | Firebase Storage (real cloud)              |
| Firebase Auth        | Real Firebase Google + Phone OTP     | Same — works on Vercel                     |

## Prerequisites

1. **A Vercel account** — sign up at https://vercel.com (free tier works)
2. **Vercel CLI installed** — `npm i -g vercel` (or use the Vercel dashboard)
3. **A Postgres database** — easiest options:
   - **Neon** (free, recommended): https://neon.tech — sign up, create a project, copy the connection string
   - **Vercel Postgres**: https://vercel.com/docs/storage/vercel-postgres — create from Vercel dashboard
   - **Supabase**: https://supabase.com — create a project, copy the Postgres URL
4. **Firebase project configured** (already done — `spideykim-9d5cf`):
   - Enable Google + Phone sign-in methods (Firebase Console → Authentication → Sign-in method)
   - Enable Firebase Storage (Firebase Console → Storage → Get started)
   - **Authorized domains**: Add your Vercel deployment URL (e.g. `your-project.vercel.app`) under Authentication → Settings → Authorized domains

## Step-by-step deployment

### Step 1: Push your code to GitHub

If you haven't already:
```bash
git init
git add .
git commit -m "Initial commit — Gemini Portal with Firebase + Python AI"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

### Step 2: Create a new Vercel project

**Option A — via Vercel Dashboard:**
1. Go to https://vercel.com/new
2. Import your GitHub repo
3. **Framework Preset**: Next.js (auto-detected)
4. **Build Command**: leave as default (uses `vercel.json`'s `bash scripts/vercel-build.sh && next build`)
5. **Install Command**: `bun install`
6. Skip the env vars for now (we'll set them in Step 3)
7. Click **Deploy**

**Option B — via Vercel CLI:**
```bash
npm i -g vercel          # one-time
vercel login             # one-time
vercel link              # in your project root — links the repo to a Vercel project
vercel --prod            # deploys to production
```

### Step 3: Set environment variables

In your Vercel project dashboard → **Settings → Environment Variables**, add each of these (Production + Preview environments):

#### Database (required)
| Key                  | Value                                                                       |
|----------------------|-----------------------------------------------------------------------------|
| `DATABASE_PROVIDER`  | `postgresql`                                                                |
| `DATABASE_URL`       | Your Postgres connection string (e.g. `postgresql://user:pass@host/db?sslmode=require`) |

#### AI backends (at least one required)
| Key                              | Value                                                        |
|----------------------------------|--------------------------------------------------------------|
| `GEMINI_API_KEY`                 | Your Gemini API key (`AQ.Ab8RN6KqZDUpl0Iu3GWdutw0yG_ihpQ-4Kn_ebY7JIRh8aJJog`) |
| `GEMINI_MODEL`                   | `gemini-3.8-flash` (or whatever Gemini model you want)        |
| `OPENAI_COMPATIBLE_API_KEY`       | Your DeepSeek key (`sk-dfd4c7460ce84525aa0e9ac752faf158`)     |
| `OPENAI_COMPATIBLE_BASE_URL`     | `https://api.deepseek.com/v1` (or your provider's /v1 URL)    |
| `OPENAI_COMPATIBLE_MODEL`         | `deepseek-flash` (or `gpt-4o-mini` for OpenAI)               |

> **Tip**: To use OpenAI instead of DeepSeek, set `OPENAI_COMPATIBLE_BASE_URL=https://api.openai.com/v1` and `OPENAI_COMPATIBLE_MODEL=gpt-4o-mini`.

#### Firebase (required for auth + storage)
These are baked into `src/lib/firebase.ts` already (your `spideykim-9d5cf` config), so you only need them if you want to override at deploy time. To keep them secret, add them as Vercel env vars and replace the hardcoded values in `src/lib/firebase.ts` with `process.env.NEXT_PUBLIC_FIREBASE_*`.

| Key                                    | Value                                              |
|----------------------------------------|----------------------------------------------------|
| `NEXT_PUBLIC_FIREBASE_API_KEY`         | `AIzaSyDPYI4X8AikTsVPdnuK072kjfp3oMoD1OQ`          |
| `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN`     | `spideykim-9d5cf.firebaseapp.com`                  |
| `NEXT_PUBLIC_FIREBASE_PROJECT_ID`      | `spideykim-9d5cf`                                  |
| `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET` | `spideykim-9d5cf.firebasestorage.app`              |
| `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` | `391527621275`                                |
| `NEXT_PUBLIC_FIREBASE_APP_ID`          | `1:391527621275:web:b9ed55b5115c97b183dc3e`        |

### Step 4: Redeploy

After setting env vars, redeploy (Vercel → Deployments → Redeploy, or `vercel --prod`).

### Step 5: Configure Firebase Authorized Domains

1. Go to Firebase Console → Authentication → Settings → Authorized domains
2. Add your Vercel deployment URL: `your-project.vercel.app`
3. (Optional) Add your custom domain if you've configured one

### Step 6: Test your deployment

1. Visit `https://your-project.vercel.app`
2. Try signing in with Google (real Firebase Google OAuth)
3. Try signing in with Phone (real SMS)
4. Try email OTP — note: in production, the demo code is hidden, you'd need a real email gateway to deliver the OTP (the email OTP fallback works only for testing in dev mode)

## How the build works

The `vercel.json` build command runs `bash scripts/vercel-build.sh && next build`. The `vercel-build.sh` script:

1. Checks `DATABASE_PROVIDER` env var
2. If `postgresql`, swaps `prisma/schema.prisma` with `prisma/schema.vercel.prisma`
3. Runs `prisma generate`
4. Runs `prisma db push --accept-data-loss` (creates/migrates the schema on first deploy)
5. Then `next build` runs as normal

The `api/chat-ai.py` Vercel Python serverless function:
- Installed via `requirements.txt` (google-genai, openai)
- Becomes available at `/api/chat-ai`
- Mirrors the logic in `mini-services/gemini-service/main.py`
- Same fallback chain: Gemini → OpenAI-compatible → offline reply

## Local dev commands

```bash
bun run dev          # Start Next.js only (port 3000)
bun run dev:python   # Start only the Python AI mini-service (port 5001)
bun run dev:all      # Start both Next.js + Python service together
bun run lint         # ESLint check
bun run db:push      # Push Prisma schema to local SQLite
```

## Troubleshooting

### "User location is not supported for the API use" (Gemini)
Your Vercel deployment region is geo-restricted by Google. Either:
- Choose a different Vercel deployment region (Vercel → Settings → Functions → Function Region) — try `iad1` (Washington DC) or `fra1` (Frankfurt)
- OR rely on the OpenAI-compatible (DeepSeek) fallback, which has fewer geo-restrictions

### "Insufficient Balance" (DeepSeek)
Your DeepSeek account has $0 balance. Top up at https://platform.deepseek.com/usage

### "Firebase: auth/domain-not-authorized"
Add your Vercel deployment URL to Firebase Console → Authentication → Settings → Authorized domains

### "Prisma can't connect to database"
Make sure `DATABASE_URL` is set in Vercel env vars and uses the format `postgresql://user:pass@host:port/db?sslmode=require`. Neon, Vercel Postgres, and Supabase all provide this string on their dashboards.

### File uploads don't persist on Vercel
That's expected — Vercel serverless has a read-only filesystem. The portal already handles this:
- On Vercel, media uploads go to **Firebase Storage** (real cloud storage)
- The local `/public/uploads` fallback is silently skipped (write fails → falls back to Firebase Storage)

To verify, check the upload toast — it should say "Uploaded to Firebase Storage" not "using local server upload".

### Python function returns 502 / 500
Check the Vercel function logs (Vercel → Functions → chat-ai → Logs). Common causes:
- Missing `GEMINI_API_KEY` or `OPENAI_COMPATIBLE_API_KEY` in env vars
- Function timeout — increase `maxDuration` in `vercel.json`
- Python package install failures — verify `requirements.txt` is at the project root

## Removing the demo OTP code in production

The email OTP demo code is automatically hidden in production (when `NODE_ENV=production`).
Vercel sets `NODE_ENV=production` automatically, so this works out of the box.

For real email OTP delivery in production, you'd need to integrate an email provider (SendGrid, Resend, Postmark, AWS SES) in `/api/auth/otp/send/route.ts` — replace the `devCode` logic with a real email send call.
