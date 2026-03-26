---
title: learning-slos — GitHub Pages Setup
date: 2026-03-26
status: draft
---

# Design: learning-slos GitHub Pages Setup

## Goal

Publish the SLO Mastery interactive tutorial as a public GitHub repo at `arnlaugsson/learning-slos`, hosted on GitHub Pages, with automatic deployment on every commit or merged PR.

## Repository Structure

```
learning-slos/
├── index.html                  ← renamed from slo-mastery-tutorial-v0-2026-03-26.html
├── README.md                   ← project documentation
└── .github/
    └── workflows/
        └── deploy.yml          ← GitHub Actions deployment workflow
```

No build step. No dependencies. The HTML file is fully self-contained.

## GitHub Actions Workflow (`deploy.yml`)

- **Trigger**: push to `main`
- **Concurrency**: group `pages`, `cancel-in-progress: true` (prevents race conditions on rapid pushes)
- **Permissions** (top-level): `contents: read`, `pages: write`, `id-token: write`
- **Two jobs**:
  1. `build` job: uses `actions/upload-pages-artifact@v3` with `path: '.'` to upload the repo root
  2. `deploy` job: `needs: build`, declares `environment: { name: github-pages, url: ${{ steps.deployment.outputs.page_url }} }`, uses `actions/deploy-pages@v4`
- **Live URL**: `https://arnlaugsson.github.io/learning-slos/`

## README Contents

- Title and one-line description
- Live demo badge/link pointing to the GitHub Pages URL
- Brief content overview (3 chapters, quizzes, error budget simulator)
- Local dev instructions (just open `index.html` in a browser)

No screenshot — avoids maintaining a stale image asset.

## What Is Not Included

- PR deploy previews
- Contributing guide or changelog
- File splitting (single-file is intentional and self-contained)
- Any server-side components or build pipeline

## Implementation Steps

1. `git init -b main` in the project directory (ensures branch is named `main`)
2. Rename `slo-mastery-tutorial-v0-2026-03-26.html` → `index.html`
3. Create `.github/workflows/deploy.yml` with the two-job Pages workflow
4. Create `README.md`
5. `git add` all files and commit to `main`
6. Create public GitHub repo and push in one command: `gh repo create arnlaugsson/learning-slos --public --source=. --remote=origin --push`
7. Enable GitHub Pages (Source: GitHub Actions) via `gh api /repos/arnlaugsson/learning-slos/pages` or repo Settings UI — must be done before or immediately after step 6 so the first workflow run succeeds
8. Verify the Actions workflow completes and the live URL is accessible
