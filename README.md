# SuperCovo — AI-Powered Resume Tailoring

**Live product → [supercovo.com](https://www.supercovo.com)**

SuperCovo tailors your resume and cover letter to any job description in under 60 seconds. It scores your alignment before and after generation, exports to PDF and Google Docs, and includes a Chrome extension that works on any job board.

---

## What it does

- **Resume tailoring** — paste a job description, get a tailored resume and cover letter generated specifically for that role
- **Alignment scoring** — see how well your profile matches the JD before and after generation, with matched and missing keywords highlighted
- **Multiple export formats** — download as PDF (Classic or Modern template), export to Google Docs, or send to email
- **Chrome extension** — right-click any job listing on any website to analyse it instantly against your profile
- **Job application tracking** — track applications through Generated → Applied → Screening → Interview → Offer stages
- **Mock interview prep** — open a role-specific AI interviewer in Claude or ChatGPT with one click

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     supercovo.com                        │
│                   Next.js 16 / Vercel                    │
└───────────────┬─────────────────────┬───────────────────┘
                │                     │
     ┌──────────▼──────────┐ ┌────────▼────────────┐
     │   Supabase           │ │  Anthropic Claude    │
     │   PostgreSQL + Auth  │ │                      │
     │   RLS + pg_cron      │ │  Haiku — analysis    │
     └──────────────────────┘ │  Sonnet — generation │
                               └─────────────────────┘
                │                     │
     ┌──────────▼──────────┐ ┌────────▼────────────┐
     │   Stripe             │ │  Google Drive API    │
     │   Subscriptions      │ │  OAuth 2.0 export    │
     │   Credit packs       │ └─────────────────────┘
     └──────────────────────┘
```

**Two-model AI pipeline:**
- **Claude Haiku** — fast, low-cost model used for job description analysis, alignment scoring, and keyword extraction. Runs twice per generation cycle (pre and post).
- **Claude Sonnet** — high-quality model used for resume and cover letter generation. Outputs structured content via delimiter-based streaming.

**Key technical decisions:**
- SHA-256 JD analysis caching — eliminates redundant Haiku calls for identical job descriptions
- Delimiter-based SSE streaming (`---RESUME---`) — allows progressive resume rendering without JSON bleed
- Append-only credit ledger — subscription and add-on balances tracked separately, never mutated
- `pg_cron` scheduled SQL function — monthly credit top-ups run entirely in the database, no cron route needed
- Google Docs export via Drive API `drive.file` scope — avoids the OAuth "unverified app" screen triggered by the `documents` scope

---

## Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 16 (App Router, Turbopack), TypeScript, Tailwind CSS v4 |
| Backend | Next.js API routes (Node.js runtime) |
| Database | Supabase (PostgreSQL, RLS, Auth, pg_cron) |
| AI | Anthropic Claude (Haiku + Sonnet) |
| Payments | Stripe (subscriptions + credit packs + webhooks) |
| Auth | Supabase Auth (email + Google OAuth) |
| PDF generation | React-based PDF renderer (two templates) |
| Browser extension | Chrome/Edge/Brave — Manifest V3 |
| Monitoring | Sentry |
| Analytics | Custom event ledger (fire-and-forget) |
| Deployment | Vercel |

---

## Product decisions worth noting

**Why annual subscription only?** Reduces churn risk at early stage and simplifies credit economics. Monthly drip (20 credits/month) is handled by a Supabase scheduled function, not a webhook dependency.

**Why two models instead of one?** Haiku handles analysis tasks (fast, cheap, good enough) while Sonnet handles generation (slow, expensive, necessary). Cost per generation cycle is ~$0.064 — margin is preserved without sacrificing output quality.

**Why clipboard for mock interview prep?** Anthropic's voice API isn't yet exposed to third-party developers. The feature redirects users to Claude.ai or ChatGPT with a pre-compressed, role-specific interviewer prompt. Zero server cost — runs on the user's own AI account.

**Why not build a WYSIWYG editor?** Google Docs export gives users a fully editable document in a tool they already know. Building a custom editor would cost weeks with unclear incremental value over a two-second export.

---

## Status

Live with subscribers. Chrome extension published on the Chrome Web Store.

---

*This repository contains no source code. The codebase is private.*
