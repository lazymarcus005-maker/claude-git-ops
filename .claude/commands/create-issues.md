# /create-issues — Create GitLab Milestone and Issues

Create a GitLab Milestone (1 per feature request) and Issues (1 per affected service)
from the analysis saved by `/analyze-srs`.

## Prerequisites
- Analysis must exist at `.claude/memory/analyses/{branch-name}.json`
- GitLab MCP server must be connected (check `mcp__gitlab__*` tools)

## Steps

### 1. Load Analysis
Read `.claude/memory/analyses/{branch-name}.json`.
If not found, tell user to run `/analyze-srs` first.

### 2. Check for Existing GitLab Items
If `gitlab.milestone_id` is already set in memory, ask:
"Milestone already exists at {url}. Do you want to add more issues, or start fresh?"

### 3. Gather GitLab Project Info
If `gitlab.project_path` is not in memory, ask user:
"Enter the GitLab project path where issues should be created (e.g., company-group/backend-services):"

Save to `.claude/memory/project.json` for reuse.

### 4. Spawn Issue Creator Subagent
Use the Agent tool with the prompt from `agents/issue-creator.md`.

Pass:
- Analysis JSON (full content)
- GitLab project path
- Optional: milestone due date

The subagent will:
1. Create the milestone
2. Create one issue per service
3. Return a summary with IDs and URLs

### 5. Update Memory
Write results to `.claude/memory/analyses/{branch-name}.json`:
```json
{
  "gitlab": {
    "project_path": "company-group/backend",
    "milestone_id": 42,
    "milestone_url": "https://gitlab.../milestones/42",
    "created_at": "2026-05-29T10:00:00Z",
    "issues": [
      {
        "iid": 101,
        "service": "payment-service",
        "title": "[payment-service] API Addition: Add Payment Feature",
        "url": "https://gitlab.../issues/101"
      }
    ]
  }
}
```

### 6. Output Report
```
✓ Milestone: [req-000001] Add Payment Feature
  https://gitlab.yourcompany.com/.../milestones/42

✓ Issues created (3):
  #101 [payment-service] API Addition         → https://...
  #102 [notification-svc] Event Consumer      → https://...
  #103 [api-gateway] Route Addition ⚠ missing-overview
```
