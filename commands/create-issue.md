# Create GitHub Issue Command

IMPORTANT: Execute all steps autonomously without pausing for input. Do not use EnterPlanMode.

Create a detailed, well-structured GitHub issue on the current repository.

## Input

`$ARGUMENTS` — A brief description of the issue to create. This can be a short phrase, a sentence, or a rough idea. You will expand it into a fully detailed issue.

If `$ARGUMENTS` is empty, examine recent git log, open TODOs, and the current state of the codebase to suggest what issue to create, then proceed with the most relevant one.

## Pre-flight Checks

1. Verify the current directory is a git repository with a GitHub remote.
2. Confirm `gh` CLI is authenticated (`gh auth status`).
3. Detect the repository owner/name from the remote URL.
4. Fetch existing labels (`gh label list --limit 100`) so you only apply labels that exist.
5. Fetch the 10 most recent open issues (`gh issue list --limit 10`) to avoid creating duplicates.

## Issue Research

Before writing the issue, gather context:

1. **Search the codebase** for files, functions, and code related to `$ARGUMENTS` to understand the current state.
2. **Check for related issues** — search open and closed issues (`gh issue list --search "$ARGUMENTS" --state all --limit 10`) to find prior discussion or duplicates.
3. **Identify affected files** — list specific files, modules, or components that are relevant.
4. **Understand the scope** — determine if this is a bug, feature request, enhancement, refactor, documentation, or chore.

## Issue Classification

Based on your research, classify the issue:

| Type | Title Prefix | Description |
|------|-------------|-------------|
| Bug | 🐛 | Something is broken or behaving incorrectly |
| Feature | ✨ | Net-new functionality |
| Enhancement | 🔧 | Improvement to existing functionality |
| Refactor | ♻️ | Code restructuring with no behavior change |
| Documentation | 📝 | Documentation additions or corrections |
| Chore | 🧹 | Maintenance, dependencies, tooling |

## Issue Body Template

Generate the issue body using this structure. Omit any section that is not applicable — do not include empty sections.

```markdown
## Summary

<1-2 concise paragraphs describing what this issue is about and why it matters.>

## Current Behavior

<What happens today. Include code snippets, error messages, or screenshots references if relevant.>

## Expected Behavior

<What should happen instead. Be specific and concrete.>

## Detailed Description

<Expanded technical details. Include:>
- Context and background
- How this fits into the broader system
- Any constraints or considerations

## Proposed Approach

<Suggested implementation strategy, broken into steps:>
1. Step one
2. Step two
3. Step three

<Include alternative approaches considered, if any.>

## Affected Areas

- **Files**: `path/to/file.ts`, `path/to/other.ts`
- **Components/Modules**: <list affected areas>
- **Dependencies**: <any external dependencies involved>

## Acceptance Criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Additional Context

<Any other relevant information: links, related issues, screenshots, logs, references.>
```

## Labels

Apply labels that **already exist** on the repository. Match based on issue type:

- Bug → `bug`
- Feature → `enhancement` or `feature`
- Documentation → `documentation`
- Refactor / Chore → `chore` or `maintenance`
- Also apply area-specific labels if they exist (e.g., `frontend`, `api`, `database`)

Do **not** create new labels.

## Creation

1. **Create the issue** using `gh issue create`. You **must** always set the `--type` flag based on the issue classification:

   | Classification | `--type` value |
   |----------------|---------------|
   | Bug | `Bug` |
   | Feature | `Feature` |
   | Enhancement | `Feature` |
   | Refactor | `Task` |
   | Documentation | `Task` |
   | Chore | `Task` |

   ```bash
   gh issue create --title "<title>" --body "<body>" --label "<label1>,<label2>" --type "<type>"
   ```
   Use a HEREDOC for the body to preserve formatting:
   ```bash
   gh issue create --title "✨ Add user session timeout" --body "$(cat <<'EOF'
   <full issue body here>
   EOF
   )" --type "Feature"
   ```

2. **Assign the issue** to the current user (`gh issue edit <number> --add-assignee @me`). If assignment fails (e.g., not a collaborator), skip silently.

3. **Link related issues** — if related issues were found during research, mention them in the Additional Context section with `Related: #<number>`.

## Output

After creating the issue, display:
- Issue number and URL
- Title
- Labels applied
- A one-line summary of what was created
