# Music Generator SaaS

> A full‑stack AI music creation platform: generates original songs from descriptions, prompts, or lyrics with GPU‑backed inference, LLM‑assisted songwriting, and a polished Next.js SaaS experience.

---

## ✨ Features

- Music synthesis with ACE-Step diffusion (text → full track).
- LLM‑assisted prompts, lyrics, and auto‑categorization via Qwen2‑7B-Instruct.
- Album artwork generation with SDXL Turbo.
- Modal serverless GPU for scalable, production‑ready inference.
- Credit‑based billing using Polar checkout & customer portal.
- Background processing with Inngest queues, per‑user concurrency control.
- S3 storage for audio/cover images and secure presigned playback.
- Auth, sessions, and Prisma/Postgres data model with BetterAuth.
- Community feed, likes, categories, and personal dashboard.

---

## 🧭 Architecture

    User → Next.js UI → Server Actions → Prisma/Postgres
    ↓
    Inngest Queue
    ↓ (HTTP fetch with Modal keys)
    Modal GPU Backend (FastAPI)
    ┌──────────────┬───────────────┬───────────────┐
    │ ACE-Step │ Qwen2-7B LLM │ SDXL Turbo │
    └──────────────┴───────────────┴───────────────┘
    ↓
    AWS S3 (Audio + Artwork)
    ↓
    Feed, Player, Dashboard

---
- Frontend: Next.js 15 (App Router), Tailwind CSS, shadcn/ui.
- Backend: Python + FastAPI on Modal (GPU: `L40S`).
- Models: ACE-Step (music), Qwen2-7B (LLM), SDXL Turbo (image).
- Orchestration: Inngest events with per‑user concurrency limit = 1.
- Data: Prisma ORM + Postgres (Neon or local Docker).
- Storage: AWS S3 for audio `.wav` and thumbnails `.png`.

---

## 🧰 Tech Stack

| Layer          | Tools / Services                                               |
| -------------- | -------------------------------------------------------------- |
| Frontend       | Next.js 15, React 18, TypeScript, Tailwind CSS, shadcn/ui      |
| Auth & Billing | BetterAuth, Polar (checkout, portal, webhooks)                 |
| Backend        | Python 3.12, FastAPI, Modal serverless GPU                     |
| AI Models      | ACE-Step, Qwen2-7B-Instruct, SDXL Turbo                        |
| Orchestration  | Inngest (event-driven jobs)                                    |
| Storage        | AWS S3 (audio, thumbnails), Prisma/Postgres (Neon or Docker)   |
| Tooling        | Prisma, ESLint/Prettier, Modal CLI, Inngest CLI                |

---

## 📂 Repository Layout

    backend/
    main.py # FastAPI endpoints on Modal; loads models; S3 uploads
    prompts.py # LLM prompt templates for prompts/lyrics
    requirements.txt # Python deps (diffusers, transformers, boto3, modal, etc.)
    frontend/
    prisma/schema.prisma # Postgres models (User, Song, Category, Like, etc.)
    src/
    app/ # Next.js routes (auth, main feed, create, etc.)
    actions/ # Server actions (generate, play, presigned URLs)
    inngest/ # Inngest client + job function (generate-song)
    lib/auth.ts # BetterAuth + Polar integration (credits)
    env.js # Env validation schema
    start-database.sh # Local Postgres Docker helper (optional)
    README.md
    LICENSE.MD


---

## 🚀 Quick Start

### 1) Clone

    git clone --recurse-submodules https://github.com/sh-arka22/music-generator.git
    cd music-generator

2) Backend (Modal):


            Prereqs: Python 3.12+, Modal CLI (pip install modal), an AWS IAM user with S3 access.
           cd backend
        python3 -m venv .venv && source .venv/bin/activate
        pip install -r requirements.txt
        modal setup
        Set up Modal secrets (used by @app.cls(..., secrets=[music_gen_secrets]) in backend/main.py):
        
        modal secret create music-gen-secret \
          S3_BUCKET_NAME=your-s3-bucket-name \
          AWS_ACCESS_KEY_ID=... \
          AWS_SECRET_ACCESS_KEY=... \
          AWS_REGION=us-east-1
        
        Test locally on Modal:
        
        modal run main.py
        Deploy endpoints (returns HTTPS URLs in the Modal dashboard):
        
        modal deploy main.py
        Notes:
        
        Models cached to named volumes: ace-step-models (ACE-Step), qwen-hf-cache (Hugging Face).
        The backend clones ACE-Step during image build and installs it:
        git clone https://github.com/ace-step/ACE-Step.git /tmp/ACE-Step && pip install .
5) Frontend (Next.js)
        Prereqs: Node 18+, npm.
        
        cd frontend
        cp .env.example .env   # create if you don't have one; see below
        npm install
        npm run dev
        Populate .env per src/env.js (server env). Required keys:
        
        BETTER_AUTH_SECRET=                       # random 32+ chars
        DATABASE_URL=postgresql://user:pass@host:port/db
        NODE_ENV=development
        
        # Modal auth for calling backend endpoints
        MODAL_KEY=...
        MODAL_SECRET=...
        
        # AWS access for presigned URL generation (frontend server only)
        AWS_ACCESS_KEY_ID=...
        AWS_SECRET_ACCESS_KEY_ID=...
        AWS_REGION=us-east-1
        S3_BUCKET_NAME=your-s3-bucket-name
        
        # Modal FastAPI endpoints from modal deploy output
        GENERATE_FROM_DESCRIPTION=https://<modal-url>/generate_from_description
        GENERATE_FROM_DESCRIBED_LYRICS=https://<modal-url>/generate_with_described_lyrics
        GENERATE_WITH_LYRICS=https://<modal-url>/generate_with_lyrics
        
        # Polar billing
        POLAR_ACCESS_TOKEN=...
        POLAR_WEBHOOK_SECRET=...
        
        
        # In frontend/
        ./start-database.sh   # reads DATABASE_URL in .env and starts Docker postgres
        npx prisma migrate dev


        cd frontend
        npx inngest-cli@latest dev



🔌 API (Modal Backend)
    All endpoints require headers:
    
    Content-Type: application/json
    Modal-Key: ${MODAL_KEY}
    Modal-Secret: ${MODAL_SECRET}
    Shared defaults (backend AudioGenerationBase):
    
    audio_duration: float = 180.0
    seed: int = -1
    guidance_scale: float = 15.0
    infer_step: int = 60
    instrumental: bool = False
    1) POST /generate_from_description
    
    
    1) POST /generate_from_description
    Body:
    
    {
      "full_described_song": "an 80s-inspired synthwave ballad about neon lights",
      "audio_duration": 180,
      "seed": -1,
      "guidance_scale": 15,
      "infer_step": 60,
      "instrumental": false
    }
    Response:
    
    {
      "s3_key": "generated-audio.wav",
      "cover_image_s3_key": "cover.png",
      "categories": ["Synthwave", "80s", "Electronic"]
    }
    Behavior:
    
    Qwen reformats the description into ACE-Step prompt tags.
    Qwen generates full lyrics (unless instrumental is true).
    ACE-Step synthesizes audio to .wav.
    SDXL Turbo generates cover art.
    Both are uploaded to S3; categories auto‑extracted by Qwen.
    
    2) POST /generate_with_lyrics
    Body:
    
    {
      "prompt": "melodic techno, 124 bpm, minor key, synthesizer, driving",
      "lyrics": "[verse]\n...\n[chorus]\n...",
      "audio_duration": 180,
      "guidance_scale": 15,
      "infer_step": 60,
      "instrumental": false
    }
    Response: same as above (S3 keys + categories). Categories inferred from prompt.
    
    3) POST /generate_with_described_lyrics
    Body:
    
    {
      "prompt": "rave, funk, 140 bpm, disco",
      "described_lyrics": "lyrics about water bottles",
      "audio_duration": 180,
      "guidance_scale": 15,
      "infer_step": 60,
      "instrumental": false
    }
    Behavior:
    
    Qwen expands described_lyrics into fully structured lyrics.
    ACE-Step + SDXL as above.
    
    
    4) POST /generate (demo)
    Returns base64‑encoded audio (quick sanity check).
    
    🧠 Generation Modes (UI)
    The UI will create two jobs per request with guidance scales 7.5 and 15 to offer variety. Inngest enforces a one‑at‑a‑time concurrency per user to prevent overload.
    
    Custom (Prompt + Lyrics)
    Prompt + Described Lyrics (LLM expands)
    Full Description (LLM derives both prompt and lyrics)
    Instrumental toggle forces [instrumental] lyrics
    📦 Data Model (Prisma)
    Key models:
    
    User: credits balance, sessions, songs, likes
    Song: status (queued → processing → processed or failed), metadata (prompt/lyrics), S3 keys, categories, listen count
    Category: names connected via connectOrCreate
    Like: user-song pairs
    See: frontend/prisma/schema.prisma



☁️ AWS Setup
    Frontend user policy (example):
    
    {
      "Version": "2012-10-17",
      "Statement": [
        { "Effect": "Allow", "Action": ["s3:PutObject", "s3:GetObject"], "Resource": "arn:aws:s3:::your-s3-bucket-name/*" },
        { "Effect": "Allow", "Action": "s3:ListBucket", "Resource": "arn:aws:s3:::your-s3-bucket-name" }
      ]
    }
    Backend user policy (example):
    
    {
      "Version": "2012-10-17",
      "Statement": [
        { "Effect": "Allow", "Action": "s3:GetObject", "Resource": "arn:aws:s3:::your-s3-bucket-name/*" },
        { "Effect": "Allow", "Action": "s3:ListBucket", "Resource": "arn:aws:s3:::your-s3-bucket-name" }
      ]
    }
    Replace your-s3-bucket-name accordingly. Set S3_BUCKET_NAME in Modal secret and frontend .env.

💳 Billing & Credits
    BetterAuth + Polar plugin manage customers, checkout, and webhooks.
    On successful purchase, credits are incremented based on product ID.
    Each successful generation deducts 1 credit.
    See: frontend/src/lib/auth.ts

🧪 Development Tips
        Modal:
        GPU type configured at backend/main.py: @app.cls(gpu="L40S").
        If GPU quota is unavailable, change to an available GPU or region.
        HuggingFace cache:
        Cached into /.cache/huggingface (mounted volume qwen-hf-cache).
        ACE-Step pipeline:
        Initialized with checkpoint_dir="/models" (mounted volume ace-step-models).
        Prisma:
        npx prisma generate and npx prisma migrate dev as needed.
        Inngest:
        Local dev server echoes logs and failures; production deploy optional.

🐛 Troubleshooting
        403/AccessDenied on S3:
        Verify bucket name, region, IAM policies, and Modal secret values.
        401 calling Modal endpoints:
        Ensure MODAL_KEY and MODAL_SECRET are correct and passed in headers.
        Modal deployment succeeds but models fail to load:
        Check GPU availability; ensure volumes are named exactly as in code.
        Inngest job stuck in “queued”:
        Run npx inngest-cli@latest dev locally; check logs and endpoint URLs.
        Database connection errors:
        Validate DATABASE_URL; if using local Docker, ensure port isn’t taken. Use ./start-database.sh.
        🗺️ Roadmap
        User controls for duration, tempo, voice/style presets.
        Waveform previews, social sharing, and embeds.
        Lyrics moderation and quality filters.
        Playlists, follows, and collaborative sessions.
        Export stems and multitrack mixing.
🤝 Contributing
        Fork and clone the repo.
        Create a feature branch: git checkout -b feat/amazing-idea.
        Keep changes focused; run formatters and type checks.
        Open a PR and tag @sh-arka22.

📝 License
        MIT — see @LICENSE.MD.

🙏 Acknowledgements
        ACE-Step authors and contributors
        Qwen team for Qwen2‑7B‑Instruct
        Stability AI for SDXL Turbo
        Modal, Inngest, Prisma, Neon, BetterAuth, and Polar for excellent tooling


