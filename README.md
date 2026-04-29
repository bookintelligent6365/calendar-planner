# Johny — Personal Secretary (Morning Brief)

ระบบ AI เลขาส่วนตัวที่รันทุกเช้า ดึงข้อมูลจาก Google Calendar + Notion + Digital Asset news แล้วส่งสรุปผ่าน LINE OA

## Features

- **Daily Morning Brief** (จันทร์-ศุกร์ ตี 5) — ตาราง + Digital Asset tasks + พลังบวก
- **Saturday Weekly Planning** (เสาร์ 08:00) — วางแผนสัปดาห์ + สร้าง Calendar events + reminder 15 นาที

---

## Setup Guide

### 1. Prerequisites

- [Claude Code CLI](https://claude.ai/code) — `npm install -g @anthropic-ai/claude-code`
- GitHub account + repository
- LINE OA account (Messaging API enabled)
- Google Calendar MCP configured

### 2. Clone & Init

```bash
git clone https://github.com/YOUR_USERNAME/johny-morning-brief.git
cd johny-morning-brief
```

### 3. GitHub Secrets

ไปที่ **Settings → Secrets and variables → Actions** แล้วเพิ่ม:

| Secret Name | ค่า | หมายเหตุ |
|-------------|-----|----------|
| `ANTHROPIC_API_KEY` | `sk-ant-...` | จาก console.anthropic.com |
| `LINE_CHANNEL_ACCESS_TOKEN` | `...` | จาก LINE Developers Console |
| `LINE_USER_ID` | `Ud06dd...` | User ID ที่จะรับข้อความ |

### 4. Google Calendar MCP

ตรวจสอบว่า MCP server `fcd84069-fb7a-414a-96d9-693d6c7eecf4` ใน Claude Code ของคุณเชื่อมต่อ Google Calendar ของ `siripong.ro@gmail.com` แล้ว

ทดสอบใน Claude Code:
```
list_events calendar: siripong.ro@gmail.com date: today
```

### 5. Notion MCP

ตรวจสอบว่า MCP server `0d253133-1396-44b2-aad2-f0e39d5fd171` เชื่อมต่อ Notion workspace แล้ว

Long-Form Video Workflow page ต้องเข้าถึงได้ผ่าน MCP

### 6. LINE OA Setup (ถ้ายังไม่มี)

1. สร้าง account ที่ [LINE Developers](https://developers.line.biz)
2. สร้าง Provider → Channel (Messaging API)
3. ออก **Channel Access Token** (Long-lived)
4. ดู **Your user ID** ใน Basic Settings
5. Enable Webhooks (optional สำหรับ approval flow)

ทดสอบ LINE OA:
```bash
curl -X POST https://api.line.me/v2/bot/message/push \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"to":"YOUR_USER_ID","messages":[{"type":"text","text":"Johny test 🤖"}]}'
```

---

## ไฟล์สำคัญ

```
├── morning_brief.md          # Prompt สำหรับ Daily brief
├── weekly_plan.md            # Prompt สำหรับ Saturday weekly plan
├── reference/
│   ├── sources.md            # แหล่งข่าว Digital Asset
│   └── perspectives.md       # Framework 3 มุมมอง
├── articles/                 # Brief ที่ generate แล้ว (git history)
├── .env.example              # Template env vars
└── .github/workflows/
    ├── morning-brief.yml     # Cron: จันทร์-ศุกร์ ตี 5
    └── weekly-plan.yml       # Cron: เสาร์ 08:00
```

Skill หลักอยู่ที่: `~/.claude/skills/johny-morning-brief/SKILL.md`

---

## รัน Manual

```bash
# Daily brief (วันนี้)
claude -p "$(cat morning_brief.md)"

# Daily brief (วันที่ระบุ)
claude -p "$(cat morning_brief.md) Date: 2026-05-01"

# Weekly planning
claude -p "$(cat weekly_plan.md)"
```

---

## Schedule

| Routine | Cron | เวลาไทย | วัน |
|---------|------|---------|-----|
| Morning Brief | `0 22 * * 0-4` | 05:00 | จันทร์-ศุกร์ |
| Weekly Planning | `0 1 * * 6` | 08:00 | เสาร์ |

---

## Troubleshooting

**Calendar ไม่ response:**
- ตรวจว่า MCP server ทำงานอยู่: `claude mcp list`
- Johny จะ abort ทันทีถ้า Calendar MCP ไม่มี (ไม่สร้างข้อมูลปลอม)

**LINE ส่งไม่ได้:**
- ตรวจ Token ใน GitHub Secrets
- ดู log ใน Actions tab
- Johny จะ log error และหยุด ไม่ retry อัตโนมัติ

**Weekly plan สร้าง event ทับกัน:**
- ไม่เกิด — Johny ตรวจสอบ existing events ก่อนสร้างทุกครั้ง
