# CLAUDE.md

## Repository Overview

This is a **GitHub Skills course template** — "Introduction to GitHub". It is a self-paced, interactive learning exercise hosted on GitHub. Learners fork or use this template repository and are guided through foundational GitHub concepts by a GitHub Actions-driven workflow that automatically advances them through steps as they complete activities.

The repository has no application code, build system, or test suite. Its content is purely instructional markdown, supporting images, and GitHub Actions workflows.

---

## Repository Structure

```
.
├── README.md                          # Active course content shown to learner (auto-updated by Actions)
├── LICENSE                            # MIT license
├── .gitignore                         # Standard OS/binary ignores; no language-specific tooling
├── images/                            # Screenshot PNGs embedded in course markdown
│   ├── profile-readme-example.png
│   ├── code-tab.png
│   ├── main-branch-dropdown.png
│   ├── create-branch-button.png
│   ├── my-profile-file.png
│   ├── create-new-file.png
│   ├── commit-full-screen.png
│   ├── compare-and-pull-request.png
│   ├── pull-request-branches.png
│   ├── Pull-request-description.png
│   ├── Actions-to-step-4.png
│   ├── Green-merge-pull-request.png
│   └── delete-branch.png
└── .github/
    ├── dependabot.yml                 # Monthly auto-updates for GitHub Actions versions
    ├── steps/                         # Canonical step content fragments (source of truth)
    │   ├── -step.txt                  # Current step counter (0–4 or X)
    │   ├── 0-welcome.md
    │   ├── 1-create-a-branch.md
    │   ├── 2-commit-a-file.md
    │   ├── 3-open-a-pull-request.md
    │   ├── 4-merge-your-pull-request.md
    │   └── X-finish.md
    └── workflows/                     # GitHub Actions that drive the course progression
        ├── 0-welcome.yml
        ├── 1-create-a-branch.yml
        ├── 2-commit-a-file.yml
        ├── 3-open-a-pull-request.yml
        └── 4-merge-your-pull-request.yml
```

---

## Course Mechanics

### Step Counter

`.github/steps/-step.txt` contains a single integer representing the learner's current step (`0` through `4`, then `X` for finished). Every workflow reads this file to decide whether to act. **Do not modify this file directly** unless you intend to reset or advance the course state.

### Workflow Progression

Each workflow is responsible for one step transition:

| Workflow file | Trigger | Condition | Advances to |
|---|---|---|---|
| `0-welcome.yml` | push to `main` | step == 0, not template repo | Step 1 |
| `1-create-a-branch.yml` | branch/tag created | step == 1, branch name == `my-first-branch` | Step 2 |
| `2-commit-a-file.yml` | push to `my-first-branch` | step == 2 | Step 3 |
| `3-open-a-pull-request.yml` | PR opened/reopened | step == 3, head == `my-first-branch` | Step 4 |
| `4-merge-your-pull-request.yml` | push to `main` | step == 4 | Step X (finish) |

All transitions are performed by `skills/action-update-step@v2`, which rewrites `README.md` in place with the next step's content and increments `-step.txt`.

### Branch Name Matters

The learner **must** name their branch exactly `my-first-branch`. The workflows hard-code this name; a differently-named branch will not trigger step advancement.

### Template Repository Guard

Every workflow checks `!github.event.repository.is_template`. When this repo is used as a template (not instantiated), none of the auto-advance logic fires. This prevents polluting the source template's state.

---

## README.md Conventions

`README.md` is the live course document. It is **generated content** — each step's content fragment lives canonically in `.github/steps/<n>-<name>.md` and is injected into `README.md` by `skills/action-update-step@v2` at runtime.

- Do not treat `README.md` as the source of truth for step content; edit the files under `.github/steps/` instead.
- The file uses a `<header>` / `<footer>` HTML wrapper pattern for course metadata.
- Images are referenced as root-relative paths like `/images/filename.png`.

---

## Development Workflows

### Modifying Step Content

1. Edit the relevant file in `.github/steps/` (e.g., `2-commit-a-file.md`).
2. If the change needs to appear immediately in `README.md` for a learner currently on that step, also update the matching section of `README.md` manually, or re-trigger the workflow.

### Updating Workflow Actions

Dependabot handles monthly updates to action versions (e.g., `actions/checkout@v4`). All workflows use `ubuntu-latest` and require only `contents: write` permission.

### Adding or Replacing Images

- Place new images in `images/` as PNG files.
- Reference them in markdown as `/images/<filename>.png` (absolute root path).
- Keep images at a reasonable resolution; they are screenshots for instructional clarity, not decorative assets.

### Resetting Course State

To reset a learner's repo to step 0:
- Set `.github/steps/-step.txt` content to `0`.
- Replace `README.md` content with the step-0 fragment from `.github/steps/0-welcome.md` wrapped in the standard `<header>`/`<footer>`.

---

## Key Conventions

- **No build system, no package manager, no linter.** There is nothing to `install` or `run`.
- **Markdown flavour**: GitHub Flavored Markdown (GFM). Use `> [!NOTE]` callouts for tips within steps.
- **Commit messages in step instructions** follow imperative mood (e.g., `Add PROFILE.md`).
- **Permissions**: All workflows use the minimum required — `contents: write` only.
- **Runner**: All jobs use `runs-on: ubuntu-latest`.
- **`fetch-depth: 0`**: All checkout steps fetch full history to support branch-aware operations in `skills/action-update-step@v2`.

---

## What This Repo Is Not

- It is not a software application — there is no code to run, compile, or test.
- It is not a documentation site — do not add build tooling (Jekyll, MkDocs, etc.) unless the course scope changes.
- Changes to workflows should be tested by forking/instantiating the template, not by pushing to `main` of the template itself (doing so triggers `0-welcome.yml`).
