---
title: Platform Cache Architecture
intent: architecture-reference
area: docs
---
# Platform Cache Architecture

> **Content type: Knowledge reference.** This is an architecture reference document for the caching layer. The one cacheable rule lives in `api-conventions.md` RULE: "Consider cacheability for new LMS data." This document provides the implementation details and context.

## Overview

The app uses a server-side in-memory cache (`src/lib/platform-cache.ts`) to eliminate redundant API calls to LearnUpon for data that rarely changes. This speeds up the app and reduces load on the LMS.

## How It Works

**Pattern: Cache-through**

Every cached data type has a function like `getAllGroupsCached(fetchFn)`. When called:

1. Check if data exists in memory and is within TTL → return instantly (no API call)
2. If not → call the provided fetch function (which hits the LMS)
3. Store the result in memory with a timestamp
4. Return the data

This means: the first request that needs the data pays the cost of the API call. All subsequent requests return from memory in microseconds.

## What's Cached

| Data | Cache Key | Used By |
|------|-----------|---------|
| All learning journeys (id, versionId, name) | `journey-list` | `/api/learning-journeys/list`, `/progress`, `/user` |
| Course tree per journey (ordered IDs + names) | `courses:{journeyId}:{versionId}` | `/api/learning-journeys/courses`, `/progress`, `/switch-preview` |
| All groups (id, title, description, memberCount) | `groups-all` | `/api/groups` |
| Group triggers (journeys triggered by group) | `group-triggers:{groupId}` | `/api/learning-journeys/group-triggers`, `/switch-preview` |

## What's NOT Cached (Always Live)

- **Enrollment data** — user-specific, changes with every enrollment/completion/deletion
- **User search results** — user-specific
- **User group memberships** — user-specific
- **Learning journey enrollment state per user** — the `/users/states.json` calls that check if a specific user is enrolled in a journey

**Principle:** Anything the app can manipulate (enrollments, group memberships) fetches live. Everything that only changes when an admin edits LearnUpon directly (groups, journeys, course structure) is cached. See `api-conventions.md` RULE: "Consider cacheability for new LMS data" for the enforceable version.

## TTL and Instance Lifetime

**TTL is set to 24 hours**, but the practical cache lifetime is shorter.

On Vercel serverless, in-memory data only lives as long as the function instance stays warm. Instances go cold after a period of inactivity (typically minutes). When a new instance starts, the cache is empty and the first request re-fetches everything.

**What this means in practice:**
- During an active admin session (30 min of clicking around), the cache stays warm. The first page load fetches from LMS; all subsequent ones are instant.
- If no one uses the app for a while, the instance goes cold. The next admin's first request takes a few extra seconds while it repopulates.
- The 24h TTL is a safety ceiling — "even if the instance lives forever, don't serve data older than 24 hours."

## Cache Invalidation

### Manual (Admin UI)
- `POST /api/cache/invalidate` — clears ALL cached platform data (ADMIN only)
- Use when: you've made structural changes in LearnUpon (created/deleted groups, edited journey structure, added courses to a journey)

### Automatic
- Data expires after 24h TTL (if the instance is still warm)
- Instance cold start naturally clears all cache

### Monitoring
- `GET /api/cache/invalidate` — returns current cache statistics (ADMIN only)
- Reports: total entries, whether journey list is cached, how many course trees and group triggers are cached

## Adding New Cached Data

To cache a new type of platform-wide data:

1. Add a typed interface and cache function to `src/lib/platform-cache.ts` following the existing pattern
2. In the API route, wrap the fetch call: `const data = await myNewCacheFn(existingFetchFunction)`
3. Add the new key to `getCacheStats()` for monitoring
4. Consider whether `invalidateAll()` is sufficient or if you need a targeted invalidation function

## Design Decisions

- **In-memory over Redis/KV:** The data volume is tiny (hundreds of items), the user count is small, and adding external infrastructure for a serverless admin tool is unnecessary complexity.
- **Cache-through over pre-warming:** No cron job needed. The first request absorbs the fetch cost, which is acceptable for an admin tool.
- **Platform-wide, not per-user:** Groups, journeys, and course structures are the same regardless of which admin is logged in. One cache fill benefits all users.
- **24h TTL ceiling:** Matches the "daily refresh" concept. In practice, cold starts are the real expiry mechanism.
