# Create Pull Request Command

IMPORTANT: Do not ask the user any questions. Do not prompt for confirmation. Execute all steps immediately and autonomously without pausing for input. Do not use EnterPlanMode or AskUserQuestion.

Create a new branch, commit changes, and submit a pull request.

## Pre-flight Checks

Before starting, verify:
- There are actual changes to commit (staged, unstaged, or untracked files). If no changes exist, stop and inform the user.
- Identify the default branch (`main` or `master`) by checking `git symbolic-ref refs/remotes/origin/HEAD` or falling back to checking which exists.
- Ensure the local default branch is up to date with the remote (`git fetch origin`).

## Branch Naming

Create a branch from the default branch using this convention:
- `feat/<short-description>` — for new features or functionality
- `fix/<short-description>` — for bug fixes
- `refactor/<short-description>` — for code restructuring
- `chore/<short-description>` — for config, deps, tooling, or non-code changes
- `docs/<short-description>` — for documentation-only changes

Determine the prefix by analyzing the nature of the changes. Use kebab-case for the description (e.g., `feat/add-blind-tasting-mode`, `fix/score-calculation-error`).

If `$ARGUMENTS` is provided, use it to inform the branch name and PR context. If it contains a GitHub issue number (e.g., `#123` or just `123`), incorporate it into the branch name (e.g., `feat/123-add-blind-tasting-mode`).

## Behavior

1. **Create branch** from the default branch using the naming convention above
2. **Format modified files** using Biome. If Biome formatting fails:
   - Attempt to fix auto-fixable issues with `biome check --fix`
   - If errors persist, proceed with the remaining files and note formatting issues in the PR description
3. **Analyze changes** and automatically split into logical commits when appropriate
4. **Create commits** using conventional commit format (see below)
5. **Push branch** to remote with `-u` flag. If push is rejected:
   - Pull with rebase and retry
   - If conflicts arise, stop and inform the user
6. **Create pull request** using the PR template below
7. **Link issues** — if `$ARGUMENTS` contains an issue number, add `Closes #<number>` to the PR body. If the changes relate to but don't fully resolve an issue, use `Relates to #<number>` instead.
8. **Add labels** — use `gh pr edit --add-label` to apply a label matching the branch prefix (`feature`, `bug`, `refactor`, `chore`, `documentation`). Only add labels that already exist on the repo; do not create new ones.
9. **Assign reviewers** — if a `CODEOWNERS` file exists, let GitHub auto-assign. Otherwise skip reviewer assignment.
10. **Verify** — after PR creation, run `gh pr checks` to confirm CI was triggered. Note the status in your output to the user.
11. **Cleanup** — switch back to the default branch, then delete the local feature branch (`git branch -d <branch>`).
12. **Output** — display the PR URL to the user.

## Commit Message Format

Use conventional commits:
```
<type>(<scope>): <short description>

<optional body explaining what and why>
```

Types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `style`, `perf`

Scope is optional but encouraged — use the component, module, or area of the codebase (e.g., `feat(auth): add session timeout handling`).

Rules:
- Commits should never include `Co-Authored-By` trailers
- Keep the first line under 72 characters
- Use imperative mood ("add", "fix", "update", not "added", "fixed", "updated")

## Guidelines for Automatic Commit Splitting

- Split commits by feature, component, or concern
- Keep related file changes together in the same commit
- Separate refactoring from feature additions
- Separate test additions from implementation code
- Ensure each commit can be understood independently
- Multiple unrelated changes should be split into separate commits
- Config/dependency changes should be their own commit

## PR Description Template

Use this exact structure:

```
## Summary

<2-4 bullet points describing what this PR does and why>

## Changes

<Bulleted list of specific changes grouped by area, e.g.:>
- **component/area**: description of change
- **component/area**: description of change

## Test Plan

<Bulleted checklist of how to verify the changes, e.g.:>
- [ ] Step-by-step verification instructions
- [ ] Edge cases to check
- [ ] Any automated tests added or modified
```

Rules:
- PR title should be concise (under 72 characters), in imperative mood, matching the primary change
- PR title should follow conventional commit style without the type prefix (e.g., "Add blind tasting mode" not "feat: Add blind tasting mode")
- PR description should never include "Generated with Claude Code" or similar attribution
- Always create a new PR, do NOT update an older PR
