# Dynamic CI Trigger Dashboard

[Open Dashboard](https://saikrishnakalle.github.io/program2/)


A browser-based dashboard that automatically changes its default code/test files the moment you select a different branch — something GitHub's own UI cannot do.

---

## 🎯 The Core Reason This Project Exists

In GitHub Actions, when you go to **Actions → Run workflow**, you get a static form. Its inputs and defaults are fixed the moment the page loads — they **do not** update based on which branch you select.

So this is **not possible** natively in GitHub:

> Select a branch in the dropdown → defaults (code file, test file) automatically update on screen → before you've even clicked Run.

We tested this directly and confirmed: GitHub Actions UI cannot do dynamic, branch-based default switching. It only shows computed values *after* a run has already started (via Step Summary), which defeats the purpose of *previewing* before triggering anything.

**This is exactly the gap this project fills.**

We built a custom HTML dashboard where:

- The moment you **select a branch**, the page **instantly and automatically updates** to show that branch's default code file and test file
- This happens entirely in the browser, with no run triggered yet — true dynamic behavior, unlike GitHub's static form
- Only after reviewing the auto-updated defaults do you click **Run Workflow** to actually trigger the CI pipeline

---

## 📌 What This Project Does

1. Select a branch from a dropdown
2. Defaults (code file + test file) **automatically change** right there on screen — dynamically, instantly, before any run starts
3. Click **Run Workflow**
4. Watch live status, step-by-step progress, and pass/fail results — right inside the dashboard

No need to dig through GitHub Actions logs manually, and no risk of running the wrong test file against the wrong branch.

---

## 🏗️ Architecture

```
HTML Dashboard (GitHub Pages)
        ↓
  Branch selected → defaults shown instantly (client-side, before run)
        ↓
  GitHub REST API (workflow_dispatch)
        ↓
  Single ci.yml workflow (lives only on "main")
        ↓
  Workflow checks out the SELECTED branch's code
        ↓
  Detects branch → assigns correct code/test file
        ↓
  Runs program + pytest
        ↓
  Dashboard polls GitHub API → shows live step status + final result
```

---

## ⚙️ What We Used

| Component | Purpose |
|---|---|
| **GitHub Actions** | Runs the actual CI pipeline (`workflow_dispatch` trigger) |
| **GitHub REST API** | Lets the webpage trigger workflows and fetch run/job status remotely |
| **GitHub Pages** | Hosts the HTML dashboard as a public static webpage |
| **HTML / CSS / JavaScript** | Builds the interactive dashboard UI — branch selector, file preview, live status |
| **Python + pytest** | The actual code and tests being run inside the CI pipeline |
| **Personal Access Token (PAT)** | Authenticates API requests from the dashboard to GitHub |

---

## 📂 Project Structure

```
main branch:
├── .github/workflows/ci.yml      ← the ONE and ONLY workflow file

hello-world branch:
├── hello.py
├── test_hello.py

math-app branch:
├── math_app.py
├── test_math.py

gh-pages branch (or docs/ folder):
├── index.html                    ← the dashboard
```

> Important: the workflow file exists **only on `main`**. It is never duplicated across branches. It dynamically checks out whichever branch is selected and figures out the right files to run.

---

## 🔑 Key Design Decisions

### 1. Single workflow, not one per branch
Rather than maintaining a separate `ci.yml` in every branch (which drifts out of sync over time), there is exactly **one workflow on `main`**. It's always triggered from `main`, and the branch to test is passed in as an *input*, not as the `ref`.

### 2. Defaults are shown client-side, before the run
GitHub can't show dynamic previews before a run — but our own HTML page can, because we mirror the branch → file mapping directly in JavaScript. The moment a branch is selected, the file preview updates instantly, with zero API calls.

### 3. Live results, not just a "triggered" message
After dispatching the workflow, the dashboard polls the GitHub API every few seconds to show:
- Running / Success / Failure status
- Each step with ✅ / ❌ / ⏳ icons and duration
- A direct link to the full GitHub Actions log

---

## 🚀 How To Use It

1. Open the dashboard (hosted via GitHub Pages)
2. Paste in your GitHub Personal Access Token (kept only in your browser session, never stored)
3. Select a branch from the dropdown
4. Review the default code/test files shown automatically
5. Click **Run Workflow**
6. Watch the run progress live, with a link to full logs if needed

---

## ✅ Why This Is Useful

- **Saves time** — no manual GitHub UI navigation for routine test runs
- **Prevents mistakes** — you always see exactly which files will run before triggering anything
- **Cleaner repo** — no duplicate workflow files scattered across branches
- **Beginner-friendly** — works as an internal tool/dashboard without needing a backend server
- **Demonstrates real CI/CD concepts** — workflow_dispatch, branch-based logic, REST API integration, and GitHub Pages hosting all in one project

---

## ⚠️ Known Limitations

- The GitHub Personal Access Token is entered directly in the browser. This is fine for personal/learning projects but **not recommended for production** — a real production setup would route requests through a backend server or GitHub App instead of exposing a token in the frontend.
- Branch → file mapping is currently hardcoded in both `ci.yml` and the dashboard's JavaScript. Adding a new branch means updating both places.

---

## 🔮 Possible Future Improvements

- Move token handling to a backend proxy instead of the browser
- Auto-generate the branch → file mapping from a shared config file instead of duplicating it in two places
- Add test coverage reports and badges per branch
- Support multiple workflows/jobs in the same dashboard
