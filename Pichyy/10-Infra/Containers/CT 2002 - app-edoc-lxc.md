---
tags:
  - proxmox
  - lxc
  - edoc
  - สารบรรณ
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
ctid: 2002
---

# CT 2002 — app-edoc-lxc (ระบบสารบรรณอิเล็กทรอนิกส์)

> [!info] ภาพรวม
> ระบบสารบรรณอิเล็กทรอนิกส์ของคณะ รันบน Apache + PHP 7.4 มีไฟล์แนบราชการ ~50 GB ย้ายมาจาก VM 3002

## สเปก

| รายการ | รายละเอียด |
| :--- | :--- |
| CTID | `2002` |
| Hostname | `app-edoc-lxc` |
| OS / Runtime | Debian 12 + Apache 2.4.41 + **PHP 7.4** (Ondřej Surý) |
| สเปก | 4 vCPU / 4 GB RAM / 80 GB SAS HDD (`storage-raid`) |
| IP | `192.168.92.25/24` (GW `.1`) |
| DocumentRoot | `/var/www/html/` |
| ไฟล์แนบ | `/var/www/html/document_file/` (~50 GB) |
| Database | `FTE_Edoc` ที่ [[CT 2000 - core-db-mariadb]] |
| URL เดิม | `http://edocument.fte.kmutnb.ac.th/` |
| URL ใหม่ | `https://202.44.35.40/edoc/` |
| สถานะ | Online (200 OK) |

## รายละเอียดระบบ

- **แอป:** ระบบสารบรรณอิเล็กทรอนิกส์ (ไม่ใช่ WordPress — เป็น PHP app แยก)
- **Config DB:** `/var/www/html/sql/connection.php` → `DB_HOST = 192.168.92.210`
- ** Dashboard:** กักเส้นทาง `../../dashboard/` ให้อยู่ใต้ `/edoc/dashboard/` 100% ผ่าน sub_filter
- ระบบมีโฟลเดอร์: `document_file`, `member`, `image`, `icon`, `function`, `sql`, `css`, `js`
- **ไฟล์แนบขนาดใหญ่** — `client_max_body_size 100M` ตั้งที่ proxy ทุกชั้น

## URL Routing (ผ่าน [[CT 1001 - core-npm-proxy]])

```
/edoc/ → proxy_pass http://192.168.92.25:80/
sub_filter จัดการเฉพาะ:
  - href="/  src="/  action="/  → /edoc/
  - href="../  src="../ → /edoc/
  - ../../dashboard/ → /edoc/dashboard/
  - window.location.href = 'dashboard' → /edoc/dashboard/
proxy_cookie_path / /edoc/  (แก้ path ของ cookies ให้อยู่ใต้ /edoc/)
```

## คำสั่งจัดการ

```bash
pct exec 2002 -- systemctl status apache2
pct exec 2002 -- du -sh /var/www/html/document_file/   # เช็คพื้นที่ไฟล์แนบ
pct restart 2002
```

## ประวัติการย้าย

- ย้ายด้วย **Workflow 1: Direct Raw Block Stream** (ปิดเครื่อง → รวม snapshot → `dd` ตรง)
- ข้อมูลจริง ~50.3 GB (เว็บ 50 GB + DB 301 MB) แต่เดิมจอง 200 GB → คืนพื้นที่ ~149.7 GB
- เครื่องเดิม VM 32 บน ESXi ถูกปิดสำรองไว้ที่ `datastore43.3`

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · วิธีย้าย: [[VM Migration Playbook]] · Routing: [[Gateway Nginx Setup]]*
