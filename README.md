# GitLab CI/CD Pipeline — React + Vite

A hands-on DevOps project built from a course starter and extended with a fully working multi-stage GitLab CI/CD pipeline, automated Netlify deployments (review, staging, production), and Playwright end-to-end testing.

> Originally forked from a course project. I wrote the complete `.gitlab-ci.yml` from scratch, debugged real pipeline failures, and extended the pipeline beyond the course scope.

---

## Pipeline Overview

```
build → test → deploy_review → deploy_staging → deploy_prod
```

| Stage | Job | What it does |
|---|---|---|
| `build` | `build_website` | `npm ci` + `vite build`, passes `build/` as artifact |
| `test` | `test_artifacts` | Verifies `build/index.html` exists |
| `test` | `unit_tests` | Runs Vitest, publishes JUnit report to GitLab |
| `deploy_review` | `netlify_review` | Deploys preview to Netlify on feature branches, captures dynamic URL |
| `deploy_staging` | `netlify_staging` | Deploys to staging alias on `main` only |
| `deploy_prod` | `netlify_prod` | Deploys to production on `main` only |

**Key pipeline features:**
- `workflow:rules` — skips pipeline on merge request events, runs only on branch pushes
- Branch-scoped rules — review deploy on feature branches, staging/prod on `main`
- Dynamic `REVIEW_URL` — captured from Netlify CLI JSON output via `jq`, passed to downstream jobs via `artifacts: reports: dotenv`
- Smoke test after every deploy — `curl` + `grep 'GitLab'` to verify the live URL
- JUnit test reports published as GitLab CI artifacts for inline test result display

---

## Tech Stack

| Layer | Tool |
|---|---|
| Frontend | React 18 + Vite 6 |
| Unit Testing | Vitest + Testing Library |
| E2E Testing | Playwright (Chromium) |
| Linting | ESLint 9 |
| CI/CD | GitLab CI/CD |
| Deployment | Netlify (review / staging / production) |
| CI Image | `node:22-alpine` |

---

## What I Built Beyond the Course

The course provided the React + Vite app skeleton. I built everything else:

- **Wrote `.gitlab-ci.yml` from scratch** — 5 stages, 6 jobs, workflow rules, environment definitions
- **Fixed Alpine bash issue** — `node:22-alpine` has no bash by default; added `apk add --no-cache bash` to unblock Netlify CLI
- **Fixed `npm ci` ordering** — moved before `netlify deploy` to resolve `vite: not found` error
- **Dynamic review URL capture** — used `netlify deploy --json | tee deploy-result.json` + `jq -r '.deploy_url'` to extract and pass the preview URL via `dotenv` artifact
- **Smoke tests on every environment** — `curl $URL | grep 'GitLab'` after each deploy
- **Vitest JUnit report** — configured `vite.config.js` reporters to output `reports/junit.xml` and wired it to `artifacts: reports: junit`

---

## Project Structure

```
├── src/                    # React app source
├── tests/                  # Vitest unit tests
├── e2e/                    # Playwright e2e tests
├── .gitlab-ci.yml          # Full CI/CD pipeline (written by me)
├── playwright.config.cjs   # Playwright config — baseURL from APP_BASE_URL env var
├── vite.config.js          # Vite + Vitest config with JUnit/HTML reporters
├── netlify.toml            # Netlify build settings (output dir: build/)
└── package.json
```

---

## Local Setup

```bash
# Install dependencies
npm install

# Start dev server
npm run dev

# Run unit tests
npm test

# Run e2e tests (app must be running at localhost:3000)
npm run e2e

# Lint
npm run lint
```

---

## CI/CD Environment Variables

Stored in **GitLab → Settings → CI/CD → Variables** (masked):

| Variable | Description |
|---|---|
| `NETLIFY_AUTH_TOKEN` | Netlify personal access token |
| `NETLIFY_SITE_ID` | Target Netlify site ID |
| `REVIEW_URL` | Captured dynamically from Netlify CLI deploy output via `dotenv` artifact |

---

## Live Environments

| Environment | Trigger | URL |
|---|---|---|
| Preview (review) | Feature branch push | Dynamic per branch (via Netlify alias) |
| Staging | `main` branch push | `https://staging--learn-gitlab-thanh.netlify.app` |
| Production | `main` branch push | Netlify production URL |