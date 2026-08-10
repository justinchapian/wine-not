---
inclusion: always
---
# Deployment & npm Safety

---

## Rules

### RULE: Use public npm registry for all package operations
- **Trigger:** Running `npm install`, `npm update`, or any command that modifies `package-lock.json`
- **Check:** Does the command include `--registry=https://registry.npmjs.org`?
- **Instruction:** Always append `--registry=https://registry.npmjs.org` to the command. After the command completes, scan `package-lock.json` for "artifactory" references. If any are found, replace all occurrences of `https://artifactory.spectrumtoolbox.com/artifactory/api/npm/npm/` with `https://registry.npmjs.org/` before committing. Warn the user if Artifactory URLs were detected so they know the lockfile was sanitized.

### RULE: Remind user about registry flag for manual installs
- **Trigger:** The user mentions they are installing packages manually (outside of agent-assisted work)
- **Check:** Did you remind them to use the `--registry` flag?
- **Instruction:** Remind them to use: `npm install <package> --registry=https://registry.npmjs.org`

### RULE: Pre-push checklist — verify before pushing to remote
- **Trigger:** About to push code to GitHub (or recommending the user push)
- **Check:** Have all of the following been verified?
  1. No Artifactory URLs in `package-lock.json`
  2. No `.npmrc` file committed (it's in `.gitignore`)
  3. Environment variables set in Vercel dashboard (not in committed files)
  4. Sandbox env vars (`LMS_SANDBOX_PORTAL_URL`, `LMS_SANDBOX_API_USERNAME`, `LMS_SANDBOX_API_PASSWORD`) added to Vercel if portal switcher is needed in production
  5. `LMS_PORTAL_BASE_URL` set in Vercel if the internal hostname differs from the API hostname
- **Instruction:** Verify each item. If any fail, fix before pushing. The most common failure is Artifactory URLs in the lockfile — Vercel gets 401 trying to fetch from the corporate proxy.

---

## Knowledge: Vercel deployment

- Hosting: Vercel **Hobby** plan, Fluid Compute enabled
- Deploys from: `main` branch on GitHub (justinchapian/lms-admin)
- Build command: `npm install && next build` (Vercel default)

## Knowledge: Vercel Hobby plan limits & constraints

- Function Invocations: 1,000,000/month included
- Edge Requests: 1,000,000/month included
- Spend Management is **Pro-only** — not available on Hobby
- No built-in alerts for approaching limits; must check dashboard manually or build custom tracking
- If you exceed limits on Hobby, the feature pauses until 30 days have passed
- Current usage is very low (~1.4K function invocations/month as of August 2026)

### Why invocation limits are not a concern
All function invocations are user-initiated (no cron jobs, webhooks, or scheduled triggers). The app's request pipeline includes a circuit breaker (trips after 5 failures), retry cap (max 3), concurrency limiter (5 slots), and adaptive throttler — making runaway invocation loops structurally impossible. Client-side polling (bulk reports) uses proper cancellation, and pagination loops are bounded by API response headers. No internal route-to-route calls exist, so chain reactions can't happen.

## Knowledge: Corporate proxy pitfall

This project is developed on a corporate network that routes npm through Artifactory (`artifactory.spectrumtoolbox.com`). Vercel cannot authenticate against that registry.

npm bakes the `resolved` URL into `package-lock.json` based on whatever registry was active at install time. If the corporate Artifactory proxy was active, those URLs end up in the lockfile. Vercel then tries to fetch from Artifactory, gets a 401 (no auth token), and the build fails.
