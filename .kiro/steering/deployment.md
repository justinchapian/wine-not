---
inclusion: always
---
# Deployment & npm Safety

---

## Rules

### RULE: Use public npm registry for all package operations
- **Trigger:** Running `npm install`, `npm update`, or any command that modifies `package-lock.json`
- **Check:** Does the command include `--registry=https://registry.npmjs.org`?
- **Instruction:** Always append `--registry=https://registry.npmjs.org` to the command. After the command completes, scan `package-lock.json` for "artifactory" references. If any are found, replace all occurrences of `https://artifactory.spectrumtoolbox.com/artifactory/api/npm/npm/` with `https://registry.npmjs.org/` before committing.

### RULE: Pre-push checklist — verify before pushing to remote
- **Trigger:** About to push code to GitHub (or recommending the user push)
- **Check:** Have all of the following been verified?
  1. No Artifactory URLs in `package-lock.json`
  2. No `.npmrc` file committed (it's in `.gitignore`)
  3. Environment variables set in hosting dashboard (not in committed files)
- **Instruction:** Verify each item. If any fail, fix before pushing.

---

## Knowledge: Hosting

_To be filled in once deployment target is decided._

## Knowledge: Corporate proxy pitfall

This project is developed on a corporate network that routes npm through Artifactory (`artifactory.spectrumtoolbox.com`). The hosting provider cannot authenticate against that registry. Always use `--registry=https://registry.npmjs.org` and sanitize the lockfile before pushing.
