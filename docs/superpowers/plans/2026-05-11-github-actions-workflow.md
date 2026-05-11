# GitHub Actions Deploy Workflow Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a GitHub Actions workflow that automatically builds the Cryogen site with `lein run` and deploys it to the `gh-pages` branch on every push to the `code` branch.

**Architecture:** Single workflow file (`.github/workflows/deploy.yml`) with four sequential steps: checkout code, setup Java environment, run lein build, deploy generated site to gh-pages using `github-pages-deploy-action`.

**Tech Stack:** GitHub Actions, Leiningen (lein), JDK 11, github-pages-deploy-action

---

## File Structure

**Files to Create:**
- `.github/workflows/deploy.yml` — Main workflow configuration file

**Directories:**
- `.github/workflows/` — Will be created by the workflow file

---

## Task 1: Create GitHub Actions Workflow File

**Files:**
- Create: `.github/workflows/deploy.yml`

- [ ] **Step 1: Create the .github/workflows directory**

```bash
mkdir -p .github/workflows
```

- [ ] **Step 2: Write the deploy.yml workflow file**

```yaml
name: Build and Deploy to gh-pages

on:
  push:
    branches:
      - code

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code branch
        uses: actions/checkout@v3
      
      - name: Set up Java environment
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'temurin'
      
      - name: Build static site with Cryogen
        run: lein run
      
      - name: Deploy to gh-pages
        uses: github-pages-deploy-action@v4.5.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: gh-pages
          folder: resources/public
          single-commit: false
```

- [ ] **Step 3: Verify the workflow file is valid YAML**

```bash
cat .github/workflows/deploy.yml
```

Expected: File displays correctly with proper indentation and structure.

- [ ] **Step 4: Commit the workflow file**

```bash
git add .github/workflows/deploy.yml
git commit -m "feat: add GitHub Actions workflow for automated deployment to gh-pages

- Triggers on push to code branch
- Builds site using lein run
- Deploys to gh-pages branch using github-pages-deploy-action

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 2: Manual GitHub Settings Configuration

**No code changes — manual GitHub UI steps**

- [ ] **Step 1: Create gh-pages branch in GitHub**

If `gh-pages` branch doesn't exist:
1. Go to your GitHub repository
2. Click "Branches"
3. Create new branch named `gh-pages` (can start from `code` or `master`)
4. Push empty branch if created locally: `git push origin gh-pages`

- [ ] **Step 2: Update GitHub Pages settings**

1. Go to your repository Settings
2. Navigate to "Pages" (in left sidebar)
3. Under "Build and deployment" → "Branch", select `gh-pages` from dropdown
4. Ensure folder is set to `/ (root)`
5. Click "Save"

**Verification:**
- You should see "Your site is live at https://humorless.github.io" message

---

## Task 3: Test the Workflow

**No code changes — testing the automation**

- [ ] **Step 1: Create a test commit on code branch**

```bash
git checkout code
echo "<!-- Workflow test commit -->" >> README.md
git add README.md
git commit -m "test: trigger workflow deployment"
```

- [ ] **Step 2: Push to code branch**

```bash
git push origin code
```

- [ ] **Step 3: Monitor workflow execution**

1. Go to GitHub repository
2. Click "Actions" tab
3. Look for "Build and Deploy to gh-pages" workflow run
4. Wait for workflow to complete (should take 1-2 minutes)

**Expected outcomes:**
- Workflow shows "✓ passed" (green checkmark)
- All steps completed successfully
- No errors in logs

- [ ] **Step 4: Verify deployment**

1. Check that `gh-pages` branch has new commits from the action
   ```bash
   git fetch origin
   git log origin/gh-pages -5 --oneline
   ```
   
   Expected: See commit from "github-pages-deploy-action"

2. Verify website is updated
   - Visit `https://humorless.github.io` in browser
   - Check that content is from your `resources/public` build

3. Verify `.git` folder is NOT in gh-pages deployment
   ```bash
   git ls-tree -r origin/gh-pages | head -20
   ```
   
   Expected: Only static files (html, css, js, etc.), no `.git` folder

- [ ] **Step 5: Clean up test file**

```bash
git checkout code
git revert HEAD --no-edit
git push origin code
```

This will create a revert commit instead of rewriting history.

---

## Verification Checklist

- [ ] `.github/workflows/deploy.yml` exists and is valid YAML
- [ ] Workflow triggers on push to `code` branch (not other branches)
- [ ] Java environment setup works without errors
- [ ] `lein run` executes successfully and generates `resources/public/`
- [ ] Deployment to `gh-pages` completes without errors
- [ ] `gh-pages` branch contains only generated static files
- [ ] Website is live and reflects the deployed content
- [ ] Failed builds prevent deployment (not tested here, but ensure this behavior)

---

## Notes

- The `github-pages-deploy-action` automatically handles git configuration and commits to `gh-pages`
- Workflow uses `${{ secrets.GITHUB_TOKEN }}` which is automatically provided by GitHub Actions
- No additional secrets or credentials needed
- The `single-commit: false` setting preserves gh-pages history (optional, but good for debugging)

---

**Status:** Ready for execution
