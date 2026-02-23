# 🤖 Advanced SecOps AI Pipeline (WebSecScan Pro 2026)

> **AI-Powered Security Automation** - ระบบอัตโนมัติสำหรับการสแกนความปลอดภัยทางไซเบอร์ โดยใช้ AI Agent ในการควบคุมเครื่องมือทดสอบเจาะระบบ (Kali Linux) และรายงานผลผ่าน Discord

---

## 📑 สารบัญ

- [ภาพรวม](#-ภาพรวม)
- [สถาปัตยกรรมระบบ](#-สถาปัตยกรรมระบบ)
- [รายละเอียด Workflow หลัก](#-รายละเอียด-workflow-หลัก)
- [รายละเอียด Workflow ย่อย](#-รายละเอียด-workflow-ย่อย)
- [การตั้งค่าที่จำเป็น](#-การตั้งค่าที่จำเป็น)
- [ตัวอย่างการใช้งาน](#-ตัวอย่างการใช้งาน)
- [การแก้ไขปัญหา](#-การแก้ไขปัญหา)

---

## 🎯 ภาพรวม

ระบบ **Advanced SecOps AI Pipeline** เป็นระบบอัตโนมัติสำหรับการสแกนความปลอดภัยทางไซเบอร์ ที่ใช้ AI Agent ในการควบคุม Security Tools ต่างๆ เช่น Nmap, Nuclei และ Metasploit บน Kali Linux Container พร้อมสร้างรายงานผลลัพธ์เป็นภาษาไทยและส่งผ่าน Discord

### สถาปัตยกรรมระบบ

```
┌─────────────────────────────────────────────────────────┐
│                  Advanced SecOps AI Pipeline            │
│                      (Main Workflow)                    │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────┐           │
│  │  Orchestrator   │    │    Reporter     │           │
│  │     Agent       │───▶│     Agent       │           │
│  │ (Gemini 2.0)    │    │ (Gemini/Kimi)   │           │
│  └────────┬────────┘    └────────┬────────┘           │
│           │                      │                      │
│           ▼                      ▼                      │
│    ┌─────────────┐        ┌─────────────┐               │
│    │  KaliCLI    │        │   Discord   │               │
│    │   (Tool)    │        │  Webhook    │               │
│    └──────┬──────┘        └─────────────┘               │
└───────────┼─────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────┐
│              Kali Executor Tool (Sub-workflow)          │
├─────────────────────────────────────────────────────────┤
│                    SSH Node                             │
│              (Host: kali-linux)                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🔧 สถาปัตยกรรมระบบ

ระบบประกอบด้วย 2 Workflows หลักที่ทำงานร่วมกัน:

### 1. Advanced SecOps AI Pipeline (Main Workflow)

*   **ไฟล์:** `Advanced SecOps AI Pipeline.json`
*   **หน้าที่:** สมองหลักของระบบ ประกอบด้วย AI Agents 2 ตัว

**Orchestrator Agent**
- **Model**: OpenRouter (Google Gemini 2.0 Flash)
- **หน้าที่**: วางแผนการสแกน ตัดสินใจเลือกใช้เครื่องมือ (Nmap, Nuclei, Metasploit)
- **ควบคุมขั้นตอน**: Recon -> Scan -> Verify

**Reporter Agent**
- **Model**: OpenRouter (Google Gemini 2.0 Flash หรือ Moonshot Kimi)
- **หน้าที่**: สรุปผลลัพธ์ทางเทคนิคให้อยู่ในรูปแบบรายงานภาษาไทยระดับมืออาชีพ

**Integration**
- Chat Trigger (รับคำสั่ง)
- Discord (ส่งผลลัพธ์)

### 2. Kali Executor Tool (Sub-workflow)

*   **ไฟล์:** `Kali Executor Tool.json`
*   **หน้าที่:** ทำหน้าที่เป็น "แขนขา" ในการรันคำสั่งจริง

**การทำงาน:**
- รับคำสั่ง Bash Command จาก Orchestrator
- เชื่อมต่อผ่าน SSH เข้าไปที่ Kali Linux Container เพื่อรันคำสั่ง
- ส่งผลลัพธ์ (Stdout) กลับมาให้ AI

---

## 📋 รายละเอียด Workflow หลัก
| Node | รายละเอียด |
|------|------------|
| **Chat Trigger** | จุดเริ่มต้นรับคำสั่งจากผู้ใช้ (ผ่านหน้า n8n UI หรือ API) |
| **Orchestrator Agent** | Model: OpenRouter (Google Gemini 2.0 Flash) - เร็วและฉลาดพอสำหรับการวางแผน |
| **System Prompt** | ถูกปรับแต่งให้ทำงานเป็น "Lead Security Orchestrator" ตามโปรโตคอล WebSecScan Pro 2026 |
| **Tools** | เรียกใช้ `Kali Executor Tool` เพื่อรันคำสั่ง |
| **KaliCLI (Tool Node)** | จุดเชื่อมต่อไปยัง Sub-workflow |
| **JSON Parser** | แปลงผลลัพธ์การสแกนให้อยู่ในรูปแบบ JSON Structure |
| **Reporter Agent** | Model: OpenRouter (Google Gemini 2.0 Flash หรือ Moonshot Kimi) |
| **Create Report File** | โค้ด JavaScript สำหรับแปลงข้อความรายงานเป็นไฟล์ `security_report.md` |
| **Discord Alert** | ส่งไฟล์รายงานเข้าสู่ห้องแชทผ่าน Webhook |

### การไหลของข้อมูล (Data Flow)

1. **User** พิมพ์คำสั่ง -> **Orchestrator** วิเคราะห์
2. **Orchestrator** สั่งรัน Nmap/Nuclei -> **Kali Executor** -> **Kali Linux**
3. **Kali Linux** ส่งผล Scan กลับ -> **Orchestrator** รวบรวม
4. **Orchestrator** ส่ง JSON Findings -> **Reporter**
5. **Reporter** เขียนรายงานภาษาไทย -> **Create File** -> **Discord**

---

## 🔌 รายละเอียด Workflow ย่อย

**ไฟล์:** `Kali Executor Tool.json`

### การทำงาน

| Component | รายละเอียด |
|-----------|------------|
| **Execute Workflow Trigger** | รอรับการเรียกใช้งานจาก Main Workflow |
| **SSH Node** | Host: `kali-linux` (ชื่อ Service ใน Docker Compose) |
| **Command** | `{{ $json.query }}` (รับคำสั่ง Dynamic) |
| **Authentication** | SSH Password (user: `root`, pass: `kali`) |

---

## ⚙️ การตั้งค่าที่จำเป็น

เพื่อให้ระบบทำงานได้ ต้องตั้งค่า Credentials ใน n8n ดังนี้:

| Credential Name | Type | Description |
| :--- | :--- | :--- |
| **OpenRouter account** | OpenRouter API | API Key สำหรับใช้งาน AI Model (Gemini/Kimi) |
| **SSH Password account** | SSH Password | สำหรับเชื่อมต่อ Kali (User: `root`, Pass: `kali`) |
| **Discord Webhook account** | Discord Webhook | Webhook URL ของห้อง Discord ที่ต้องการรับรายงาน |

---

## 🧪 ตัวอย่างการใช้งาน

คุณสามารถนำ Prompt เหล่านี้ไปวางในช่อง Chat ของ Workflow เพื่อทดสอบระบบ:

### Scenario 1: สแกนเป้าหมายพื้นฐาน (Basic Scan)
> "ช่วยสแกนเครื่อง juice-shop-victim ที่พอร์ต 3000 ให้หน่อย ตรวจสอบ tech stack และหาช่องโหว่พื้นฐาน สรุปผลเป็นภาษาไทย"

### Scenario 2: ทดสอบช่องโหว่เจาะจง (Targeted Vulnerability)
> "ตรวจสอบ metasploitable-victim ที่พอร์ต 21 (FTP) ว่ามี Backdoor CVE-2011-2523 หรือไม่ ถ้ามีให้ลองยืนยันด้วย Metasploit check module แล้วรายงานผลความเสี่ยง"

### Scenario 3: สแกนเต็มรูปแบบตามมาตรฐาน (Full ASVS Scan)
> "เริ่มกระบวนการ WebSecScan Pro บน metasploitable-victim โดยเน้นการตรวจสอบตาม OWASP Top 10 วิเคราะห์ Service ที่เปิดอยู่ทั้งหมด และประเมินระดับความปลอดภัยตามมาตรฐาน ASVS พร้อมคำแนะนำในการแก้ไข"

---

## 🛠️ การแก้ไขปัญหา

### ปัญหาที่พบบ่อย

| ปัญหา | สาเหตุ | วิธีแก้ไข |
|-------|--------|----------|
| **Error: Workflow is not active** | `Kali Executor Tool` ยังไม่ถูก Activate | ตรวจสอบว่า `Kali Executor Tool` ถูก Activate (เปิดใช้งาน) แล้วหรือยัง |
| **Discord ไม่ส่งข้อความ** | Webhook URL ไม่ถูกต้อง | ตรวจสอบ Webhook URL และเช็คว่าโหมดส่งข้อความไม่ใช่ Bot Token |
| **AI ตอบวนลูป** | Prompt ไม่ชัดเจน หรือ AI พยายามรันคำสั่งเดิมซ้ำๆ | Orchestrator มีการจำกัด Step ไว้ที่ 3 เพื่อป้องกันปัญหานี้ |

---

<div align="center">

**⚠️ ใช้อย่างมีความรับผิดชอบ ⚠️**

*ระบบนี้ใช้สำหรับการศึกษาและฝึกอบรมเท่านั้น*

</div>
