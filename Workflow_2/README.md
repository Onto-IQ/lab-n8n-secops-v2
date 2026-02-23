# 🌐 WebSecScan Pro - Multi-Agent AI Security Scanner

> **OWASP ASVS Based Web Security Scanner** - ระบบ Multi-Agent AI ที่ทำงานบน n8n เพื่อตรวจสอบความปลอดภัยของเว็บไซต์ตามมาตรฐาน OWASP ASVS โดยใช้ Orchestrator-Workers Pattern

---

## 📑 สารบัญ

- [ภาพรวม](#-ภาพรวม)
- [สถาปัตยกรรมระบบ](#-สถาปัตยกรรมระบบ)
- [รายละเอียด Agents](#-รายละเอียด-agents)
- [การติดตั้งและใช้งาน](#-การติดตั้งและใช้งาน)
- [หมายเหตุ](#-หมายเหตุ)

---

## 🎯 ภาพรวม

ระบบ **WebSecScan Pro** เป็นระบบ Multi-Agent AI ที่ทำงานบน n8n เพื่อตรวจสอบความปลอดภัยของเว็บไซต์ตามมาตรฐาน OWASP ASVS โดยใช้ **Orchestrator-Workers Pattern** โดยมี Orchestrator เป็นผู้สั่งการ และมี Sub-Agents ที่มีความเชี่ยวชาญเฉพาะด้านทำงานเบื้องหลังผ่าน Workflow Tools

### สถาปัตยกรรมระบบ

```
┌─────────────────────────────────────────────────────────────┐
│                      User (ผู้ใช้งาน)                        │
│                  (กรอก URL ผ่าน Form)                       │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    Form Trigger                             │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              Orchestrator Agent                             │
│            (Claude 3.5 Sonnet)                              │
│         ผู้ควบคุมหลัก - วางแผนและสั่งการ                     │
└──┬──────────┬──────────┬──────────┬─────────────────────────┘
   │          │          │          │
   ▼          ▼          ▼          ▼
┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
│Recon │  │Trans-│  │Ident-│  │Corre-│
│Tool   │  │port  │  │ity   │  │lation│
│       │  │Tool   │  │Tool  │  │Tool  │
└──┬────┘  └──┬────┘  └──┬────┘  └──┬────┘
   │          │          │          │
   └──────────┴──────────┴──────────┘
                   │
                   ▼
        ┌─────────────────────┐
        │  Sub-Agent Worker   │
        │ (Claude 3.5 Sonnet) │
        │   รับบทตามคำสั่ง     │
        └──────────┬──────────┘
                   │
                   ▼
        ┌─────────────────────┐
        │   สร้างไฟล์ .md     │
        │   ส่ง Discord       │
        └─────────────────────┘
```

---

## 🔧 สถาปัตยกรรมระบบ

## 🤖 รายละเอียด Agents

ระบบประกอบด้วย Agent หลักและ Agent ย่อยดังนี้:

### Orchestrator Agent (ผู้ควบคุม)

| รายการ | รายละเอียด |
|--------|-----------|
| **หน้าที่** | รับ Input (URL), วางแผนการสแกน, เรียกใช้งาน Sub-Agents ตามลำดับ, และรวบรวมผลลัพธ์เพื่อสร้างรายงานสรุป |
| **Model** | Anthropic Claude 3.5 Sonnet (ผ่าน OpenRouter) |
| **ความสามารถหลัก** | แปลงผลลัพธ์ทางเทคนิคทั้งหมดให้เป็น **รายงานภาษาไทย** ที่เข้าใจง่ายแต่คงศัพท์เทคนิคไว้ |

### Sub-Agents (ผู้ปฏิบัติงาน)

ทำงานผ่าน **Workflow Tool** (`@n8n/n8n-nodes-langchain.toolWorkflow`) โดยส่งงานไปที่ Workflow กลาง (`WebSec_SubAgent_Worker`) ซึ่งจะสวมบทบาทตามที่ได้รับมอบหมาย:

| Agent | หน้าที่ |
|-------|---------|
| **Reconnaissance Agent** | ตรวจสอบ Tech Stack, CMS, และประเมินระดับความปลอดภัยเบื้องต้น (ASVS Level) |
| **Transport Security Agent** | ตรวจสอบความปลอดภัยของการรับส่งข้อมูล (TLS, HSTS, Headers) |
| **Identity & Access Agent** | ตรวจสอบระบบยืนยันตัวตนและการจัดการสิทธิ์ (AuthN/AuthZ) |
| **Correlation Agent** | วิเคราะห์ความเชื่อมโยงของช่องโหว่ (Attack Chain) และจัดลำดับความสำคัญ |

---

## ⚙️ การติดตั้งและใช้งาน

ระบบปัจจุบันถูกติดตั้งและพร้อมใช้งานบน n8n โดยแบ่งเป็น 2 Workflows หลัก:

### Main Orchestrator Workflow

| รายการ | รายละเอียด |
|--------|-----------|
| **ชื่อ** | `WebSecScan Pro - Workflow Tools Edition` |
| **ID** | `xcv9j8hu2oqy7LoB` |
| **Trigger** | `Form Trigger` (สร้างหน้าเว็บฟอร์มให้กรอก URL) |
| **Output** | ส่งรายงานเป็นไฟล์ **Markdown (.md)** ไปยัง Discord |
| **การเชื่อมต่อ** | ใช้ Workflow Tools ในการเชื่อมต่อไปยัง Worker |

### Sub-Agent Worker Workflow

| รายการ | รายละเอียด |
|--------|-----------|
| **ชื่อ** | `WebSec_SubAgent_Worker` |
| **ID** | `hnStxD29gy2uauhe` |
| **Trigger** | `Execute Workflow Trigger` (รอรับคำสั่งจาก Main Workflow) |
| **Logic** | เป็น AI Agent ตัวเดียวที่รับ Parameter (`role`, `task`, `target`) แล้วเปลี่ยนบทบาทไปตามคำสั่งนั้นๆ |

### วิธีการใช้งาน

1.  เปิด Workflow **"WebSecScan Pro - Workflow Tools Edition"** ใน n8n
2.  ตรวจสอบการตั้งค่า **Discord Node**:
    *   คลิกที่โหนด `Send Report to Discord`
    *   ตรวจสอบว่าเลือก **Credential** (Discord Webhook account) ถูกต้องแล้ว
3.  กดปุ่ม **Execute Workflow** ด้านล่าง
4.  คลิกที่โหนดแรก **Start Audit Form** แล้วกดปุ่ม **Test URL** (หรือ Open URL)
5.  ในหน้าเว็บที่เปิดขึ้นมา:
    *   กรอก **Target URL** (เว็บไซต์ที่ต้องการสแกน เช่น `https://example.com`)
    *   กด **Submit**
6.  รอสักครู่... ระบบจะทำการวิเคราะห์และส่งไฟล์รายงาน `WebSecScan_Report.md` (ภาษาไทย) ไปยังห้อง Discord ของคุณ

---

## 📝 หมายเหตุ

*   **Credential**: ระบบใช้ Discord Webhook Credential ที่มีอยู่แล้ว ไม่ต้องแก้ไขประเภท Authentication
*   **Output Format**: รายงานถูกตั้งค่าให้ส่งเป็นไฟล์แนบ `.md` เพื่อหลีกเลี่ยงข้อจำกัดจำนวนตัวอักษรของ Discord (2,000 chars) และเพื่อให้ได้รายงานที่ละเอียดครบถ้วนที่สุด

---

<div align="center">

**⚠️ ใช้อย่างมีความรับผิดชอบ ⚠️**

*ระบบนี้ใช้สำหรับการศึกษาและฝึกอบรมเท่านั้น*

</div>
