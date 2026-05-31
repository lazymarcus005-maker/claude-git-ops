# /gitops-status — Show Current Feature Request Status

Show the analysis and GitLab creation status for the current branch.

## Steps

### 1. Get Branch
```bash
git branch --show-current
```

### 2. Check Memory
Read `.claude/memory/analyses/{branch-name}.json`.

### 3. Output Status

**If no analysis found:**
```
No analysis found for branch: req-000001

To start:
  /analyze-srs    — analyze SRS.md and overview files
  /gitops-run     — full pipeline (analyze + create GitLab items)
```

**If analysis found but no GitLab items:**
```
Branch: req-000001
Feature: {title}
Analyzed: {analyzed_at}

Services ({n} affected):
  ✓ payment-service    — API Addition
  ✓ notification-svc   — Event Consumer
  ⚠ api-gateway        — overview.md missing

GitLab: not created yet
  Run /create-issues to create milestone and issues.
```

**If GitLab items created:**
```
Branch: req-000001
Feature: {title}
Analyzed: {analyzed_at}

GitLab Milestone: #42 [req-000001] {title}
  https://gitlab.yourcompany.com/.../milestones/42

Issues ({n}/{total}):
  #101 ✓ [payment-service] API Addition
  #102 ✓ [notification-svc] Event Consumer
  #103 ✓ [api-gateway] Route Addition  (⚠ missing-overview)

Created: {created_at}
```
