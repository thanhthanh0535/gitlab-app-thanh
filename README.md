What I Built Beyond...

The course provided the React + Vite app skeleton. I built everything else:


Wrote .gitlab-ci.yml from scratch — 5 stages, 6 jobs, workflow rules, environment definitions
Fixed Alpine bash issue — node:22-alpine has no bash by default; added apk add --no-cache bash to unblock Netlify CLI
Fixed npm ci ordering — moved before netlify deploy to resolve vite: not found error
Dynamic review URL capture — used netlify deploy --json | tee deploy-result.json + jq -r '.deploy_url' to extract and pass the preview URL via dotenv artifact
Smoke tests on every environment — curl $URL | grep 'GitLab' after each deploy
Vitest JUnit report — configured vite.config.js reporters to output reports/junit.xml and wired it to artifacts: reports: junit



Project Structure

├── src/                    # React app source
├── tests/                  # Vitest unit tests
├── e2e/                    # Playwright e2e tests
├── .gitlab-ci.yml          # Full CI/CD pipeline (written by me)
├── playwright.config.cjs   # Playwright config — baseURL from APP_BASE_URL env var
├── vite.config.js          # Vite + Vitest config with JUnit/HTML reporters
├── netlify.toml            # Netlify build settings (output dir: build/)
└── package.json


Local Setup

bash# Install dependencies
npm install

# Start dev server
npm run dev

# Run unit tests
npm test

# Run e2e tests (app must be running at localhost:3000)
npm run e2e

# Lint
npm run lint


CI/CD Environment Variables

Stored in GitLab → Settings → CI/CD → Variables (masked):

VariableDescriptionNETLIFY_AUTH_TOKENNetlify personal access tokenNETLIFY_SITE_IDTarget Netlify site IDREVIEW_URLCaptured dynamically from Netlify CLI deploy output via dotenv artifact


Live Environments

EnvironmentTriggerURLPreview (review)Feature branch pushDynamic per branch (via Netlify alias)Stagingmain branch pushhttps://staging--learn-gitlab-thanh.netlify.appProductionmain branch pushNetlify production URL
