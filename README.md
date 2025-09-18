````markdown
# Music Generator SaaS

<p align="center">
  <img src="thumbnail.png" alt="Music Generator Thumbnail" width="320" />
</p>

<p align="center">
  <a href="https://github.com/sh-arka22/music-generator/actions">
    <img alt="CI status" src="https://img.shields.io/github/actions/workflow/status/sh-arka22/music-generator/ci.yml?logo=github">
  </a>
  <a href="https://github.com/sh-arka22/music-generator/blob/main/LICENSE.MD">
    <img alt="License" src="https://img.shields.io/badge/license-MIT-blue.svg">
  </a>
  <a href="https://modal.com/">
    <img alt="Built with Modal" src="https://img.shields.io/badge/Built%20with-Modal-000">
  </a>
</p>

> A full-stack AI music creation platform by
> <a href="https://github.com/sh-arka22">Arka Jyoti Saha</a>, combining state-of-the-art generative audio, LLM-assisted songwriting, GPU-backed serverless execution, and a polished Next.js experience.

---

## Table of Contents

- [Highlights](#-highlights)
- [System Architecture](#-system-architecture)
- [Directory Structure](#-directory-structure)
- [Tech Stack](#-tech-stack)
- [Quick Start](#-quick-start)
  - [Backend (Modal)](#2-backend-modal)
  - [Frontend (Nextjs)](#3-frontend-nextjs)
  - [Queue Worker (Inngest)](#4-queue-worker-inngest)
- [Configuration](#-configuration)
  - [Environment Variables](#environment-variables)
  - [AWS S3 Setup](#aws-s3-setup)
  - [Database (Neon + Prisma)](#database-neon--prisma)
  - [Modal Setup](#modal-setup)
  - [Inngest Setup](#inngest-setup)
- [Generation Modes & API](#-generation-modes--api)
- [Processing Pipeline](#-processing-pipeline)
- [Deployment & Ops](#-deployment--ops)
- [Security & Privacy](#-security--privacy)
- [Troubleshooting](#-troubleshooting)
- [FAQ](#-faq)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [Maintainer](#-maintainer)
- [License](#license)

---

## ✨ Highlights

- **ACE-Step diffusion audio model** for high-fidelity music synthesis.
- **Qwen2-7B-Instruct** powers prompt expansion, lyric writing, and auto-tagging.
- **SDXL Turbo** generates album-ready artwork on the fly.
- **Modal GPU functions** keep inference scalable without server management.
- **Next.js 15 + BetterAuth** deliver a production-ready SaaS front end and auth.
- **Credit-based monetization** via Polar checkout & customer portal.
- **Inngest queue** gives smooth background processing and reliable orchestration.
- **AWS S3 integration** for song storage, streaming, and thumbnail delivery.

---

## 🏗️ System Architecture

```text
User → Next.js UI (App Router) → Server Actions → Prisma/Postgres (Neon)
                               │
                               ▼
                         Inngest Queue
                               │      (Modal webhook)
                               ▼
                       Modal GPU Backend (FastAPI)
            ┌──────────────┬───────────────┬───────────────┐
            │   ACE-Step   │  Qwen2-7B LLM │   SDXL Turbo  │
            └──────────────┴───────────────┴───────────────┘
                               │
                               ▼
                            AWS S3
                               │
                               ▼
                       Song Feed & Player
````

**Components**

* **Frontend**: Next.js 15 (App Router), Tailwind, shadcn/ui, TanStack Query.
* **Backend**: Python + FastAPI via Modal’s serverless GPU runtime.
* **Data**: Neon Postgres (Prisma ORM), AWS S3 for assets.
* **Automation**: Inngest for job orchestration; Polar webhooks manage credits.

---

## 📁 Directory Structure

```
music-generator/
├─ frontend/                # Next.js 15 app (App Router)
│  ├─ app/
│  ├─ components/
│  ├─ lib/                  # auth, api clients, utils
│  ├─ inngest/              # event handlers & schedules
│  ├─ prisma/               # schema.prisma & migrations
│  ├─ public/               # static assets
│  └─ .env.example
├─ backend/                 # Modal + FastAPI serverless functions
│  ├─ main.py               # entrypoints / @app.function
│  ├─ ace_step/             # ACE-Step integration
│  ├─ sd/                   # SDXL Turbo image gen helpers
│  ├─ qwen/                 # Qwen2-7B prompt/lyrics utilities
│  └─ requirements.txt
├─ infra/                   # IaC, policies, scripts
│  ├─ s3-policy.json
│  └─ scripts/
├─ .github/workflows/       # CI pipelines
├─ README.md
└─ LICENSE.MD
```

---

## 🧰 Tech Stack

| Layer          | Tools / Services                                               |
| -------------- | -------------------------------------------------------------- |
| Frontend       | Next.js 15, React 18, TypeScript, Tailwind CSS, shadcn/ui      |
| Auth & Billing | BetterAuth, Polar checkout & customer portal                   |
| Backend        | Python 3.12, FastAPI, Modal serverless functions               |
| AI Models      | ACE-Step (music diffusion), Qwen2-7B-Instruct, SDXL Turbo      |
| Orchestration  | Inngest event-driven jobs                                      |
| Storage        | AWS S3 for audio & thumbnails, Neon/Postgres for relational DB |
| Tooling        | Prisma ORM, ESLint, Prettier, Modal CLI, Inngest CLI           |

---

## 🚀 Quick Start

### 1) Clone

```bash
git clone --recurse-submodules https://github.com/sh-arka22/music-generator.git
cd music-generator
```

### 2) Backend (Modal)

```bash
cd backend
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# One-time auth & config
modal setup

# Local test
modal run main.py

# Deploy API endpoints
modal deploy main.py
```

> Modal automatically clones ACE-Step and caches models using named volumes (`ace-step-models`, `qwen-hf-cache`).

### 3) Frontend (Next.js)

```bash
cd ../frontend
npm install
```

Create `.env` from the schema below (see **Environment Variables**). Then:

```bash
npm run dev
```

### 4) Queue Worker (Inngest)

```bash
# From the frontend workspace
npx inngest-cli@latest dev
```

---

## ⚙️ Configuration

### Environment Variables

Create `frontend/.env` with:

```bash
# Auth
BETTER_AUTH_SECRET=

# Database
DATABASE_URL=                             # Neon connection string

# Modal
MODAL_KEY=
MODAL_SECRET=

# AWS S3
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=
S3_BUCKET_NAME=

# Modal HTTPS endpoints (from your modal deploy)
GENERATE_FROM_DESCRIPTION=
GENERATE_WITH_LYRICS=
GENERATE_FROM_DESCRIBED_LYRICS=

# Billing (Polar)
POLAR_ACCESS_TOKEN=
POLAR_WEBHOOK_SECRET=
```

> Tip: keep separate `.env` for `development`, `staging`, and `production`.

### AWS S3 Setup

* Create two buckets (recommended): one for **audio** and one for **thumbnails**.
* Enable CORS for GET/PUT (origin = your frontend domain).
* Apply a minimal bucket policy (tweak ARNs and principals as needed):

`infra/s3-policy.json` (example)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListBucket",
      "Effect": "Allow",
      "Principal": {"AWS": "*"},
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME",
      "Condition": {
        "StringLike": {"s3:prefix": ["public/*"]}
      }
    },
    {
      "Sid": "GetPublicObjects",
      "Effect": "Allow",
      "Principal": {"AWS": "*"},
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/public/*"
    }
  ]
}
```

> For private objects (uploads from Modal), prefer signed URLs generated server-side.

### Database (Neon + Prisma)

* Provision a Neon Postgres database.
* Set `DATABASE_URL`.
* Run migrations:

```bash
cd frontend
npx prisma generate
npx prisma migrate dev   # or `npx prisma migrate deploy` in CI/CD
```

> Consider separate branches/projects per environment in Neon.

### Modal Setup

```bash
# Authenticate and set up profile
modal setup

# Deploy your FastAPI endpoints
modal deploy backend/main.py

# Optional: set named volumes for model caches in code or in the modal config
```

### Inngest Setup

* Local dev:

```bash
npx inngest-cli@latest dev
```

* Cloud deployment (optional):

```bash
npx inngest-cli@latest deploy
```

* Configure webhooks/secrets in Inngest to talk to your Modal endpoints & your frontend server actions.

---

## 🧠 Generation Modes & API

### Endpoints

| Endpoint                         | Description                                                     | Body (JSON)                                                               |
| -------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `generate_from_description`      | Turns a free-form description into prompts, lyrics, categories  | `{ "description": "cinematic synthwave, 120bpm, minor", "duration": 30 }` |
| `generate_with_lyrics`           | Uses provided lyrics + prompt                                   | `{ "prompt": "lofi chill", "lyrics": "…" }`                               |
| `generate_from_described_lyrics` | Expands described-lyrics to full lyrics, then uses given prompt | `{ "description": "power ballad about hope", "prompt": "arena rock" }`    |
| `generate` (demo)                | Test endpoint returning base64 audio for quick validation       | `{ "seed": 42 }`                                                          |

> All endpoints are exposed by the Modal FastAPI backend and consumed by the frontend server actions & the Inngest worker.

### cURL Examples

```bash
# Replace with your deployed Modal URL
MODAL_URL="https://<your-modal-app>.modal.run"

curl -X POST "$MODAL_URL/generate_from_description" \
  -H "Content-Type: application/json" \
  -d '{"description":"cinematic synthwave, 120bpm, minor","duration":30}'
```

---

## 🔄 Processing Pipeline

1. User submits a generation request from the `/create` UI.
2. Two song jobs (guidance scales **7.5** & **15**) are created in Postgres.
3. Inngest event `generate-song-event` enqueues background work.
4. Worker checks credits, selects the Modal endpoint, and calls it.
5. Modal backend:

   * (If needed) generates prompt/lyrics via **Qwen2-7B**,
   * runs **ACE-Step** diffusion for audio,
   * produces **SDXL** artwork,
   * uploads assets to **S3**,
   * auto-tags categories.
6. Worker updates song status, stores S3 keys, connects categories, deducts credits.
7. Frontend revalidates and displays the track in dashboards and community feed.

**Artifacts per run**

* `.wav` uploaded to S3 (`s3_key`)
* PNG cover image via SDXL (`cover_image_s3_key`)
* Auto-categorized tags driven by Qwen output

---

## 📡 Deployment & Ops

* **Modal**: GPU-backed Python functions, scalable via `modal deploy`.
* **Inngest Cloud (optional)**: `inngest deploy` to host the queue remotely.
* **Database**: Neon production branch with Prisma migrations (`npx prisma migrate`).
* **S3 Buckets**: Separate for audio & thumbnails; lock down with signed URLs where needed.
* **CI/CD**:

  * GitHub Actions for linting/building both frontend and backend.
  * Suggested jobs:

    * `frontend-lint`: `npm ci && npm run lint`
    * `frontend-build`: `npm run build`
    * `backend-lint`: `ruff` or `flake8`
    * `backend-deploy`: `modal deploy backend/main.py`
    * `prisma-migrate-deploy`: `npx prisma migrate deploy`

---

## 🔐 Security & Privacy

* Store secrets in platform key vaults (GitHub Actions Secrets, Vercel/Netlify env vars).
* Prefer **signed URLs** for private S3 assets.
* Validate & sanitize all user inputs; enforce file size/type limits on uploads.
* Rate-limit generation endpoints; protect with authenticated server actions.
* Log PII-free operational metrics only.

---

## 🛠️ Troubleshooting

**Modal cannot find app / deprecation warnings**

* Ensure entrypoint exports the Modal app and functions correctly.
* Update deprecated parameters per latest Modal docs.

**Model downloads are slow**

* Use named volumes for Hugging Face/ACE-Step caches.
* Warm caches with a one-time “dry run” job.

**S3 access denied**

* Check IAM permissions and bucket policies.
* Verify region and bucket names match your env.

**Inngest events not firing**

* Ensure Inngest CLI is running locally or deployed in Cloud.
* Check webhook URL & secrets configuration.

**Prisma schema mismatch**

* Run `npx prisma generate` and `npx prisma migrate dev` after schema edits.

---

## ❓ FAQ

**Q: Can I change the diffusion model?**
A: Yes—wrap another audio model with the same interface the ACE-Step runner expects.

**Q: How are credits deducted?**
A: The worker validates balance pre-run, decrements on successful completion (adjust logic to refund on failure as needed).

**Q: Do I need separate S3 buckets?**
A: Recommended (audio vs thumbnails) for cleaner policies and lifecycle rules, but not strictly required.

**Q: How big are generated files?**
A: Depends on duration/sample rate; consider 44.1kHz mono `.wav` for predictable sizes. Enable lifecycle policies to transition to infrequent access.

---

## 🧭 Roadmap

* User-adjustable duration, tempo, and voice options.
* Waveform previews & shareable links.
* Improved moderation & quality filters for generated lyrics.
* Playlisting and collaborative sessions.
* Multi-track stems export.

---

## 🤝 Contributing

1. **Fork & clone** the repo.
2. Create a feature branch: `git checkout -b feat/amazing-idea`.
3. Run formatters before committing:

   * Frontend: `npm run lint`
   * Backend: `python -m black .` (or `ruff`)
4. Add/Update tests where relevant.
5. Open a PR and tag **@sh-arka22**.

**Commit style**: Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`, etc.)

---

## 🧑‍💻 Maintainer

| Name            | GitHub                                                |
| --------------- | ----------------------------------------------------- |
| Arka Jyoti Saha | <a href="https://github.com/sh-arka22">@sh-arka22</a> |

Open issues or reach out via GitHub Discussions for feature requests, bug reports, or collaboration ideas.

---

## License

This project is licensed under the **MIT License** — see [LICENSE.MD](./LICENSE.MD).

---

<p align="center">
  <em>“Creating original music with code, one beat at a time.”</em>
</p>
```
