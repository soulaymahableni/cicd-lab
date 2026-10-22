# Report — TP3: CI/CD with GitHub Actions and Render

## 1. The pipeline in my own words
The app is a small Express/TypeScript service. Locally, a single `npm run verify` command runs the type check, the tests and
the build; CI runs exactly the same command, so "it works on my machine" and "it works in CI" mean the same thing.

Every change goes through a **feature branch and a pull request**. Opening the PR triggers the GitHub Actions workflow
`CI`, whose job **Verify** installs dependencies with `npm ci` on Node 24 and runs `npm run verify`. The ruleset
**Protect main** makes a pull request mandatory and lists Verify as a required status check, with an empty bypass list,
so nobody (not even the owner) can merge red code. After the merge, Verify runs again on `main`, and Render is set to
**Auto-Deploy: After CI Checks Pass**, so production is only updated with a commit that has passed CI.

I demonstrated the safety net by pushing a test for `/api/version` before the route existed: the test failed
(404 instead of 200), Verify turned red and the merge button was blocked. Adding the route to the same branch updated
the pull request, Verify passed and the merge became possible; `/api/version` then returned `{"version":"development"}`
in production. Finally, Render **pull request previews** built a temporary copy of the app for the open PR
(`cicd-lab-pr-4-…onrender.com`) showing "Preview deployment" while production did not; after the merge production showed
the message and the preview was removed.

## 2. Comparison with TP2 (Coolify webhook vs GitHub Actions + Render)

| Criterion | TP2 — Forgejo + Coolify (webhook) | TP3 — GitHub Actions + Render |
|---|---|---|
| Hosting | Self-hosted in my own VM | Managed cloud services (GitHub, Render) |
| Trigger | Forgejo webhook on every push to `main` | Pull request + required check, then merge |
| Tests before deploy | None: any push is built and deployed | Typecheck + tests + build must pass first |
| Bad commit | Build failed, old container kept serving | Blocked at the pull request, never reaches `main` |
| Review step | Push straight to `main` | Pull request, rules enforced by the ruleset |
| Previews | Not set up | Automatic preview URL per pull request |
| Network | Private IP, hosts file, webhook allow-list | Public HTTPS URLs out of the box |
| Control / cost | Full control, my hardware, more maintenance | Little maintenance; free tier limits (cold start ~50 s) |
| Configuration as code | Coolify settings stored in its database | Workflow `ci.yml` versioned in the repository |

**Conclusion:** TP2 shows *continuous deployment* (every push is deployed); TP3 adds *continuous integration* in front of
it, so only verified code reaches production. The TP3 approach is safer for teamwork; the TP2 approach is attractive
when everything must stay on-premises.

## 3. Difficulties encountered
- **Full disk on the VM** (`No space left on device` while installing Node 24): Coolify build caches from TP2 had
  filled the 19 GB disk; `docker builder prune` and `docker image prune` freed about 5 GB.
- **Port 3000 already used** on the VM by Semaphore: the dev server was started with `PORT=3005`.
- **GitHub token scopes**: pushing a file under `.github/workflows` requires the `workflow` scope in addition to `repo`.
- **Render free instance** sleeps when idle; the first request after inactivity takes up to ~50 seconds.
- **Ruleset side effect**: once `main` is protected, even documentation (this report) must go through a pull request.
