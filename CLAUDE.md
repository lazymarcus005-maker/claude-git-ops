# GitOps Agents — Claude Code Native

ระบบ agent สำหรับวิเคราะห์ Feature Request และสร้าง GitLab Milestone + Issues อัตโนมัติ
โดยอ่านจาก `SRS.md` และ `request/{service}/{feature}/overview.md`

---

## Flow ภาพรวม

```
Branch: req-00xxxx
    │
    ├── SRS.md                              ← entry point (services affected + mapping)
    └── request/
        ├── {service-a}/{feature}/
        │   └── overview.md                 ← API spec + dependencies + diagrams
        └── {service-b}/{feature}/
            └── overview.md
```

เมื่อมี branch `req-00xxxx` พร้อมเอกสาร ให้รัน:

```bash
/gitops-run          # full pipeline: analyze → confirm → create GitLab issues
```

หรือแยก step:

```bash
/analyze-srs         # วิเคราะห์ SRS + overview.md → บันทึก memory
/create-issues       # สร้าง GitLab Milestone + Issues จาก memory
/gitops-status       # ดูสถานะ analysis ปัจจุบัน
```

---

## Setup

### 1. เชื่อมต่อ GitLab MCP (Docker)

```bash
cd docker/
cp .env.example .env
# แก้ไข .env ใส่ token + URL
docker compose up -d
```

ดู [`docker/HOW_TO_CONNECT.md`](docker/HOW_TO_CONNECT.md) สำหรับรายละเอียดเต็ม

### 2. ตั้งค่า Claude Code

`.claude/settings.json` ถูกตั้งค่าไว้แล้ว อัปเดต environment variables:

```bash
export GITLAB_TOKEN="glpat-xxxxxxxxxxxxxxxxxxxx"
export GITLAB_URL="https://gitlab.yourcompany.com"
```

### 3. สร้าง Feature Request Branch

```bash
git checkout -b req-000001
cp templates/SRS.md SRS.md
# แก้ไข SRS.md
mkdir -p request/payment-service/add-payment
cp templates/overview.md request/payment-service/add-payment/overview.md
# แก้ไข overview.md
git add . && git commit -m "feat: add SRS for payment feature"
```

---

## Agents & Subagents

### Orchestrator (`.claude/commands/gitops-run.md`)
Main pipeline — อ่าน SRS, spawn subagents ขนาน, รวม output, สร้าง issues

### Subagents (`agents/`)

| Agent | หน้าที่ |
|-------|--------|
| `srs-analyzer` | Parse SRS.md → extract services + mapping |
| `overview-parser` | Parse overview.md → extract API spec + deps |
| `issue-creator` | รับ analysis JSON → สร้าง GitLab milestone + issues |

### Memory (`.claude/memory/analyses/`)

บันทึก analysis ต่อ branch เป็น JSON:

```
.claude/memory/analyses/req-000001.json
```

---

## โครงสร้างเอกสาร

### SRS.md
ดู [`templates/SRS.md`](templates/SRS.md)

| Section | เนื้อหา |
|---------|--------|
| Services Affected | list of services + change type |
| Service Mapping | service → repo → overview path |
| Dependencies | upstream/downstream services |

### overview.md
ดู [`templates/overview.md`](templates/overview.md)

| Section | เนื้อหา |
|---------|--------|
| API Specification | endpoint, method, auth |
| Request/Response | JSON schemas |
| Dependencies | internal services + external repos |
| Database Changes | migrations, new tables |
| Diagrams | sequence, ER (Mermaid) |

---

## GitLab Output

ต่อ 1 Feature Request (`req-00xxxx`):

```
GitLab Milestone: [req-000001] Add Payment Feature
├── Issue: [payment-service] API Addition: Add Payment Feature
├── Issue: [notification-service] Event Consumer: Add Payment Feature
└── Issue: [api-gateway] Route Addition: Add Payment Feature
```

แต่ละ Issue มี:
- Description จาก overview.md
- API endpoints list
- Acceptance criteria
- Dependencies noted
- Link ไปยัง overview.md
- Labels: service name + change type
