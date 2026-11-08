---
tags:
  - proxmox
  - lxc
  - conference
  - nc
  - ic
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
ctid: 2003
---

# CT 2003 — app-nc-ic-lxc (งานประชุมวิชาการ NC & IC)

> [!info] ภาพรวม
> Container เดียวครอบ 2 เว็บ — Web NC (ประชุมวิชาการระดับชาติ) และ Web IC (ประชุมวิชาการนานาชาติ) ย้ายมาจาก VM 3003 แบบ Zero-Downtime

## สเปก

| รายการ | รายละเอียด |
| :--- | :--- |
| CTID | `2003` |
| Hostname | `app-nc-ic-lxc` |
| OS / Runtime | Debian 12 + Apache 2.4 + **PHP 7.2** (Ondřej Surý) |
| สเปก | 2 vCPU / 4 GB RAM / 20 GB SAS HDD (`storage-raid`) |
| IP | `192.168.92.56/24` (GW `.1`, DNS `202.44.32.29`) |
| DocumentRoot | `/var/www/html/` |
| Database | `web_nc`, `web_ic`, `conference_db`, `web_student` ที่ [[CT 2000 - core-db-mariadb]] |
| สถานะ | Online (200 OK) ทั้ง 2 เว็บ |

## เว็บที่ให้บริการ 2 ระบบ

| เว็บ | URL เดิม | URL ใหม่ | พาธใน Container |
| :--- | :--- | :--- | :--- |
| **Web NC** | `ncteched.fte.kmutnb.ac.th` | `https://202.44.35.40/nc/` | `/var/www/html/Conference/` |
| **Web IC** | `icteched.fte.kmutnb.ac.th` | `https://202.44.35.40/ic/` | `/var/www/html/web_ic/` |

- โฟลเดอร์อื่นใน DocumentRoot: `/web_nc`, `/WStudent`
- **Config DB:** `wp-config.php` & `connDB.php` → `DB_HOST = 192.168.92.210`

## URL Routing (ผ่าน [[CT 1001 - core-npm-proxy]])

```
/nc/ → proxy_pass http://192.168.92.56:80/web_nc/  (Host: ncteched.fte.kmutnb.ac.th)
/ic/ → proxy_pass http://192.168.92.56:80/web_ic/  (Host: icteched.fte.kmutnb.ac.th)
```

> [!warning] เปลี่ยน Routing รอบ 15 ต.ค. 2569
> เดิมชี้ที่ DocumentRoot ตรง ๆ (`http://192.168.92.56:80/`) ตอนนี้ชี้เข้า sub-folder จริง **`/web_nc/`** และ **`/web_ic/`** แทน
> พร้อมเพิ่ม `proxy_cookie_path /web_nc/ /nc/` และ `/web_ic/ /ic/`

Sub-filter ของทั้งคู่แปลง:
- `https://ncteched...` และ `http://ncteched...` → `/nc/` (หรือ `/ic/`)
- `http://192.168.92.56/web_nc/` → `/nc/` (เดิมเป็น `http://192.168.92.56/`)
- `href="/web_nc/`, `src="/web_nc/`, `action="/web_nc/` → ใส่ prefix `/nc/` (หรือ `/ic/`)
- **ระวัง Protocol-Relative URLs** — ปล่อย `href="//` `src="//` ไว้ ห้ามแปลง (กัน fonts.googleapis.com พัง)

## คำสั่งจัดการ

```bash
pct exec 2003 -- systemctl status apache2
pct exec 2003 -- ls /var/www/html/
pct restart 2003
```

## ประวัติการย้าย

- ย้ายด้วย **Workflow 2: Zero-Downtime Live Migration** (snapshot + kernel web server + chain merge)
- **ต้นทาง VM 53 บน ESXi ยังคงเปิดทำงานปกติ** (IP `192.168.92.55`) ไม่ดับ — ใช้ IP ใหม่ `.56` + MAC ใหม่ `BC:24:11:30:03:56` ป้องกัน collision
- ข้อมูลจริง ~6.9 GB (เว็บ 5.3 GB + DB 1.6 GB) เดิมจอง 150 GB

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · วิธีย้าย: [[VM Migration Playbook]] · Routing: [[Gateway Nginx Setup]] · เครื่องเดิม: [[ESXi Legacy Servers]]*
