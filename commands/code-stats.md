Generate a comprehensive code statistics report for all git repositories in the current directory.

IMPORTANT: Do not ask the user any questions. Do not prompt for confirmation. Execute all steps immediately and autonomously without pausing for input. Do not use EnterPlanMode or AskUserQuestion.

## Argument Parsing

Parse `$ARGUMENTS` for the following options:
- **Start date**: A date like `2025-06-01`. Defaults to January 1 of the current year.
- **End date**: A second date after `..` or `to`, e.g. `2025-06-01..2025-12-31` or `2025-06-01 to 2025-12-31`. Defaults to today.
- **Author filter**: `--author "Name"` to scope all stats to a single contributor. Defaults to all authors.

Examples: `2025-01-01`, `2025-01-01..2025-06-30`, `2025-01-01 --author "Jane Doe"`, (empty = current year start to today)

Use `--since=<start>` and `--until=<end>` on all `git log` commands. If an author filter is provided, add `--author="<name>"` to all `git log` commands and `--author "<name>"` / `--search "author:<name>"` to `gh` commands where supported.

## Instructions

1. Find all directories containing `.git` folders

2. **Process repositories in parallel** using multiple Bash tool calls (or the Task tool for large sets). Do not process repos one at a time sequentially — maximize parallelism.

3. For each git repository, gather all of the following data for the configured date range:

### Core Metrics
   - Number of commits: `git log --oneline --no-merges --since=START --until=END`
   - Lines of code added and removed: `git log --numstat --no-merges --since=START --until=END`
   - **Filter out binary files**: `git log --numstat` outputs `-` for binary files — skip any line where additions or deletions is `-`
   - **Exclude generated/vendored files**: Ignore lines matching these paths in `--numstat` output:
     - Lock files: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `Gemfile.lock`, `Pipfile.lock`, `poetry.lock`, `composer.lock`, `go.sum`, `Cargo.lock`
     - Vendored dirs: `vendor/`, `node_modules/`, `third_party/`, `_vendor/`
     - Build output: `dist/`, `build/`, `out/`, `.next/`, `__pycache__/`
     - Generated: `*.min.js`, `*.min.css`, `*.bundle.js`, `*.generated.*`, `*.pb.go`, `*_generated.go`
   - For GitHub repos, count merged PRs using `gh pr list --state merged --search "merged:>=START" --limit 1000 --json number`
     - Use `--search ""` (no author filter) to get all PRs unless an author filter is specified
   - For GitHub repos, count issues created in the date range: `gh issue list --state all --search "created:>=START created:<=END" --limit 1000 --json number`
   - For GitHub repos, count issues closed in the date range: `gh issue list --state closed --search "closed:>=START closed:<=END" --limit 1000 --json number`
     - **If any `gh` command fails** (not a GitHub repo, auth issue, no remote, rate limit), skip all GitHub-specific metrics for that repo and note it as "GitHub data unavailable" in the report. Do not let `gh` failures stop processing.

### Test vs Application Code
   - Separate line counts into **test code** vs **application code** by classifying each file path from `--numstat` output (after filtering). Test files match any of these patterns:
     - Directories: `test/`, `tests/`, `__tests__/`, `spec/`, `testing/`
     - File suffixes: `_test.go`, `.test.js`, `.test.ts`, `.test.jsx`, `.test.tsx`, `.spec.js`, `.spec.ts`, `.spec.jsx`, `.spec.tsx`, `_spec.rb`, `_test.py`, `_test.rb`
     - File prefixes: `test_*.py`
     - Files named exactly: `conftest.py`, `setup_test.go`
   - Everything else is application code

### Activity & Velocity
   - Week-by-week commit counts for the period (using `git log --no-merges --format="%aI"` and grouping by ISO week)
   - Average commits per day and per week
   - Most active days of the week (using `git log --no-merges --format="%ad" --date=format:"%A"`)
   - Most active time-of-day buckets: Morning (6-12), Afternoon (12-18), Evening (18-24), Night (0-6) (using `git log --no-merges --format="%ad" --date=format:"%H"`)
   - Longest streak of consecutive days with at least one commit

### Code Quality Signals
   - Average PR size in lines changed (total lines added+removed / number of merged PRs) — use `gh pr list --json additions,deletions --limit 1000`
   - Commit-to-PR ratio (commits / merged PRs)
   - **Churn rate (rework indicator)**: For each file appearing in `--numstat`, count how many distinct commits touched it. Files modified in **3 or more commits** with both additions and deletions are "high-churn" files. Report the top 10 highest-churn files per repo as a rework signal.
   - File type breakdown by lines added (group by extension: `.ts`, `.py`, `.go`, `.js`, `.css`, `.html`, `.md`, etc.) — exclude generated/vendored files

### Collaboration (GitHub repos only — skip if `gh` unavailable)
   - PR review turnaround time: average time from PR creation to merge (using `gh pr list --json createdAt,mergedAt --limit 1000`)
   - Average number of comments per PR (using `gh pr list --json comments --limit 1000`)
   - Number of unique PR reviewers per repo (using `gh pr list --json reviews --limit 1000`)
   - **Weighted bus factor**: Count distinct commit authors, then calculate how many authors account for 80% of commits. Report both numbers (e.g., "3 of 8 authors cover 80% of commits").

### Issue Tracking (GitHub repos only — skip if `gh` unavailable)
   - Issues created in the date range (counted above)
   - Issues closed in the date range (counted above)
   - Issue close rate: closed / created as a percentage (>100% means backlog is shrinking)
   - Average issue resolution time: for issues closed in the range, average time from creation to close (using `gh issue list --state closed --search "closed:>=START closed:<=END" --limit 1000 --json createdAt,closedAt`)

### Repo Health (GitHub repos only — skip if `gh` unavailable)
   - Date of most recent commit per repo
   - Open issues count (using `gh issue list --state open --json number --limit 1`)
   - Open PR count (using `gh pr list --state open --json number`)
   - Number of remote branches: `git branch -r --list "origin/*" | grep -v HEAD | wc -l`

4. Create a markdown report with the following sections in this order:

### Report Structure

   1. **Executive Summary** — A 2-3 sentence paragraph highlighting the most important takeaways (most active project, notable trends, any concerns). Include the date range and any author filter applied.

   2. **Summary Table** — Totals for commits, lines added, lines removed, net lines, merged PRs, issues created, issues closed

   3. **Test vs Application Code Breakdown** — Table showing lines added/removed for each category with percentages. Include a second adjusted table excluding any repos with bulk imports (>500K lines)

   4. **Weekly Velocity** — Table with one row per week showing commit count and PR count. Include a Mermaid `xychart-beta` bar chart visualizing weekly commits.

   5. **Activity Patterns** — Tables for day-of-week distribution and time-of-day distribution. Include Mermaid `xychart-beta` bar charts for both (day-of-week in Mon-Sun order, time-of-day in chronological order). Include commit streak info.

   6. **Repository Breakdown** — Single consolidated table sorted by commits descending, with columns: Repository, Commits, App Lines +/-, Test Lines +/-, Net Change, Test %, Merged PRs, Issues Created, Issues Closed, Bus Factor. This replaces the old separate "Active Repositories", "Test Coverage by Repository", and "Most Active Projects" sections.

   7. **Code Quality Signals** — Table per active repo showing: avg PR size, commit-to-PR ratio. Flag repos where avg PR size > 500 lines. Include top high-churn files across all repos.

   8. **Collaboration Metrics** — Table showing per-repo: avg PR turnaround time, avg comments per PR, unique reviewers, weighted bus factor. Only shown for repos with GitHub data.

   9. **Issue Activity** — Table showing per-repo: issues created, issues closed, close rate %, avg resolution time. Only shown for repos with GitHub data. Flag repos where close rate < 50% as potentially falling behind on issue triage.

   10. **Repo Health Dashboard** — Table for all repos showing: last commit date, open issues, open PRs, branch count. Flag repos with >5 stale branches or >10 open issues.

   11. **File Type Distribution** — Table showing lines added by file extension across all repos, sorted by lines added descending. Include a Mermaid pie chart.

   12. **Inactive Repositories** — Table with columns: Repository, GitHub Org/Owner, Last Commit Date, Total Lines. (Inactive = zero commits in the date range.)

   13. **Top Contributors** — By commits, by lines added, by merged PRs

   14. **Changes Since Last Report** — If a previous `code-stats.json` exists, read it and include a delta section comparing key metrics (commits, PRs, lines added, lines removed, issues created, issues closed) to the previous run. Show changes as +/- values and percentage change.

   15. **Notes** — Methodology notes (including: merge commits excluded, generated/vendored files excluded, binary files excluded, list any repos where GitHub data was unavailable), caveats, generation date

5. Save two files to `/Users/guntharp/Code/fusion94/` (always use this directory regardless of the current working directory):
   - `/Users/guntharp/Code/fusion94/code-stats.md` — the human-readable report
   - `/Users/guntharp/Code/fusion94/code-stats.json` — machine-readable metrics for future delta comparisons, structured as:
     ```json
     {
       "generated": "ISO-8601 timestamp",
       "dateRange": { "start": "...", "end": "..." },
       "authorFilter": "...",
       "totals": { "commits": N, "linesAdded": N, "linesRemoved": N, "mergedPRs": N, "issuesCreated": N, "issuesClosed": N },
       "repos": { "repo-name": { "commits": N, "linesAdded": N, "linesRemoved": N, "mergedPRs": N, "testLinesAdded": N, "appLinesAdded": N, "issuesCreated": N, "issuesClosed": N } }
     }
     ```

6. Open the report when complete: try `code -n /Users/guntharp/Code/fusion94/code-stats.md`, but if the `code` command is not available, skip this step silently.

## Output Format

- Use clean markdown tables and include percentages where relevant
- Note any repositories with unusually large line counts that may indicate bulk imports or generated files
- Use Mermaid `xychart-beta` chart syntax for bar charts and `pie` for pie charts (GitHub and VS Code both render these)
- Format durations as human-readable (e.g., "2h 15m", "1.5 days")
- Round percentages to one decimal place
