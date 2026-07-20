# CLAUDE.md

This file gives Claude Code (and other AI assistants) the context needed to work in this repository.

## What this repository is

This is **"Introduction to GitHub"**, one of GitHub's official **Skills** courses
(https://github.com/skills). It is a **template repository**, not an application — there
is no source code, build system, package manifest, or test suite. The entire "product" is
a self-updating interactive lesson: a learner clicks through GitHub.com actions (create a
branch, commit a file, open a pull request, merge it), and GitHub Actions workflows detect
each action and rewrite `README.md` to reveal the next step.

Because there is nothing to build, lint, or test, most standard software-engineering
workflows (install deps, run tests, start a dev server) do not apply here. Treat requests
to "run the tests" or "build the project" as a sign the requester may be assuming this is a
normal app — clarify with them if it's ambiguous.

## Repository structure

```
README.md                          # Rendered lesson content; overwritten by workflows as the learner progresses
.github/
  steps/
    -step.txt                      # Single integer: the current step number (source of truth for workflow gating)
    0-welcome.md                   # Step 0 content, injected into README.md between <!-- readme --> markers
    1-create-a-branch.md           # (step 1 content lives inline in README.md itself — see note below)
    2-commit-a-file.md
    3-open-a-pull-request.md
    4-merge-your-pull-request.md
    X-finish.md                    # Final "course complete" content
  workflows/
    0-welcome.yml                  # push to main            -> advances step 0 to 1
    1-create-a-branch.yml          # branch `create` event   -> advances step 1 to 2
    2-commit-a-file.yml            # push to my-first-branch -> advances step 2 to 3
    3-open-a-pull-request.yml      # pull_request opened     -> advances step 3 to 4
    4-merge-your-pull-request.yml  # push to main             -> advances step 4 to X (finish)
  dependabot.yml                   # Monthly dependency-update checks for GitHub Actions versions
images/                            # Screenshots referenced by the step markdown / README
LICENSE                            # MIT
```

There is no `src/`, no package manager files, and no CI beyond the step-progression
workflows above.

## How the step-progression mechanism works

Each workflow in `.github/workflows/` follows the same pattern:

1. **`get_current_step` job**: checks out the repo and reads `.github/steps/-step.txt` to
   get the current step number.
2. **Gated job** (e.g. `on_create_a_branch`): runs only if all of these hold:
   - the repository is not the template repo itself (`!github.event.repository.is_template`)
   - the current step number matches what this workflow expects
   - the triggering event matches (right branch name, right event type, etc.)
3. If the conditions pass, it runs `skills/action-update-step@v2`, which:
   - Rewrites `README.md` with the content for `to_step` (pulled from `.github/steps/<N>-*.md`)
   - Updates `.github/steps/-step.txt` to `to_step`
   - Commits the change back to the repo

The expected learner flow is: create branch `my-first-branch` → commit a file on it → open a
PR from it → merge the PR — each action is the trigger for the corresponding workflow. The
branch name `my-first-branch` is hard-coded throughout the workflows and step content.

## Conventions to follow when editing this repo

- **Step content markdown** (`.github/steps/N-*.md`): starts with an HTML comment of
  "Author notes" giving guidance for course authors (not shown to learners), followed by
  the actual learner-facing content. Preserve this pattern in any new/edited step file.
- **Workflow files**: heavily commented, explaining *why* each trigger/condition/permission
  exists (with links to GitHub Docs), not just what it does. Match this style — comments
  reference the relevant GitHub Actions docs page inline.
- **Permissions**: workflows request the minimum of `contents: read`/`contents: write`
  needed to check out and update `README.md` / `-step.txt`. Don't broaden this.
- **README.md** is generated/overwritten by the workflows via `action-update-step`. Don't
  hand-edit the step-specific body content directly — edit the corresponding
  `.github/steps/N-*.md` file instead, since the next workflow run will otherwise clobber
  manual README changes.
- **Images**: lesson screenshots live flat in `images/`, referenced with root-relative paths
  (e.g. `/images/code-tab.png`).
- Keep step numbering (`0`, `1`, `2`, `3`, `4`, `X`) and the `from_step`/`to_step` pairs in
  workflows consistent if steps are ever added, removed, or reordered — the gating logic
  depends on exact matches.

## Practical guidance for AI assistants

- There's no build/test/lint command to run — verifying a change means reasoning through
  the workflow YAML and step-content files, since there's no way to execute the full
  GitHub-driven flow locally.
- If asked to "add a step," update: the new `.github/steps/N-*.md` file, a new
  `.github/workflows/N-*.yml` file (copy the pattern from an existing one), and the
  `to_step`/`from_step` values in the adjacent workflows so the chain stays connected.
- This repo is meant to be used as a GitHub template (`is_template` checks assume that);
  changes should keep working correctly both in the template repo and in repos generated
  from it.
