# Johny — Personal Secretary (Morning Brief)

ระบบ AI เลขาส่วนตัวที่รันทุกเช้า ดึงข้อมูลจาก Google Calendar + Digital Asset news แล้วส่งสรุปผ่าน LINE OA

## Architecture (Hybrid — Free)

```
[Claude Routine] (รัน 5am ทุกวัน, ใช้ subscription quota)
    ├─ ดึง Google Calendar (MCP)
    ├─ Research Digital Asset (WebSearch)
    ├─ Compose message
    └─ git commit briefs/YYYY-MM-DD.txt → push
              │
              ▼
[GitHub Actions] (trigger on push, ฟรี — ไม่ใช้ API key)
    └─ curl ส่ง LINE OA
              │
              ▼
        [LINE OA]
```

**ทำไมต้อง Hybrid:** Routine environment บล็อก outbound HTTP ไป `api.line.me` — ใช้ GitHub เป็น relay แทน

## Features

- **Daily Morning Brief** (ทุกวัน ตี 5) — ตาราง + Digital Asset tasks + พลังบวก
- **Saturday Weekly Planning** (เสาร์ 08:00) — วางแผนสัปดาห์ + สร้าง Calendar events + reminder 15 นาที

---

## Setup Guide

### 1. GitHub Secrets

ไปที่ **Settings → Secrets and variables → Actions** แล้วเพิ่ม 2 secrets:

| Secret Name | ค่า |
|-------------|-----|
| `LINE_CHANNEL_ACCESS_TOKEN` | Token จาก LINE Developers |
| `LINE_USER_ID` | LINE User ID ที่จะรับข้อความ |

**ไม่ต้องใส่ `ANTHROPIC_API_KEY`** เพราะ GitHub Actions ไม่ต้องเรียก Claude — แค่ส่ง curl ธรรมดา

### 2. Claude Remote Routine

Routine จะ commit ไฟล์ไปที่ `briefs/YYYY-MM-DD.txt` แล้ว push เข้า repo
GitHub Actions จะ trigger เมื่อมีการ push ไฟล์ใน `briefs/` แล้วส่ง LINE ทันที

ดู Routine ที่: [claude.ai/code/routines](https://claude.ai/code/routines)

---

## ไฟล์สำคัญ

```
├── briefs/                          # Brief ที่ Routine generate (auto-commit)
├── .github/workflows/
│   └── send-line.yml                # ส่ง LINE เมื่อมี push ใน briefs/
└── README.md
```

---

## Schedule

| Routine | เวลาไทย | วัน |
|---------|---------|-----|
| Daily Morning Brief | 05:00 BKK | ทุกวัน |
| Weekly Planning | 08:00 BKK | เสาร์ |

---

## ค่าใช้จ่าย

| Component | Cost |
|-----------|------|
| Claude Routine | ใช้ subscription quota (ไม่จ่ายเพิ่ม) |
| GitHub Actions | ฟรี (private repo 2,000 นาที/เดือน) |
| LINE OA | ฟรี |
| **รวม** | **ฟรี** ✨ |

---

## Troubleshooting

**LINE ไม่ได้รับข้อความ:**
- ตรวจ Secret `LINE_CHANNEL_ACCESS_TOKEN` ถูกต้องไหม
- ดู log ที่ Actions tab → workflow run
- ตรวจ User ID ถูกต้องไหม

**Routine ไม่ commit ไฟล์:**
- ตรวจว่า GitHub App ติดตั้งบน repo แล้ว
- ดู log ที่ [claude.ai/code/routines](https://claude.ai/code/routines)
