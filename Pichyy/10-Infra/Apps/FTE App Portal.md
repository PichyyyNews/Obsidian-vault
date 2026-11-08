---
tags:
  - portal
  - react
  - primer
  - proxmox
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate\fte-app-portal
status: production
---

# FTE App Portal (หน้ารวมลิงก์ทุกระบบ)

> [!info] ภาพรวม
> หน้าเว็บกลาง **FTE Application Portal** รวมลิงก์ระบบงานทั้งหมดของคณะครุศาสตร์อุตสาหกรรม มจพ. เป็น SPA แบบ static รันบน [[CT 2007 - app-portal-lxc]] → **`https://202.44.35.40/portal/`**

## 1. สแต็กเทคโนโลยี

| รายการ | เวอร์ชัน / รายละเอียด |
| :--- | :--- |
| UI Framework | **GitHub Primer Design System** — `@primer/react` 38.41, `@primer/primitives` 11.10, `@primer/octicons-react` 19.40 |
| โครงสร้าง | React **19.2** + TypeScript **6.0** + Vite **8.3** + styled-components 6.6 |
| Linter | **oxlint** (`npm run lint`) — ไม่ใช้ ESLint |
| สคริปต์ | `dev` · `build` (`tsc -b && vite build`) · `preview` |
| ไม่ใช้ | **ไม่มี Tailwind CSS / Bootstrap** — Primer ล้วน 100% |
| ที่อยู่โค้ด | `fte-app-portal/` (มี `.gitignore` กัน `node_modules/`, `dist/`) |

## 2. มาตรฐานการออกแบบ

- **ฟอนต์:** Google Noto ทั้งชุด (`Noto Sans`, `Noto Sans Thai`, `Noto Serif`, `Noto Sans Mono`) + `line-height: 1.5` กันสระ/วรรณยุกต์ไทยล้นกรอบ (**Anti-tofu**)
- **กัน UI จำเจ (Anti-AI-Slop):** ไม่ใช้การ์ดซ้ำ ๆ แบบ grid เดิม ๆ แต่ใช้ **Dense Bordered Container** + หัวข้อบอกจำนวนระบบออนไลน์ + Definition Metadata List
- **ค้นหา/กรอง:** Filter Toolbar — `TextInput` ค้นหาแบบเรียลไทม์ + ปุ่มกรองหมวดหมู่ + แสดง **`Blankslate`** เมื่อไม่พบผล
- **ธีม:** สลับ **Light / Dark** ผ่าน Primer Design Tokens
- **Sticky Infrastructure Pane:** คอลัมน์ขวา (296px) โชว์สเปก Host Proxmox, คลัสเตอร์ MariaDB NVMe, VLAN 266 + ปุ่ม Quick Admin (PVE Console, NPM)

## 3. รายการระบบใน Portal (11 รายการ)

### ระบบเว็บสาธารณะ (8 — สถานะ `operational`)
| ชื่อที่แสดง | URL | Container | หมวด |
| :--- | :--- | :--- | :--- |
| เว็บคณะ (ภาษาไทย) | `/fte-web/th/` | [[CT 2005 - app-fte-web-lxc]] | เว็บหลักและวิชาการ |
| เว็บคณะ (ภาษาอังกฤษ) | `/fte-web/en/` | [[CT 2005 - app-fte-web-lxc]] | เว็บหลักและวิชาการ |
| FTE Edoc | `/edoc/` | [[CT 2002 - app-edoc-lxc]] | งานสารบรรณและบริหาร |
| Open House FTE | `/openhouse/` | [[CT 2001 - app-openhouse-lxc]] | เว็บหลักและวิชาการ |
| NC TechED | `/nc/` | [[CT 2003 - app-nc-ic-lxc]] | งานประชุมและวารสาร |
| IC TechED | `/ic/` | [[CT 2003 - app-nc-ic-lxc]] | งานประชุมและวารสาร |
| JTechEE | `/jtechee/` | [[CT 2006 - app-jtechee-lxc]] | งานประชุมและวารสาร |
| AIET | `/aiet/` | [[CT 2004 - app-aiet-lxc]] | เว็บหลักและวิชาการ |

### โครงสร้างพื้นฐาน (3 — สถานะ `internal`, เฉพาะผู้ดูแล)
| ชื่อที่แสดง | ลิงก์ | ระบบจริง |
| :--- | :--- | :--- |
| ฐานข้อมูลกลาง MariaDB 10.11 Cluster | `#` (ไม่มีลิงก์) | [[CT 2000 - core-db-mariadb]] |
| Nginx Proxy Manager | `http://192.168.92.211:81` | [[CT 1001 - core-npm-proxy]] |
| คอนโซล Proxmox VE | `https://202.44.35.40:8006/` | Host [[Host FTEPVE]] |

> [!note] หมวดหมู่ (category)
> `faculty` เว็บหลักและวิชาการ · `conferences` งานประชุมและวารสาร · `internal` งานสารบรรณและบริหาร · `infra` โครงสร้างพื้นฐาน

## 4. ข้อสังเกตที่ควรแก้ (Source ปัจจุบัน)

> [!warning] ป้าย `tech` ใน `App.tsx` ไม่ตรงของจริง
> | ระบบ | แสดงใน Portal | ของจริง (เอกสาร) |
> | :--- | :--- | :--- |
> | Edoc | `PHP 5.6` | **PHP 7.4** |
> | Open House | `PHP 7.4` | **PHP 7.2** |
> | NC / IC | `PHP 7.4` | **PHP 7.2** |
> | AIET | `PHP 7.4` | **PHP 8.3** |
> | คอนโซล PVE | `Proxmox VE 8.x` | **Proxmox VE 9.1.1** |
>
> ถ้าจะให้ Portal เป็นแหล่งข้อมูลที่เชื่อถือได้ ควรแก้ metadata เหล่านี้ใน `fte-app-portal/src/App.tsx`

## 5. วิธี Deploy

```bash
# 1) สร้าง CT 2007 + Nginx
python provision_portal_ct.py

# 2) build + แพ็ก dist + อัปโหลด + ตั้ง proxy อัตโนมัติ
python deploy_portal.py
```

- `deploy_portal.py` จะ tar `fte-app-portal/dist` → `portal_dist.tar.gz` → `pct push` เข้า CT 2007 → แตกไฟล์ที่ `/var/www/html/portal/` → เพิ่ม block `/portal/` ใน NPM (CT 1001) และ Edge Nginx (Host) → ทดสอบ `curl -skI`
- ดูการตั้งค่า Container และ Nginx ทั้งหมดที่ [[CT 2007 - app-portal-lxc]]

## 6. สถานะ
- ✅ Deploy แล้ว — บันทึกใน `LATEST_UPDATE_NOTES.md` (หมวด 7) และอ้างอิงใน `README.md`
- ✅ ถูกรวมใน VZDump รายวัน (8 CT) — ดู [[Backup Plan FTE]]
- 📌 ภาพรวมการอัปเดตรอบนี้: [[Update Log]]

---
*กลับไปที่: [[Proxmox VE Migration FTE]]*
