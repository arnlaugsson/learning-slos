# learning-slos GitHub Pages Setup — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish the SLO Mastery tutorial as a public GitHub repo at `arnlaugsson/learning-slos` with automatic GitHub Pages deployment on every push to `main`.

**Architecture:** Single self-contained `index.html` served directly from the repo root. GitHub Actions runs on every push to `main` — one job uploads the repo root as a Pages artifact, a second job deploys it. No build step.

**Tech Stack:** GitHub Pages, GitHub Actions (`actions/upload-pages-artifact@v3`, `actions/deploy-pages@v4`), GitHub CLI (`gh`)

---

### Task 1: Initialize the git repository

**Files:**
- Create: `.git/` (via `git init`)

- [ ] **Step 1: Initialize git with `main` as the default branch**

```bash
cd /Users/arnlaugsson/Projects/Learning-SLOs
git init -b main
```

Expected output: `Initialized empty Git repository in .../Learning-SLOs/.git/`

- [ ] **Step 2: Verify branch name**

```bash
git branch
```

Expected: `* main`

---

### Task 2: Rename the HTML file to `index.html`

**Files:**
- Rename: `slo-mastery-tutorial-v0-2026-03-26.html` → `index.html`

- [ ] **Step 1: Rename the file**

```bash
mv /Users/arnlaugsson/Projects/Learning-SLOs/slo-mastery-tutorial-v0-2026-03-26.html \
   /Users/arnlaugsson/Projects/Learning-SLOs/index.html
```

- [ ] **Step 2: Verify**

```bash
ls /Users/arnlaugsson/Projects/Learning-SLOs/
```

Expected: `index.html` present, old filename gone.

---

### Task 3: Create the GitHub Actions deployment workflow

**Files:**
- Create: `.github/workflows/deploy.yml`

- [ ] **Step 1: Create the workflows directory**

```bash
mkdir -p /Users/arnlaugsson/Projects/Learning-SLOs/.github/workflows
```

- [ ] **Step 2: Write the workflow file**

Create `/Users/arnlaugsson/Projects/Learning-SLOs/.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

concurrency:
  group: pages
  cancel-in-progress: true

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 3: Verify the file exists and is well-formed**

```bash
cat /Users/arnlaugsson/Projects/Learning-SLOs/.github/workflows/deploy.yml
```

---

### Task 4: Create the README

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write the README**

Create `/Users/arnlaugsson/Projects/Learning-SLOs/README.md`:

```markdown
# SLO Mastery — Interactive Tutorial

**[Live Demo →](https://arnlaugsson.github.io/learning-slos/)**

An interactive tutorial and game for mastering Service Level Objectives (SLOs). Based on the Google SRE approach to reliability engineering.

## Contents

- **Chapter 1 — Embracing Risk:** Why 100% reliability is wrong, error budgets, the cost of reliability
- **Chapter 2 — SLIs, SLOs & SLAs:** How to define and measure service health
- **Chapter 3 — Implementing SLOs:** The full process from SLI spec to production
- **Final Challenge:** Error Budget Simulator game
- **Lab:** Interactive SLI Builder

## Run locally

```
open index.html
```

No build step, no dependencies.
```

- [ ] **Step 2: Verify**

```bash
cat /Users/arnlaugsson/Projects/Learning-SLOs/README.md
```

---

### Task 5: Commit all files

**Files:** all of the above

- [ ] **Step 1: Stage all files**

```bash
cd /Users/arnlaugsson/Projects/Learning-SLOs
git add index.html README.md .github/workflows/deploy.yml docs/
```

- [ ] **Step 2: Verify staged files**

```bash
git status
```

Expected: `index.html`, `README.md`, `.github/workflows/deploy.yml`, and `docs/` all staged under "Changes to be committed".

- [ ] **Step 3: Commit**

```bash
git commit -m "Initial commit: SLO Mastery tutorial with GitHub Pages deployment"
```

Expected: commit hash printed, `main` branch, 3+ files changed.

---

### Task 6: Create the GitHub repo and push

**Files:** remote `origin` added

- [ ] **Step 1: Create the repo and push in one command**

```bash
cd /Users/arnlaugsson/Projects/Learning-SLOs
gh repo create arnlaugsson/learning-slos --public --source=. --remote=origin --push
```

Expected output includes: `✓ Created repository arnlaugsson/learning-slos on GitHub` and `✓ Pushed commits to github.com/arnlaugsson/learning-slos`

- [ ] **Step 2: Verify remote is set**

```bash
git remote -v
```

Expected: `origin  https://github.com/arnlaugsson/learning-slos.git` (fetch and push)

---

### Task 7: Enable GitHub Pages

**Note:** GitHub Pages must be enabled before the first workflow deployment can succeed.

- [ ] **Step 1: Enable GitHub Pages via API (source: GitHub Actions)**

```bash
gh api \
  --method POST \
  -H "Accept: application/vnd.github+json" \
  /repos/arnlaugsson/learning-slos/pages \
  -f build_type=workflow
```

If this returns a 409 (already exists) or 422, Pages may already be configured — proceed to Step 2.

- [ ] **Step 2: Verify Pages is enabled**

```bash
gh api /repos/arnlaugsson/learning-slos/pages
```

Expected: JSON with `"source": {"branch": "main"}` and `"build_type": "workflow"` (or similar).

---

### Task 8: Verify deployment

- [ ] **Step 1: Watch the Actions workflow run**

```bash
gh run list --repo arnlaugsson/learning-slos --limit 5
```

Wait for the `Deploy to GitHub Pages` workflow to show `completed` status.

```bash
RUN_ID=$(gh run list --repo arnlaugsson/learning-slos --limit 1 --json databaseId -q '.[0].databaseId')
gh run watch "$RUN_ID" --repo arnlaugsson/learning-slos
```

- [ ] **Step 2: Confirm the live URL is accessible**

```bash
open https://arnlaugsson.github.io/learning-slos/
```

Expected: The SLO Mastery tutorial loads correctly in the browser.

- [ ] **Step 3: Done** ✓
