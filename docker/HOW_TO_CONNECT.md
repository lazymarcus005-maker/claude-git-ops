# เชื่อมต่อ GitLab MCP กับ Claude Code

ระบบนี้ใช้ [MCP (Model Context Protocol)](https://modelcontextprotocol.io) เพื่อให้ Claude Code
สามารถสร้าง Milestone และ Issue ใน GitLab ได้โดยตรง

---

## ขั้นตอนการ Setup

### 1. สร้าง GitLab Personal Access Token

1. ไปที่ `https://gitlab.yourcompany.com/-/user_settings/personal_access_tokens`
2. สร้าง token ใหม่ ตั้งชื่อว่า `claude-gitops-agent`
3. เลือก scopes: **api**, **read_repository**, **write_repository**
4. กด "Create personal access token"
5. **คัดลอก token ทันที** (จะแสดงแค่ครั้งเดียว)

### 2. ตั้งค่า Environment Variables

```bash
cd docker/
cp .env.example .env
```

แก้ไข `.env`:
```env
GITLAB_TOKEN=glpat-xxxxxxxxxxxxxxxxxxxx
GITLAB_URL=https://gitlab.yourcompany.com
```

### 3. Build Docker Image

```bash
cd docker/
docker compose build
```

ตรวจสอบ:
```bash
docker images | grep gitops-gitlab-mcp
# gitops-gitlab-mcp   latest   abc123   ...
```

### 4. ตั้งค่า Claude Code

`.claude/settings.json` ถูกตั้งค่าไว้แล้ว แต่ต้อง export env vars ก่อนรัน Claude Code:

```bash
export GITLAB_TOKEN="glpat-xxxxxxxxxxxxxxxxxxxx"
export GITLAB_URL="https://gitlab.yourcompany.com"
claude  # เปิด Claude Code
```

หรือเพิ่มใน shell profile (`~/.bashrc` / `~/.zshrc`):
```bash
export GITLAB_TOKEN="glpat-xxxxxxxxxxxxxxxxxxxx"
export GITLAB_URL="https://gitlab.yourcompany.com"
```

### 5. ทดสอบการเชื่อมต่อ

เมื่อ Claude Code เปิดขึ้น ทดสอบด้วย:
```
ลอง list projects ใน GitLab ให้หน่อย
```

Claude ควรเรียก `mcp__gitlab__list_projects` และแสดงผลได้

---

## วิธีการทำงาน (Technical)

```
Claude Code (CLI)
    │
    │ spawn subprocess (stdio)
    ▼
Docker Container: gitops-gitlab-mcp
    │
    │ HTTPS API calls
    ▼
Private GitLab Instance
(https://gitlab.yourcompany.com/api/v4)
```

Claude Code รัน Docker container เป็น subprocess และสื่อสารผ่าน **stdin/stdout** (stdio transport)
ไม่มี port ที่ expose ออกมา ทุกอย่างอยู่ใน process communication

---

## การ Update Settings สำหรับ Docker Compose Path ต่างๆ

ถ้า repo อยู่ใน path ที่ต่างออกไป แก้ `.claude/settings.json`:

```json
{
  "mcpServers": {
    "gitlab": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-e", "GITLAB_PERSONAL_ACCESS_TOKEN",
        "-e", "GITLAB_API_URL",
        "gitops-gitlab-mcp:latest"
      ],
      "env": {
        "GITLAB_PERSONAL_ACCESS_TOKEN": "${GITLAB_TOKEN}",
        "GITLAB_API_URL": "${GITLAB_URL}/api/v4"
      }
    }
  }
}
```

---

## Alternative: SSE Mode (สำหรับ Team หรือ CI/CD)

ถ้าต้องการให้ MCP server รันตลอดเวลาเป็น HTTP service แทน stdio:

### docker-compose.sse.yml (เพิ่มเติม)

```yaml
services:
  gitlab-mcp-sse:
    image: gitops-gitlab-mcp:latest
    command: ["--transport", "sse", "--port", "3001"]
    environment:
      - GITLAB_PERSONAL_ACCESS_TOKEN=${GITLAB_TOKEN}
      - GITLAB_API_URL=${GITLAB_URL}/api/v4
    ports:
      - "3001:3001"
    restart: unless-stopped
```

```bash
docker compose -f docker-compose.sse.yml up -d
```

แก้ `.claude/settings.json` ให้ใช้ SSE:

```json
{
  "mcpServers": {
    "gitlab": {
      "type": "sse",
      "url": "http://localhost:3001/sse"
    }
  }
}
```

---

## Troubleshooting

| ปัญหา | แนวทางแก้ |
|-------|----------|
| `mcp__gitlab__*` tools ไม่ปรากฏ | ตรวจสอบว่า Docker image build แล้ว (`docker images`) |
| `401 Unauthorized` | Token หมดอายุหรือ scope ไม่ครบ สร้างใหม่ |
| `SSL certificate error` | GitLab ใช้ self-signed cert: เพิ่ม `NODE_TLS_REJECT_UNAUTHORIZED=0` ใน env (dev only) |
| Container ไม่ start | `docker run --rm -i -e GITLAB_PERSONAL_ACCESS_TOKEN=xxx -e GITLAB_API_URL=xxx/api/v4 gitops-gitlab-mcp:latest` ทดสอบ manual |
| Connection timeout | ตรวจสอบ network policy ระหว่าง Docker host กับ GitLab server |

---

## Security Notes

- อย่า commit `.env` ลงใน Git (ถูก gitignore แล้ว)
- ใช้ token ที่มี expiry date
- ใน production ใช้ secrets manager (Vault, AWS Secrets Manager) แทน env vars
- Token ควร read/write เฉพาะ project ที่จำเป็น ไม่ใช้ admin scope
