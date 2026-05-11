# GitHub Actions Workflow Design: Automated Build and Deploy to gh-pages

**Date:** 2026-05-11  
**Project:** humorless.github.io (Cryogen static site)

## Overview

Design a GitHub Actions workflow that automatically builds the Cryogen static site and deploys it to the `gh-pages` branch whenever changes are pushed to the `code` branch.

## Requirements

- **Trigger:** Push to `code` branch
- **Build:** Execute `lein run` to generate static site to `resources/public/`
- **Deploy:** Push generated content to `gh-pages` branch (complete replacement, not incremental)
- **Failure Handling:** Workflow fails if build fails; no deployment occurs
- **Credentials:** Use GitHub-provided `GITHUB_TOKEN`

## Architecture

### Workflow Structure

**File:** `.github/workflows/deploy.yml`

**Trigger Event:** `push` on `code` branch only

**Jobs:** Single job with sequential steps

### Build and Deployment Flow

```
User pushes to code branch
    ↓
GitHub Actions triggers workflow
    ↓
Step 1: Checkout code branch
    ↓
Step 2: Set up Java/Clojure environment
    ↓
Step 3: Run `lein run` (generates to resources/public/)
    ↓
Step 4: Deploy resources/public/ to gh-pages branch
    ↓
Workflow completes
```

## Detailed Workflow Steps

### Step 1: Checkout
- Uses `actions/checkout@v3`
- Checks out `code` branch (default behavior)

### Step 2: Setup Java Environment
- Uses `actions/setup-java@v3`
- Java version: 11 (Clojure/lein compatible)
- Distribution: temurin (OpenJDK)

### Step 3: Build Static Site
- Run: `lein run`
- Generates HTML/CSS/JS to `resources/public/`
- If this step fails, workflow stops (no deployment)

### Step 4: Deploy to gh-pages
- Uses `github-pages-deploy-action`
- Source folder: `resources/public/`
- Target branch: `gh-pages`
- Token: `${{ secrets.GITHUB_TOKEN }}`
- Strategy: Force update (complete replacement)

## Error Handling

- Any failed step (lein compile errors, missing dependencies, etc.) causes entire workflow to fail
- Failed workflows do NOT proceed to deployment step
- GitHub Actions native failure handling (no custom error logic needed)
- Users can view failure details in Actions tab

## Prerequisites (Manual Setup)

**Before deploying workflow to production:**

1. Create `gh-pages` branch in GitHub (can be empty, or pushed once for safety)
2. Go to repository Settings → Pages
3. Change "Build and deployment" → "Branch" from `master` to `gh-pages`
4. Save changes
5. GitHub Pages will read from `gh-pages` branch moving forward

## Configuration Details

### Secrets and Tokens
- Uses GitHub-provided `GITHUB_TOKEN` (automatically available)
- No additional secrets required
- Token has necessary permissions for gh-pages branch writes

### Caching (Optional, Not Included)
- Could add Maven/Clojure dependency caching for faster builds
- Not critical for this project; can add later if build times become an issue

### Workflow Permissions
- Read: Default (code access)
- Write: `GITHUB_TOKEN` has necessary permissions for gh-pages push

## Verification Steps (Post-Deployment)

1. Make test commit on `code` branch
2. Verify workflow runs in Actions tab
3. Check `gh-pages` branch has new commits from action
4. Verify website updates at `https://humorless.github.io`

## File Structure After Deployment

```
gh-pages branch:
  index.html
  css/
  js/
  posts/
  ... (all content from resources/public/)
```

Note: `resources/public/` directory itself is NOT pushed; only its contents.

---

**Status:** Ready for review and implementation planning
