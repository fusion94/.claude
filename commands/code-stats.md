Generate a comprehensive code statistics report for all git repositories in the current directory.

IMPORTANT: Do not ask the user any questions. Do not prompt for confirmation. Execute all steps immediately and autonomously without pausing for input. Do not use EnterPlanMode or AskUserQuestion.

## Instructions

1. Find all directories containing `.git` folders

2. For each git repository, gather all of the following data since $ARGUMENTS (default: "2026-01-01" if not specified):

### Core Metrics
   - Number of commits (using `git log --oneline`)
   - Lines of code added and removed (using `git log --numstat`)
   - For GitHub repos, count merged PRs using `gh pr list --state merged --search "merged:>=DATE"`

### Test vs Application Code
   - Separate line counts into **test code** vs **application code** by classifying each file path from `--numstat` output. Test files match any of these patterns:
     - Directories: `test/`, `tests/`, `__tests__/`, `spec/`, `testing/`
     - File suffixes: `_test.go`, `.test.js`, `.test.ts`, `.test.jsx`, `.test.tsx`, `.spec.js`, `.spec.ts`, `.spec.jsx`, `.spec.tsx`, `_spec.rb`, `_test.py`, `_test.rb`
     - File prefixes: `test_*.py`
     - Files named exactly: `conftest.py`, `setup_test.go`
   - Everything else is application code

### Activity & Velocity
   - Week-by-week commit counts for the period (using `git log --format="%aI"` and grouping by ISO week)
   - Average commits per day and per week
   - Most active days of the week (using `git log --format="%ad" --date=format:"%A"`)
   - Most active time-of-day buckets: Morning (6-12), Afternoon (12-18), Evening (18-24), Night (0-6) (using `git log --format="%ad" --date=format:"%H"`)
   - Longest streak of consecutive days with at least one commit

### Code Quality Signals
   - Average PR size in lines changed (total lines added+removed / number of merged PRs) — use `gh pr list --json additions,deletions`
   - Commit-to-PR ratio (commits / merged PRs)
   - Churn rate: identify files that were both added to and removed from in the period (lines added then later removed = rework). Use `git log --numstat` to find files with both additions and deletions across multiple commits
   - File type breakdown by lines added (group by extension: `.ts`, `.py`, `.go`, `.js`, `.css`, `.html`, `.md`, etc.)

### Collaboration (GitHub repos only)
   - PR review turnaround time: average time from PR creation to merge (using `gh pr list --json createdAt,mergedAt`)
   - Average number of comments per PR (using `gh pr list --json comments`)
   - Number of unique PR reviewers per repo (using `gh pr list --json reviews`)
   - Bus factor per repo: number of distinct commit authors

### Repo Health (GitHub repos only)
   - Date of most recent commit per repo
   - Open issues count (using `gh issue list --state open --limit 1 --json number` or `gh api repos/OWNER/REPO`)
   - Open PR count (using `gh pr list --state open --json number`)
   - Number of branches (using `git branch -r | wc -l`)

3. Create a markdown report with the following sections in this order:

### Report Structure

   1. **Executive Summary** — A 2-3 sentence paragraph highlighting the most important takeaways (most active project, notable trends, any concerns)

   2. **Summary Table** — Totals for commits, lines added, lines removed, net lines, merged PRs

   3. **Test vs Application Code Breakdown** — Table showing lines added/removed for each category with percentages. Include a second adjusted table excluding any repos with bulk imports (>500K lines)

   4. **Weekly Velocity** — Table with one row per week showing commit count and PR count. Include a Mermaid bar chart visualizing weekly commits:
      ```mermaid
      gantt or xychart-beta
      ```

   5. **Activity Patterns** — Tables for day-of-week distribution and time-of-day distribution. Include Mermaid `xychart-beta` bar charts for both (day-of-week in Mon-Sun order, time-of-day in chronological order). Include commit streak info.

   6. **Active Repositories** — Detailed breakdown table sorted by commits descending, with columns: Repository, Commits, App Lines Added, Test Lines Added, Total Added, Total Removed, Net Change, Merged PRs

   7. **Test Coverage by Repository** — Table for repos with test activity showing test %, sorted by test % descending

   8. **Code Quality Signals** — Table per active repo showing: avg PR size, commit-to-PR ratio, file types breakdown. Flag repos where avg PR size > 500 lines as potentially needing smaller PRs.

   9. **Collaboration Metrics** — Table showing per-repo: avg PR turnaround time, avg comments per PR, unique reviewers, bus factor

   10. **Repo Health Dashboard** — Table for all repos showing: last commit date, open issues, open PRs, branch count. Flag repos with >5 stale branches or >10 open issues.

   11. **File Type Distribution** — Table showing lines added by file extension across all repos, sorted by lines added descending. Include a Mermaid pie chart.

   12. **Inactive Repositories** — List with GitHub org/owner

   13. **Top Contributors** — By commits, by lines added, by merged PRs

   14. **Most Active Projects** — Top 3 by combined activity with test % noted

   15. **Changes Since Last Report** — If a previous `code-stats.md` exists, read it first and include a delta section comparing key metrics (commits, PRs, lines) to the previous run. Show the change as +/- values.

   16. **Notes** — Methodology notes, caveats, generation date

4. Save the report to `code-stats.md` in the current directory

5. Open the file when complete in a new VS Code window using `code -n code-stats.md`

## Output Format

- Use clean markdown tables and include percentages where relevant
- Note any repositories with unusually large line counts that may indicate bulk imports or generated files
- Use Mermaid chart syntax for visualizations (GitHub and VS Code both render these)
- Format durations as human-readable (e.g., "2h 15m", "1.5 days")
- Round percentages to one decimal place
