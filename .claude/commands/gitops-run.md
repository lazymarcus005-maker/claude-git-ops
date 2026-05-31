# /gitops-run — Full GitOps Pipeline

Run the complete pipeline: analyze feature request documents → create GitLab milestone + issues.

## Steps

### 1. Validate Branch
```bash
git branch --show-current
```
Branch must match `req-XXXXXX` pattern. If not, warn the user and stop.

### 2. Check Existing Analysis
Check `.claude/memory/analyses/{branch-name}.json`.
- If exists and `gitlab.milestone_id` is already set → ask user if they want to re-run or just view status.
- If exists but no milestone yet → skip to step 5 (create issues).
- If not exists → proceed to step 3.

### 3. Analyze SRS — Spawn SRS Analyzer Subagent

Use the Agent tool with this prompt (read full instructions from `agents/srs-analyzer.md`):

> Analyze the SRS.md file in the current repository. Extract the feature title, branch ID, summary, and complete service mapping. For each service in the mapping, note the overview.md path. Return a structured JSON object matching the schema in agents/srs-analyzer.md.

Save result to memory before continuing.

### 4. Analyze Overview Files — Spawn Overview Parser Subagents (parallel)

For EACH service found in step 3, use the Agent tool to spawn an `overview-parser` subagent in parallel. Use the prompt from `agents/overview-parser.md`, passing the specific overview path.

Collect all results and merge into the analysis JSON at `.claude/memory/analyses/{branch-name}.json`.

### 5. Show Analysis Summary

Display to user:
- Feature: title + branch
- Services affected (count + list with change types)
- Overview coverage (how many overview.md files were found vs missing)
- Any warnings

Ask: "Proceed to create GitLab milestone and issues? (yes/no)"

### 6. Create GitLab Items — Spawn Issue Creator Subagent

Use the Agent tool with the prompt from `agents/issue-creator.md`.

Pass:
- The full analysis JSON path
- Ask user for GitLab project path if not in memory (e.g., `company-group/backend`)
- Ask for milestone due date (optional)

### 7. Update Memory & Report

After creation, update `.claude/memory/analyses/{branch-name}.json` with:
- `gitlab.milestone_id`
- `gitlab.milestone_url`
- `gitlab.issues[]` (id + url + service)
- `gitlab.created_at`

Output final report:
```
✓ Milestone created: [req-XXXXXX] {title}
  URL: {milestone_url}

✓ Issues created ({n}):
  - [{service}] {title} → {url}
  - ...
```

## Error Handling

- **SRS.md not found**: Show path to `templates/SRS.md` and explain required format.
- **overview.md missing for a service**: Note the gap, create the issue with a warning label `missing-overview`, continue.
- **GitLab MCP not connected**: Check `mcp__gitlab__*` tools are available. If not, guide user to `docker/HOW_TO_CONNECT.md`.
- **Duplicate milestone**: Check if milestone with same title exists before creating.
