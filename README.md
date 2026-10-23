# CI/CD Lab — GitHub Actions + Render

A small Express/TypeScript app delivered through a CI/CD pipeline: every pull request is verified by **GitHub Actions**,
`main` is protected by a ruleset that requires the **Verify** check, and **Render** deploys `main` only after CI passes.
Pull requests also get a temporary **preview deployment**.

| | Link |
|---|---|
| GitHub repository | https://github.com/soulaymahableni/cicd-lab |
| Production (Render) | https://cicd-lab-hb9k.onrender.com |
| Health endpoint | https://cicd-lab-hb9k.onrender.com/api/health |
| Version endpoint | https://cicd-lab-hb9k.onrender.com/api/version |

## Pipeline

```
feature branch ──push──► Pull request ──► GitHub Actions "Verify"
                                              │  npm ci → typecheck → test → build
                                              ▼
                         Ruleset "Protect main": merge blocked until Verify passes
                                              │ merge
                                              ▼
                         main ──► Verify on main ──► Render deploys (After CI Checks Pass)

Pull request opened ──► Render preview (cicd-lab-pr-N.onrender.com), deleted after merge
```

## Local development

Requires Node.js 24.

```bash
npm ci
npm run dev            # http://localhost:3000
npm run verify         # typecheck + tests + build
```

| Script | Purpose |
|---|---|
| `dev` | Development server with reload (tsx watch) |
| `typecheck` | TypeScript check without output |
| `test` | Node test runner (`/api/health`, `/api/version`) |
| `build` | Compile to `dist/` |
| `start` | Run the compiled server |
| `verify` | `typecheck && test && build` — the same command CI runs |

## Render configuration

| Setting | Value |
|---|---|
| Service type | Node Web Service, branch `main`, Free instance |
| Build Command | `npm ci && npm run build` |
| Start Command | `npm start` |
| Health Check Path | `/api/health` |
| Auto-Deploy | After CI Checks Pass |
| Pull Request Previews | Automatic |

## Evidence

### 0. Local verification (`npm run verify`)
![Local verify](captures/local-verify.png)

### 1. First Render deployment and `/api/health`
![First Render deploy](captures/render-first-deploy.png)

### 2. Verify job passing in GitHub Actions
![Verify passing](captures/actions-verify-pass.png)

Workflow runs for the pull request and for `main` after the merge:

![Workflow runs](captures/actions-workflows.png)

### 3. Render Auto-Deploy: After CI Checks Pass
![Render auto-deploy](captures/render-auto-deploy.png)

### 4. Ruleset "Protect main" requiring Verify
![Ruleset](captures/ruleset-protect-main.png)

### 5. Home page updated through the pipeline
![Homepage updated](captures/homepage-updated.png)

### 6. Failing Verify blocks the merge
The test for `/api/version` was pushed before the route existed (404 instead of 200):

![Local failing test](captures/local-test-failing.png)

![PR failing verify](captures/pr-failing-verify.png)

After adding the route to the same branch, the pull request turned green:

![PR fixed](captures/pr-verify-fixed.png)

### 7. `/api/version` after deployment
![API version](captures/api-version.png)

### 8. Preview vs production
Pull Request Previews set to Automatic in Render:

![Render PR previews](captures/render-pr-previews.png)

![Preview vs production](captures/preview-vs-production.png)

## Report

See [REPORT.md](REPORT.md).
