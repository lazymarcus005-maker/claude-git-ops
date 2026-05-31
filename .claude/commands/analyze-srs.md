# /analyze-srs — Analyze SRS and Overview Documents

Analyze the SRS.md and all linked overview.md files for the current feature request branch.
Saves results to `.claude/memory/analyses/{branch-name}.json`.

## Steps

### 1. Get Branch Name
```bash
git branch --show-current
```
Extract the feature ID: `req-000001` → ID = `000001`.

### 2. Read SRS.md
Read `SRS.md` from the repo root. If not found, show the user `templates/SRS.md` and explain required sections.

### 3. Spawn SRS Analyzer Subagent
Use the Agent tool with the system prompt from `agents/srs-analyzer.md`.

Pass the full content of SRS.md as input. The subagent returns:
```json
{
  "branch": "req-000001",
  "feature_id": "000001",
  "title": "...",
  "summary": "...",
  "services": [
    {
      "name": "payment-service",
      "repository": "...",
      "change_type": "...",
      "overview_path": "request/payment-service/add-payment/overview.md"
    }
  ],
  "dependencies": {
    "upstream": [],
    "downstream": []
  }
}
```

### 4. Spawn Overview Parser Subagents (parallel)
For each service with a non-null `overview_path`:
- Verify the file exists
- Use the Agent tool with the prompt from `agents/overview-parser.md`
- Pass the overview_path and file content

Merge results back into each service object.

### 5. Save to Memory
Write the complete analysis to `.claude/memory/analyses/{branch-name}.json`.

### 6. Output Summary
```
Analysis complete for branch: req-000001
Feature: {title}

Services affected ({n}):
  ✓ payment-service    — API Addition     (overview: found)
  ✓ notification-svc   — Event Consumer   (overview: found)
  ⚠ api-gateway        — Route Addition   (overview: MISSING)

Saved to: .claude/memory/analyses/req-000001.json

Run /create-issues to create GitLab milestone and issues.
```
