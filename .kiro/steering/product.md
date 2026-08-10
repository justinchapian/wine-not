---
inclusion: always
---
# Product — LMS Admin Portal

> **Content type: Knowledge reference.** This file describes what the app is, who uses it, and its key constraints. It contains no auditable rules — consult for planning context when understanding the product scope, user populations, or external dependencies.

## What it is
Internal admin web app for managing LearnUpon LMS users, enrollments, learning journeys, and groups. Deployed on Vercel (Hobby plan, Fluid Compute enabled) at ebli-lms-admin.vercel.app.

## Who uses it
- Small team of admins at EBLI (~3-10 people), all with @ebli.com Google accounts
- Primary use: look up learners, check their learning journey progress, manage enrollments, monitor engagement
- RBAC: VIEWER (read-only), OPERATOR (future), ADMIN (mutations + delete)

## Two user populations

- **App users (admins):** @ebli.com Google accounts. Authenticated via NextAuth. These are the people operating this web app.
- **LMS users (learners):** People who take courses in LearnUpon. Their data is what admins view and manage in this app. Learners can have any email address — they are NOT restricted to @ebli.com.

## Key features
- User search with cached all-users list (3000 users, 24h localStorage cache)
- Enrollment management (view, delete, mark complete, bulk actions)
- Learning journey progress visualization: segmented status bar (per-course colored rectangles with crossover/admin indicators) + engagement timeline (dot-based proportional axis)
- User engagement graph: multi-row timeline showing completion activity across ALL learning journeys in one view, with last-login indicator and "View Entire History" modal (dot mode or histogram for long histories)
- Course completion time box plot: horizontal box and whisker showing distribution of organic completion durations (excludes admin completions)
- Group management (add/remove/switch groups)
- Multi-user bulk progress report (Excel download with sequential API runner)
- Engagement badges (Active/Slowing/Stalled/Stopped based on last completion date)
- Engagement attribution: distinguishes "journey-enrolled" completions from "crossover" completions (courses completed via another overlapping journey). Engagement level is computed from journey-enrolled completions only, preventing crossover courses from inflating activity metrics.
- Portal switcher: toggle between Production and Sandbox LearnUpon portals from the nav bar (ADMIN only). Clears all caches and LMS session on switch. Sandbox mode shown with amber indicator to prevent accidental production mutations.

## External dependencies
- **LearnUpon LMS** — Two portals:
  - Production: public API on `ebli.learnupon.com`, internal/session on `eblireads.learnupon.com` (different hostnames!)
  - Sandbox: `eblisandbox.learnupon.com` (same hostname for both API and internal)
  - Two auth methods per portal:
    - Public API (Basic Auth from env vars): users, enrollments, groups, courses
    - Internal session API (cookie-based): learning journeys (no public API for these)
- **Google OAuth** via NextAuth for admin authentication
- **Vercel**: Hosting, serverless functions (300s max with Fluid Compute)

## Planned enhancements
See `docs/ROADMAP.md` for planned features and tech debt.

## Key constraints
- LearnUpon has per-portal rate limits (per-minute and per-week). All outbound calls flow through a request pipeline with adaptive throttling.
- Vercel Hobby plan: 100k function invocations/month, 300s max duration per function
- All changes stay local until explicitly pushed (user controls when to commit/deploy)
- Corporate proxy requires `NODE_TLS_REJECT_UNAUTHORIZED="0"` for local dev
