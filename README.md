# LMS Admin Portal

Internal admin tool for managing LearnUpon LMS users, enrollments, learning journeys, and groups at EBLI.

## Quick Start

```powershell
# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Fill in values (see .env.example for descriptions)

# Run dev server (corporate proxy requires TLS override)
$env:NODE_TLS_REJECT_UNAUTHORIZED="0"; npm run dev
```

## Deployment

Deployed on Vercel (Hobby plan, Fluid Compute) at ebli-lms-admin.vercel.app. Push to `main` to deploy.

## Architecture

See `.kiro/steering/` for detailed architecture documentation:
- `product.md` — what the app does and who uses it
- `tech.md` — stack, patterns, file structure
- `api-conventions.md` — rules for adding new API calls and mutations

## Key Design Decisions

- **Request Pipeline**: All outbound LMS calls flow through `src/lib/request-pipeline.ts` which provides concurrency limiting, adaptive throttling, retry, circuit breaking, and deduplication.
- **Two-layer caching**: Server-side (journey metadata, 24h) + client-side (progress data, 5min) to minimize API calls.
- **Authentication**: Google OAuth (@ebli.com only) + LearnUpon session credentials (encrypted in HttpOnly cookie).
- **RBAC**: VIEWER/OPERATOR/ADMIN roles with server-side enforcement on every route.
