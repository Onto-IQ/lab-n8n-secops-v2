# 🛡️ Automated Penetration Testing & SecOps Lab with n8n

> **Lab Environment สำหรับการศึกษา** - ระบบจำลองสภาพแวดล้อมสำหรับหลักสูตร "Automated Penetration Testing & DevSecOps with n8n" ออกแบบมาเพื่อให้ผู้เรียนสามารถฝึกเขียน Workflow ควบคุม Security Tools (Nuclei, Nmap) และทดสอบเป้าหมายจำลองได้อย่างปลอดภัยและสมจริง

---

## 📑 สารบัญ

- [ภาพรวม](#-ภาพรวม)
- [คุณสมบัติหลัก](#-คุณสมบัติหลัก)
- [ความต้องการของระบบ](#-ความต้องการของระบบ)
- [โครงสร้างโปรเจกต์](#-โครงสร้างโปรเจกต์)
- [การติดตั้งและตั้งค่า](#-การติดตั้งและตั้งค่า)
- [การใช้งาน](#-การใช้งาน)
- [Wazuh SIEM](#-wazuh-siem)
- [SecOps Demo Workflow](#-secops-demo-workflow)
- [การตรวจสอบระบบ](#-การตรวจสอบระบบ)
- [การแก้ไขปัญหา](#-การแก้ไขปัญหา)
- [เอกสารอ้างอิง](#-เอกสารอ้างอิง)
- [ข้อกำหนดและข้อจำกัด](#-ข้อกำหนดและข้อจำกัด)

---

## 🎯 ภาพรวม

โปรเจกต์นี้เป็น Lab Environment ที่ออกแบบมาเพื่อการศึกษาและฝึกอบรมด้าน Security Operations (SecOps) โดยใช้ n8n เป็น Orchestration Platform ในการควบคุม Security Tools ต่างๆ เช่น Nuclei และ Nmap เพื่อทำการ Automated Penetration Testing บนเป้าหมายจำลองที่ปลอดภัย

### สถาปัตยกรรมระบบ

```
┌─────────────────────────────────────────────────────────┐
│                    n8n-secops                           │
│         (Orchestrator + Security Tools)                 │
│              (Kali, Nuclei, Nmap)                       │
└────────┬────────────────────────────────────────────────┘
         │
         ├───┐                    ┌──────────────────┐
         │   │                    │  Wazuh Stack      │
         │   │                    │  (SIEM/EDR)       │
         │   │                    │                   │
         │   │◄───────────────────│ • Indexer         │
         │   │                    │ • Manager         │
         │   │                    │ • Dashboard       │
         │   │                    └──────────────────┘
         ▼   ▼
    ┌────────────────────────────────────────────────────┐
    │              Victim Targets                       │
    │  ┌─────────────┐  ┌─────────────┐  ┌───────────┐ │
    │  │ victim-app  │  │ juice-shop  │  │metaspl2   │ │
    │  │  (Nginx)    │  │  (Node.js)  │  │           │ │
    │  └─────────────┘  └─────────────┘  └───────────┘ │
    └────────────────────────────────────────────────────┘
         │
    ┌────▼────┐
    │ postgres│  ← Database for n8n
    └─────────┘
```

---

## ✨ คุณสมบัติหลัก

- **🔧 Security Tools Integration**: รองรับ Nuclei v3.x และ Nmap พร้อมใช้งานทันที
- **🔍 Wazuh SIEM Integration**: รวม Wazuh Stack (Indexer, Manager, Dashboard) สำหรับการตรวจสอบความปลอดภัยแบบ Real-time และการวิเคราะห์ Logs
- **🤖 AI-Powered Automation**: ใช้ OpenAI เพื่อแปลง Natural Language เป็น Security Commands
- **📱 Line Messaging Integration**: รับคำสั่งและส่งรายงานผ่าน Line Bot
- **🐳 Docker-Based Environment**: รันได้บนทุก Platform (Windows/Linux/macOS, ARM64/AMD64)
- **🔒 Isolated Network**: ใช้ Docker Network เพื่อแยกระบบทดสอบออกจากระบบจริง
- **📊 Automated Reporting**: สร้างรายงานสรุปผลการสแกนอัตโนมัติด้วย AI

---

## 📋 ความต้องการของระบบ

### ระบบปฏิบัติการ

- **Windows 10/11** (ต้องใช้ WSL2 - Ubuntu)
- **Linux** (Ubuntu 20.04+ หรือเทียบเท่า)
- **macOS** (10.15+)

### ซอฟต์แวร์ที่จำเป็น

| ซอฟต์แวร์ | เวอร์ชันขั้นต่ำ | หมายเหตุ |
|-----------|----------------|----------|
| Docker | 20.10+ | ต้องเปิดใช้งาน WSL Integration (Windows) |
| Docker Compose | 2.0+ | รวมอยู่ใน Docker Desktop |
| Git | 2.30+ | สำหรับ Clone Repository |

### สถาปัตยกรรมที่รองรับ

- **Intel/AMD (x86_64)**
- **ARM64** (Apple Silicon, Snapdragon)

> **หมายเหตุ**: การตั้งค่าในโปรเจกต์นี้ปรับจูนสำหรับ ARM64 โดยเฉพาะ

---

## 📂 โครงสร้างโปรเจกต์

```
n8n-secops-lab/
├── .env                      # Environment Variables (ไม่ถูก commit)
├── .env.example              # ตัวอย่าง Environment Variables
├── docker-compose.yml        # Docker Compose Configuration (รวม Wazuh Stack)
├── Dockerfile.n8n           # Custom n8n Image with Security Tools
├── Dockerfile.kali          # Kali Linux Image with Security Tools
├── Dockerfile.wazuh-manager # Custom Wazuh Manager Image
├── wazuh_manager_entrypoint.sh  # Wazuh Manager Entrypoint Script
├── wazuh_n8n_integration.py     # Wazuh-n8n Integration Script
├── manual_alert.json        # Manual Alert for Testing
├── security_report.md       # Sample Security Report
├── README.md                # เอกสารนี้
├── vulnerable_data/          # Vulnerable Target Data
│   └── .env                 # ไฟล์จำลองที่เปิดเผย (สำหรับ Demo)
├── Workflow_1/              # Advanced SecOps AI Pipeline
│   ├── Advanced SecOps AI Pipeline.json
│   ├── Kali Executor Tool.json
│   └── README.md
└── Workflow_2/              # WebSecScan Pro - Multi-Agent AI
    ├── WebSecScan Pro - Workflow Tools Edition.json
    ├── WebSec_SubAgent_Worker.json
    └── README.md
```

### คำอธิบายไฟล์สำคัญ

- **`docker-compose.yml`**: กำหนด Services ทั้งหมด (n8n, victim-app, postgres, Kali Linux, **Wazuh Stack**)
- **`Dockerfile.n8n`**: สร้าง Custom n8n Image ที่ติดตั้ง Nuclei, Nmap, และ Tools อื่นๆ
- **`Dockerfile.kali`**: สร้าง Kali Linux Container พร้อม Security Tools
- **`Dockerfile.wazuh-manager`**: สร้าง Wazuh Manager Custom Image
- **`wazuh_n8n_integration.py`**: Script สำหรับ Integration ระหว่าง Wazuh และ n8n
- **`vulnerable_data/.env`**: ไฟล์จำลองที่เปิดเผยเพื่อใช้ในการทดสอบ

---

## 🚀 การติดตั้งและตั้งค่า

### ขั้นตอนที่ 1: Clone Repository

```bash
git clone <repository-url>
cd n8n-secops-lab
```

### ขั้นตอนที่ 2: สร้าง Environment Variables

คัดลอกไฟล์ `.env.example` เป็น `.env`:

```bash
cp .env.example .env
```

แก้ไขไฟล์ `.env` และตั้งค่าตัวแปรต่อไปนี้:

| ตัวแปร | คำอธิบาย | ตัวอย่าง |
|--------|----------|----------|
| `N8N_USER` | Username สำหรับเข้าสู่ระบบ n8n | `admin` |
| `N8N_PASS` | Password สำหรับเข้าสู่ระบบ n8n | `your_secure_password` |
| `N8N_PORT` | Port สำหรับ n8n Web UI | `5678` |
| `DB_PASS` | Password สำหรับ PostgreSQL | `your_db_password` |
| `CF_TUNNEL_TOKEN` | Cloudflare Tunnel Token (Optional) | _(เว้นว่างได้)_ |

> **⚠️ คำเตือน**: ไฟล์ `.env` จะไม่ถูก commit ขึ้น Repository เพื่อความปลอดภัย

### ขั้นตอนที่ 3: เตรียม Vulnerable Data

สร้างโฟลเดอร์และไฟล์สำหรับเป้าหมายจำลอง:

```bash
mkdir -p vulnerable_data
cat > vulnerable_data/.env << EOF
SECRET_KEY=this_is_a_fake_secret_key_for_demo
DB_PASSWORD=super_secret_password_123
API_KEY=demo_api_key_12345
EOF
```

### ขั้นตอนที่ 4: Build และ Start Services

รันคำสั่งต่อไปนี้เพื่อ Build Docker Images และ Start Services:

```bash
docker-compose up -d --build
```

คำสั่งนี้จะ:
1. Build Custom n8n Image (ติดตั้ง Security Tools)
2. สร้าง Docker Network (`secops_net`)
3. Start Services ทั้งหมด (n8n, victim-app, postgres)

### ขั้นตอนที่ 5: ตรวจสอบสถานะ

ตรวจสอบว่า Services ทั้งหมดรันอยู่:

```bash
docker-compose ps
```

ควรเห็น Services ทั้งหมดมีสถานะ `Up`

---

## 💻 การใช้งาน

### เข้าถึง n8n Web UI

1. เปิด Browser ไปที่: **http://localhost:5678**
2. Login ด้วย:
   - **Username**: ตามที่ตั้งใน `N8N_USER` (default: `admin`)
   - **Password**: ตามที่ตั้งใน `N8N_PASS`

### การจัดการ Workflow

#### Import Workflow

1. คลิก **Workflows** → **Import from File**
2. เลือกไฟล์ `secops-workflow.json`
3. Workflow จะถูก Import เข้ามาในระบบ

#### Activate Workflow

1. เปิด Workflow ที่ต้องการ
2. คลิก **Active** toggle ที่มุมบนขวา
3. Workflow จะเริ่มทำงานและรอรับ Trigger

### คำสั่งที่ใช้บ่อย

#### อัปเดต Nuclei Templates

```bash
docker exec -it n8n-secops nuclei -update-templates
```

#### ตรวจสอบ Version ของ Tools

```bash
# ตรวจสอบ Nuclei Version
docker exec -it n8n-secops nuclei -version

# ตรวจสอบ Nmap Version
docker exec -it n8n-secops nmap --version
```

#### ทดสอบการเชื่อมต่อ

```bash
# ทดสอบการเชื่อมต่อไปยัง victim-app
docker exec -it n8n-secops curl http://victim-app/.env
```

### ตัวอย่างคำสั่ง Security Scan

#### Nmap Scan

```bash
# Fast Scan
nmap -F -T4 victim-app

# Full Scan with Service Detection
nmap -sV -sC victim-app

# Scan Specific Ports
nmap -p 80,443,8080 victim-app
```

#### Nuclei Scan

```bash
# Basic Scan
nuclei -u http://victim-app -silent

# Scan with Specific Tags
nuclei -u http://victim-app -tags exposure -silent

# JSON Output
nuclei -u http://victim-app -json -silent
```

> **💡 เคล็ดลับ**: ใน n8n ให้ใช้ `http://victim-app` แทน `localhost` เพราะ Container อยู่ใน Network เดียวกัน

---

## � Wazuh SIEM

### เข้าถึง Wazuh Dashboard

1. เปิด Browser ไปที่: **https://localhost** (หรือ **https://localhost:443**)
2. Login ด้วย:
   - **Username**: `admin`
   - **Password**: `SecretPassword123!`

### การตรวจสอบ Agents

ตรวจสอบสถานะ Wazuh Agents:

```bash
# ดูรายการ Agents ทั้งหมด
docker exec -it wazuh-manager /var/ossec/bin/agent_control -l

# ตรวจสอบสถานะ Agent ที่เฉพาะเจาะจง (ID 001)
docker exec -it wazuh-manager /var/ossec/bin/agent_control -i 001
```

### การดู Alerts

Alerts จะถูกส่งไปยัง Wazuh Dashboard โดยอัตโนมัติ คุณสามารถดูได้ที่:
- **Security Events** → **Alerts**
- หรือผ่าน API: `https://localhost/app/wazuh`

### การทดสอบส่ง Alert (เพื่อทดสอบ Integration)

```bash
# ส่ง Alert ทดสอบผ่าน Wazuh API
curl -k -u admin:SecretPassword123! -X POST \
  https://localhost:55000/security/events \
  -H "Content-Type: application/json" \
  -d @manual_alert.json
```

### Integration กับ n8n

ระบบมี `wazuh_n8n_integration.py` สำหรับเชื่อมต่อ Wazuh Alerts เข้ากับ n8n Workflows:

```bash
# รัน Integration Script
docker exec -it n8n-secops python /home/node/wazuh_n8n_integration.py
```

สามารถดูรายงานตัวอย่างได้ที่ `security_report.md`

---

## 🔄 SecOps Demo Workflow

โปรเจกต์นี้มี 2 Workflows หลักสำหรับการทดสอบความปลอดภัยแบบอัตโนมัติ:

### 1. Advanced SecOps AI Pipeline (`Workflow_1/`)

ใช้ Kali Linux + AI Agents สำหรับการสแกนความปลอดภัย:
- **Natural Language Commands** - รับคำสั่งภาษาไทย/อังกฤษ
- **Kali Tools Integration** - Nmap, Nuclei, Metasploit
- **Discord Reporting** - ส่งรายงานผ่าน Discord Webhook

**ตัวอย่างคำสั่ง**:
```
สแกนเครื่อง juice-shop-victim ที่พอร์ต 3000 ตรวจสอบ tech stack
```

### 2. WebSecScan Pro (`Workflow_2/`)

ใช้ Multi-Agent AI ตามมาตรฐาน OWASP ASVS:
- **Form Trigger** - กรอก URL ผ่านหน้าเว็บฟอร์ม
- **Multi-Agent System** - Orchestrator + Sub-Agents (Recon, Transport, Identity, Correlation)
- **ASVS Compliance** - ตรวจสอบตามมาตรฐาน OWASP
- **Markdown Reports** - ส่งรายงานเป็นไฟล์ `.md` ไปยัง Discord

**วิธีใช้งาน**: เปิด Workflow → Execute → กรอก Target URL ในฟอร์ม → รอรับรายงาน

### เอกสารเพิ่มเติม

ดูรายละเอียดการตั้งค่าและใช้งานใน **[Workflow_1/README.md](./Workflow_1/README.md)** และ **[Workflow_2/README.md](./Workflow_2/README.md)**

---

## ✅ การตรวจสอบระบบ

### Pre-flight Checklist

ก่อนเริ่มใช้งาน ให้ตรวจสอบความพร้อมของระบบ:

#### 1. ตรวจสอบ Nuclei Version

```bash
docker exec -it n8n-secops nuclei -version
```

**ผลลัพธ์ที่คาดหวัง**: `v3.x.x`

#### 2. ตรวจสอบ Victim App Accessibility

```bash
docker exec -it n8n-secops curl http://victim-app/.env
```

**ผลลัพธ์ที่คาดหวัง**: ควรเห็นเนื้อหาไฟล์ `.env`

#### 3. ตรวจสอบ Network Connectivity

```bash
docker network inspect secops_secops_net
```

**ผลลัพธ์ที่คาดหวัง**: ควรเห็น Containers ทั้งหมดเชื่อมต่ออยู่ใน Network เดียวกัน

#### 4. ตรวจสอบ n8n UI

เปิด Browser ไปที่ **http://localhost:5678** และ Login

**ผลลัพธ์ที่คาดหวัง**: ควรเข้าสู่ระบบได้และเห็น Dashboard

#### 5. ตรวจสอบ Wazuh Dashboard

เปิด Browser ไปที่ **https://localhost** และ Login

**ผลลัพธ์ที่คาดหวัง**: ควรเข้าสู่ Wazuh Dashboard ได้และเห็น Security Events

#### 6. ตรวจสอบ Wazuh Agents

```bash
# ดูรายการ Agents
docker exec -it wazuh-manager /var/ossec/bin/agent_control -l
```

**ผลลัพธ์ที่คาดหวัง**: ควรเห็น victim-app Agent มีสถานะ **Active**

---

## 🔧 การแก้ไขปัญหา

### ปัญหาที่พบบ่อย

#### Error: exec format error

**สาเหตุ**: Nuclei Binary ไม่ตรงกับ Architecture (AMD64 vs ARM64)

**วิธีแก้ไข**:
```bash
docker-compose down --rmi local
docker-compose up -d --build
```

#### Container ไม่สามารถเชื่อมต่อกันได้

**สาเหตุ**: Docker Network ยังไม่ถูกสร้าง

**วิธีแก้ไข**:
```bash
# ตรวจสอบ Network
docker network ls | grep secops_net

# สร้าง Network ใหม่ (ถ้ายังไม่มี)
docker network create secops_net

# Restart Services
docker-compose restart
```

#### Wazuh Dashboard ไม่เปิด

**วิธีแก้ไข**:

1. **ตรวจสอบ Wazuh Services**:
   ```bash
   docker-compose ps | grep wazuh
   ```

2. **ตรวจสอบ Wazuh Manager Logs**:
   ```bash
   docker logs wazuh-manager
   ```

3. **ตรวจสอบ Wazuh Indexer Logs**:
   ```bash
   docker logs wazuh-indexer
   ```

4. **Restart Wazuh Stack**:
   ```bash
   docker-compose restart wazuh.manager wazuh.indexer wazuh.dashboard
   ```

#### Wazuh Agent ไม่เชื่อมต่อ

**วิธีแก้ไข**:

1. **ตรวจสอบ Agent Status**:
   ```bash
   docker exec -it wazuh-manager /var/ossec/bin/agent_control -l
   ```

2. **รีสตาร์ท victim-app Container**:
   ```bash
   docker-compose restart victim-app
   ```

3. **ตรวจสอบ Agent Logs**:
   ```bash
   docker exec -it victim-app tail -f /var/ossec/logs/ossec.log
   ```

#### n8n UI ไม่เปิด

**วิธีแก้ไข**:

1. **ตรวจสอบ Port ถูกใช้งานหรือไม่**:
   ```bash
   docker ps | grep n8n-secops
   netstat -tuln | grep 5678
   ```

2. **ตรวจสอบ Logs**:
   ```bash
   docker logs n8n-secops
   ```

3. **Restart Container**:
   ```bash
   docker-compose restart n8n
   ```

#### Nuclei ไม่พบช่องโหว่

**วิธีแก้ไข**:

1. **อัปเดต Templates**:
   ```bash
   docker exec -it n8n-secops nuclei -update-templates
   ```

2. **ตรวจสอบว่า victim-app มีไฟล์ที่เปิดเผย**:
   ```bash
   docker exec -it n8n-secops curl http://victim-app/.env
   ```

3. **ทดสอบ Nuclei โดยตรง**:
   ```bash
   docker exec -it n8n-secops nuclei -u http://victim-app -tags exposure -v
   ```

### การตรวจสอบ Logs

#### ดู Logs ของ Service ทั้งหมด

```bash
docker-compose logs -f
```

#### ดู Logs ของ Service เฉพาะ

```bash
# n8n Logs
docker-compose logs -f n8n

# victim-app Logs
docker-compose logs -f victim-app

# postgres Logs
docker-compose logs -f postgres

# Wazuh Manager Logs
docker-compose logs -f wazuh.manager

# Wazuh Dashboard Logs
docker-compose logs -f wazuh.dashboard
```

---

## 📚 เอกสารอ้างอิง

### เอกสารทางการ

- [n8n Documentation](https://docs.n8n.io/) - เอกสาร n8n อย่างเป็นทางการ
- [Nuclei Documentation](https://docs.nuclei.sh/) - เอกสาร Nuclei Scanner
- [Nmap Documentation](https://nmap.org/book/) - เอกสาร Nmap Network Scanner
- [Wazuh Documentation](https://documentation.wazuh.com/) - เอกสาร Wazuh SIEM
- [Docker Documentation](https://docs.docker.com/) - เอกสาร Docker

### API Documentation

- [Line Messaging API](https://developers.line.biz/en/docs/messaging-api/) - Line Bot API
- [OpenAI API](https://platform.openai.com/docs/) - OpenAI API Reference

### เอกสารภายในโปรเจกต์

- **[Workflow_1/README.md](./Workflow_1/README.md)** - เอกสาร Advanced SecOps AI Pipeline (Kali + AI)
- **[Workflow_2/README.md](./Workflow_2/README.md)** - เอกสาร WebSecScan Pro (Multi-Agent AI)

---

## ⚠️ ข้อกำหนดและข้อจำกัด

### ⚖️ ข้อกำหนดการใช้งาน

1. **วัตถุประสงค์**: โปรเจกต์นี้ใช้สำหรับ **การศึกษาและฝึกอบรมเท่านั้น**
2. **การใช้งาน**: ใช้ได้เฉพาะในสภาพแวดล้อมที่ควบคุมได้และได้รับอนุญาต
3. **ความรับผิดชอบ**: ผู้ใช้ต้องรับผิดชอบต่อการกระทำของตนเอง

### 🚫 สิ่งที่ห้ามทำ

- ❌ ใช้กับระบบจริงโดยไม่ได้รับอนุญาต
- ❌ ใช้เพื่อโจมตีระบบที่ไม่มีสิทธิ์เข้าถึง
- ❌ แชร์ Credentials หรือ Sensitive Data
- ❌ ใช้ใน Production Environment โดยไม่มีการปรับแต่ง Security

### 🔒 ข้อควรระวังด้านความปลอดภัย

1. **Environment Variables**: อย่า Commit ไฟล์ `.env` ขึ้น Repository
2. **Credentials**: ใช้ Password ที่แข็งแรงและไม่ซ้ำกับระบบอื่น
3. **Network Isolation**: ตรวจสอบว่า Docker Network ไม่เชื่อมต่อกับระบบจริง
4. **Access Control**: จำกัดการเข้าถึง n8n UI ด้วย Authentication

### 📝 License

โปรเจกต์นี้ใช้สำหรับการศึกษาเท่านั้น กรุณาอ่านและปฏิบัติตาม License Agreement ที่เกี่ยวข้อง

---

## 🤝 การมีส่วนร่วม

หากพบปัญหา或有ข้อเสนอแนะ กรุณา:

1. สร้าง Issue ใน Repository
2. อธิบายปัญหาหรือข้อเสนอแนะอย่างชัดเจน
3. แนบ Logs หรือ Screenshots (ถ้ามี)

---

## 📞 ติดต่อ

สำหรับคำถามหรือความช่วยเหลือ กรุณาติดต่อผ่าน:

- **Issues**: สร้าง Issue ใน Repository
- **Documentation**: อ่านเอกสารใน `Workflow_1/README.md` และ `Workflow_2/README.md`

---

<div align="center">

**⚠️ ใช้อย่างมีความรับผิดชอบ ⚠️**

*โปรเจกต์นี้ใช้สำหรับการศึกษาเท่านั้น*

</div>
