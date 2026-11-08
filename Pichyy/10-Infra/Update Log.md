---
tags:
  - proxmox
  - update-log
  - changelog
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
---

# อัปเดตระบบ 15 ต.ค. 2569 (เพิ่ม CT 2005–2006)

> [!summary] สรุปการเปลี่ยนแปลง
> รอบอัปเดตนี้ **เพิ่ม 3 เว็บไซต์ใหม่** (เว็บคณะ TH, เว็บคณะ EN, วารสาร JTechEE) เป็น **2 Container ใหม่คือ CT 2005 และ CT 2006** พร้อมย้ายฐานข้อมูลใหม่ 3 ตัวเข้า [[CT 2000 - core-db-mariadb]] และปรับ Routing ของ NC/IC — ทั้งหมดแบบ **Zero Downtime** (เครื่องเดิมบน ESXi ยังเปิดอยู่)

> [!info] อ้างอิงจาก
> ไฟล์ใหม่ `LATEST_UPDATE_NOTES.md` + commit `ad6a80c`, `9e1034c` (README.md อัปเดตแล้ว แต่ `HANDOFF.md` และ `SYSTEM_HANDOVER_CREDENTIALS.txt` ยังไม่ครอบคลุม CT 2005–2006)

## 1. เพิ่ม Container ใหม่ 2 ตัว

| CTID | ชื่อ | IP ใหม่ | สเปก | Runtime | Storage |
| :--: | :--- | :--- | :--- | :--- | :--- |
| **2005** | `app-fte-web-lxc` | `192.168.92.134` | 4 vCPU / 4 GB / 40 GB | PHP 7.2 | `storage-raid` (SAS) |
| **2006** | `app-jtechee-lxc` | `192.168.92.119` | 2 vCPU / 2 GB / 20 GB | PHP 7.4 | `storage-raid` (SAS) |

- ทั้งคู่ใช้ template `debian-12-standard` + `--features nesting=1` (จำเป็นสำหรับ WordPress) + `onboot=1`
- Nameserver: `202.44.32.29 202.44.32.30`
- สร้างด้วยสคริปต์ `provision_new_cts.py`, ย้ายข้อมูลด้วย `migrate_new_systems_full.py`

## 2. เว็บไซต์ใหม่ 3 ตัว และ URL

| เว็บ | Container | URL ผ่าน Gateway | Host Header | พาธใน Container |
| :--- | :--- | :--- | :--- | :--- |
| **FTE Web TH** (เว็บคณะ) | CT 2005 | `https://202.44.35.40/fte-web/th/` | `fte.kmutnb.ac.th` | `/var/www/html/th/` |
| **FTE Web EN** (เว็บคณะ) | CT 2005 | `https://202.44.35.40/fte-web/en/` | `fte-en.kmutnb.ac.th` | `/var/www/html/en/` |
| **JTechEE Journal** (วารสาร) | CT 2006 | `https://202.44.35.40/jtechee/` | `jtechee.fte.kmutnb.ac.th` | `/var/www/html/` |

> [!note] จุดพิเศษของ CT 2005
> รวม **2 เว็บ (ไทย + อังกฤษ) ไว้ในตู้เดียว** แยกด้วย sub-path `/th/` และ `/en/`
> Root `/fte-web` และ `/fte-web/` จะ **301 redirect ไป `/fte-web/th/` อัตโนมัติ** (ผ่าน `index.php` ที่ root เช่นกัน)

## 3. เพิ่มฐานข้อมูลใหม่ 3 ตัว (รวมเป็น 10 ฐาน)

| Database | Tables | ระบบงาน | DB User | Password |
| :--- | :--: | :--- | :--- | :--- |
| `km_fte` | 17 | เว็บคณะ ภาษาไทย (CT 2005) | `phai` | `Ph@ii8899#` |
| `web_en` | 45 | เว็บคณะ ภาษาอังกฤษ (CT 2005) | `admin_en` | `12345Fte` |
| `web_jtechee` | 12 | วารสาร JTechEE (CT 2006) | `jtech` | `1234Jtech` |

- ย้ายเข้า [[CT 2000 - core-db-mariadb]] ทั้งหมด, charset `utf8mb4`
- แก้ `wp-config.php` ให้ `DB_HOST = 192.168.92.210`
- ⚠️ `web_en` เดิมเคยใช้ DB ชื่อ `web_en` ทั้งสองที่ — CT 2006 ต้องแก้ `DB_NAME` เป็น `web_jtechee` (สคริปต์ `migrate_new_systems_full.py` จัดการให้)

## 4. เปลี่ยน Routing ของ NC / IC (สำคัญ)

จากเดิมที่ CT 2003 ให้ `/nc/` และ `/ic/` ไปที่ DocumentRoot ตรง ๆ ตอนนี้เปลี่ยนให้ชี้เข้า sub-folder จริง:

| Path | เดิม | ใหม่ |
| :--- | :--- | :--- |
| `/nc/` | `http://192.168.92.56:80/` | `http://192.168.92.56:80/**web_nc**/` |
| `/ic/` | `http://192.168.92.56:80/` | `http://192.168.92.56:80/**web_ic**/` |

- เพิ่ม `proxy_cookie_path /web_nc/ /nc/` และ `/web_ic/ /ic/`
- เพิ่ม sub_filter แปลง `/web_nc/` → `/nc/` และ `/web_ic/` → `/ic/`
- ดูรายละเอียดที่ [[CT 2003 - app-nc-ic-lxc]] และ [[Gateway Nginx Setup]]

## 5. Gateway เพิ่ม Path ใหม่

[[Gateway Nginx Setup]] ทั้ง 2 ชั้น (Edge + NPM) เพิ่ม:
- `/fte-web/th/` → CT 2005 (Host `fte.kmutnb.ac.th`)
- `/fte-web/en/` → CT 2005 (Host `fte-en.kmutnb.ac.th`)
- `/jtechee/` → CT 2006 (Host `jtechee.fte.kmutnb.ac.th`)
- `server_name` เพิ่ม `fte.kmutnb.ac.th`, `fte-en.kmutnb.ac.th`, `jtechee.fte.kmutnb.ac.th`
- Deploy ทั้งหมดด้วยสคริปต์ `deploy_new_routes_and_backups.py`

## 6. Backup ครอบคลุมมากขึ้น

| รายการ | เดิม | ใหม่ |
| :--- | :--- | :--- |
| mysqldump (00:00) | 5 ฐาน (`fte_openhouse FTE_Edoc aiet ncteched icteched`) | **10 ฐาน** ตามรายการด้านบน |
| VZDump (01:00) | CT 2000 + CT 2001–2004 | CT **2000, 2001, 2002, 2003, 2004, 2005, 2006** (7 ตัว) |
| คำสั่ง dump | `mysqldump` | `mariadb-dump -u root -p...` |

ดูรายละเอียดที่ [[Backup Plan FTE]]

## 7. FTE App Portal (CT 2007)

- เพิ่ม Container **CT 2007 `app-portal-lxc`**: 2 vCPU/1 GB/10 GB, Nginx static SPA, IP `192.168.92.213`, URL `https://202.44.35.40/portal/`
- App: React 19 + TypeScript + Vite + **GitHub Primer Design System** (ไม่มี Tailwind/Bootstrap)
- รวมลิงก์ทุกระบบ: fte-th/en, edoc, openhouse, nc/ic, jtechee, aiet + admin (NPM :81, PVE :8006)
- การ deploy: `provision_portal_ct.py` → `deploy_portal.py` (bundle dist → push → setup nginx/proxy)
- VZDump รายวันเพิ่มเป็น **8 CT** (รวม 2007), Edge/NPM เพิ่ม `/portal/` (ดู [[CT 2007 - app-portal-lxc]] · [[FTE App Portal]] · [[Gateway Nginx Setup]] · [[Backup Plan FTE]])

## 8. ESXi Cluster Audit (3 โฮสต์ / 46 VM)

- สร้าง **[[ESXi Cluster Audit - 3 Hosts 46 VMs]]** + อัปเดต `ESXI_INFRASTRUCTURE_AUDIT.md` / `README.md` / `SYSTEM_HANDOVER_CREDENTIALS.txt`
- สรุป: **3 โหนด, 46 VM** (เปิด 21 / ปิด 25). โหนด 1 (ESXi 6.5.0) 17 VM, โหนด 2 (ESXi 6.0.0) 18 VM, โหนด 3 (ESXi 5.5.0) 11 VM
- ระบบหลักที่ยังอยู่บน ESXi: AD/DC, File Server, Karupan, Vijai, IPR, Conference, Legacy (35.11, DB/Web Edoc เก่า, jtechee เก่า, CCTV/UniFi) เป็นต้น

## 9. เอกสาร/สคริปต์เพิ่มเติม

- **ESXi Host 2 Spec & Stack:** เพิ่มโน้ต `ESXi Host 2 - Spec & Stack` ครอบคลุม 6 VM (AD1/AD2, FS7 4.17TB, vNIOS, Net_Monitor, Tech) พร้อมเครือข่าย 4 VLAN, RAM/storage gap, แผนย้าย 4 เฟส


- เอกสาร: `ESXI_INFRASTRUCTURE_AUDIT.md` (อัพเดต), `README.md` (เพิ่มลิงก์ audit), `SYSTEM_HANDOVER_CREDENTIALS.txt` (เพิ่ม ESXi creds ภาค audit — **ห้ามคัดลอกเข้า vault**)



---
*ดูภาพรวมทั้งหมด: [[Proxmox VE Migration FTE]] · [[ESXi Cluster Audit - 3 Hosts 46 VMs]] · [[ESXi Host 2 - Spec & Stack]]*
