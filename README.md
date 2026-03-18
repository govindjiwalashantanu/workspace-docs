# Workspace Context

This repo is the single source of truth for all context, architecture, protocols, and plans across active projects. Pull it down on any machine to resume work with full context.

---

## Projects

| Project | Repo | Stack | Status |
|---|---|---|---|
| **sensei-webapp** | `govindjiwalashantanu/sensei-webapp` | Next.js 16, Prisma, PostgreSQL | Active |
| **echo-backend** | `govindjiwalashantanu/echo-backend` | Next.js 16, Prisma, PostgreSQL | Active |
| **echo-mobile** | `govindjiwalashantanu/echo-mobile` | Expo, React Native | Active |

---

## What's in here

```
workspace-docs/
├── WORKING_PROTOCOL.md        # Rules that apply to all projects — read first every session
├── sensei-webapp/
│   ├── CLAUDE.md              # Architecture, deployment (EC2), env vars, key patterns
│   ├── ROADMAP.md             # Feature roadmap — completed, in-progress, backlog
│   └── SESSION_NOTES.md       # Session history — what was built, decisions made, next steps
├── echo-backend/
│   ├── CLAUDE.md              # Architecture, env vars, billing, API patterns
│   ├── ROADMAP.md             # Feature roadmap
│   ├── SESSION_NOTES.md       # Session history
│   ├── PRODUCT_PLAN.md        # Product strategy, tiers, positioning
│   ├── PRICING_ADVISOR.md     # Pricing model, margin analysis, regional pricing
│   ├── BUSINESS_CASE.md       # Business case documentation
│   └── ECHO_PATENTS.md        # Patent considerations
└── echo-mobile/
    └── CLAUDE.md              # Mobile architecture, key patterns
```

**Rule:** Every Claude Code session must update the relevant files in this repo (and in the project repo) before closing. Never let these go stale.

---

## New Machine Setup

### 1. Prerequisites

```bash
# Node.js 20+
brew install node

# Or via nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20
nvm use 20

# Git
brew install git

# GitHub CLI
brew install gh
gh auth login
```

### 2. Install Claude Code

```bash
npm install -g @anthropic/claude-code
```

### 3. Clone this repo first

```bash
git clone https://github.com/govindjiwalashantanu/workspace-docs.git ~/Documents/workspace-docs
```

Read `WORKING_PROTOCOL.md` before anything else.

### 4. Clone the project repos

```bash
# All repos into ~/Documents/ (matches the paths in CLAUDE.md files)
git clone https://github.com/govindjiwalashantanu/sensei-webapp.git ~/Documents/sensei-webapp
git clone https://github.com/govindjiwalashantanu/echo-backend.git ~/Documents/echo-backend
git clone https://github.com/govindjiwalashantanu/echo-mobile.git ~/Documents/echo-mobile
```

### 5. Install dependencies

```bash
cd ~/Documents/sensei-webapp && npm install
cd ~/Documents/echo-backend  && npm install
cd ~/Documents/echo-mobile   && npm install
```

### 6. Environment variables

Each project needs a `.env.local` file. These are **not** in git. Get the values from:
- The EC2 instance: `cat /var/www/sensei-webapp/.env.local`
- 1Password / secure notes
- AWS Secrets Manager (if set up)

**sensei-webapp** needs:
```
DATABASE_URL
DIRECT_URL
NEXTAUTH_URL
NEXTAUTH_SECRET
OKTA_CLIENT_ID / OKTA_CLIENT_SECRET / OKTA_ISSUER
GOOGLE_CLIENT_ID / GOOGLE_CLIENT_SECRET
RESEND_API_KEY
EMAIL_FROM
LITELLM_API_KEY / LITELLM_BASE_URL / LITELLM_MODEL
GROQ_API_KEY
ENCRYPTION_KEY
AGENT_CRON_SECRET
```

**echo-backend** needs:
```
DATABASE_URL
DIRECT_URL
NEXTAUTH_URL
NEXTAUTH_SECRET
OKTA_CLIENT_ID / OKTA_CLIENT_SECRET / OKTA_ISSUER
GOOGLE_CLIENT_ID / GOOGLE_CLIENT_SECRET
RESEND_API_KEY
EMAIL_FROM
LITELLM_API_KEY / LITELLM_BASE_URL / LITELLM_MODEL
GROQ_API_KEY
ENCRYPTION_KEY
STRIPE_SECRET_KEY / STRIPE_WEBHOOK_SECRET (if billing active)
REVENUECAT_WEBHOOK_SECRET
DEEPGRAM_API_KEY (optional)
ASSEMBLYAI_API_KEY (optional)
```

**echo-mobile** needs:
```
EXPO_PUBLIC_API_URL   # e.g. https://echo.your-domain.com
```

### 7. Database setup

```bash
# sensei-webapp
cd ~/Documents/sensei-webapp && npx prisma generate && npx prisma db push

# echo-backend
cd ~/Documents/echo-backend && npx prisma generate && npx prisma db push
```

### 8. Run locally

```bash
# sensei-webapp
cd ~/Documents/sensei-webapp && npm run dev      # http://localhost:3000

# echo-backend
cd ~/Documents/echo-backend && npm run dev       # http://localhost:3000

# echo-mobile
cd ~/Documents/echo-mobile && npm start          # Expo dev server
```

---

## Starting a Claude Code Session

1. Pull this repo: `git pull` in `~/Documents/workspace-docs`
2. Open the project in your terminal
3. Run `claude` to start Claude Code
4. Claude will read the project's `CLAUDE.md` automatically
5. For cross-project context, paste the relevant section from this repo

**At the end of every session:**
- Update `SESSION_NOTES.md` in the project repo
- Update `SESSION_NOTES.md` in this repo
- Update `ROADMAP.md` if features shipped
- Commit and push both repos

---

## EC2 Production (sensei-webapp)

- **URL:** https://okta.se-n-sei.com
- **Instance:** t3.medium, us-east-1
- **SSH:** `ssh -i ~/.ssh/sensei-ec2-key.pem ubuntu@<ELASTIC_IP>`
- **Deploy:** push to `main` → GitHub Actions auto-deploys via SSH

See `sensei-webapp/CLAUDE.md` for full deployment architecture.

---

## Key Decisions (permanent record)

| Decision | Rationale |
|---|---|
| sensei-webapp is an internal Okta enterprise app | No public sign-up, no GDPR data export needed |
| Echo uses Groq Turbo for all tiers in v1 | Healthy margins globally; Deepgram/AssemblyAI activate automatically when keys added |
| SSE over WebSocket for AI job tracker | Single EC2 instance — in-process EventEmitter works perfectly, no Redis needed |
| Echo multi-language: `['en','hi']` → `lang=auto` | Whisper auto-detects per recording when user speaks multiple languages |
| sensei AI rate limits: generous (200/hr) | Goes through Okta's internal LiteLLM proxy |
| No fire-and-forget for AI jobs (v1) | React Query mutations continue after unmount; same-tab navigation works fine |
