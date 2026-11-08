---
tags:
  - proxmox
  - lxc
  - wordpress
  - fte-web
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
ctid: 2005
---

# CT 2005 — app-fte-web-lxc (เว็บคณะ TH + EN)

> [!info] ภาพรวม
> Container เดียวรวม **เว็บคณะ 2 ภาษา** — เว็บไทย (`/th/`) และเว็บอังกฤษ (`/en/`) รัน WordPress บน Apache + PHP 7.2 ย้ายจาก VM 58 (webfte) + VM 48 (test web) แบบ Zero-Downtime

## สเปก

| รายการ | รายละเอียด |
| :--- | :--- |
| CTID | `2005` |
| Hostname | `app-fte-web-lxc` |
| OS / Runtime | Debian 12 + Apache 2.4 + **PHP 7.2** (Ondřej Surý) |
| สเปก | 4 vCPU / 4 GB RAM / 40 GB SAS HDD (`storage-raid`) |
| IP (Staging) | `192.168.92.134/24` (Bridge `vmbr0`, VLAN 266, GW `.1`) |
| DNS | `202.44.32.29`, `202.44.32.30` |
| Features | `nesting=1` (จำเป็นสำหรับ WordPress) |
| Auto-Start | `onboot=1` |
| สถานะ | Online (200 OK) ทั้ง 2 เว็บ |

## 2 เว็บที่ให้บริการ

| เว็บ | URL ใหม่ | Host Header | พาธใน Container |
| :--- | :--- | :--- | :--- |
| **FTE Web TH** | `https://202.44.35.40/fte-web/th/` | `fte.kmutnb.ac.th` | `/var/www/html/th/` |
| **FTE Web EN** | `https://202.44.35.40/fte-web/en/` | `fte-en.kmutnb.ac.th` | `/var/www/html/en/` |

- Root `/fte-web`, `/fte-web/`, `/fte-web/th` → **301 redirect ไป `/fte-web/th/`**
- Root ของ container เองมี `index.php` ที่ `header("Location: /th/", true, 301)` ด้วย

## ฐานข้อมูล (บน [[CT 2000 - core-db-mariadb]])

| Database | Tables | User | ระบบ |
| :--- | :--: | :--- | :--- |
| `km_fte` | 17 | `phai` | เว็บคณะ ไทย (TH) |
| `web_en` | 45 | `admin_en` | เว็บคณะ อังกฤษ (EN) |

- Config: `/var/www/html/th/wp-config.php` และ `/var/www/html/en/wp-config.php` → `DB_HOST = 192.168.92.210`

## วิธีตั้งค่า (จากสคริปต์)

```bash
# สร้าง CT (provision_new_cts.py)
pct create 2005 local:vztmpl/debian-12-standard_*.tar.zst \
  --hostname app-fte-web-lxc --cores 4 --memory 4096 --swap 1024 \
  --rootfs storage-raid:40 \
  --net0 name=eth0,bridge=vmbr0,tag=266,ip=192.168.92.134/24,gw=192.168.92.1 \
  --nameserver "202.44.32.29 202.44.32.30" --onboot 1 --unprivileged 0

# ย้ายข้อมูล (migrate_new_systems_full.py)
# 1) dump km_fte จาก 192.168.92.34 → import เข้า CT 2000
# 2) dump web_en จาก 192.168.92.29 → import เข้า CT 2000
# 3) rsync /var/www/html/ จาก .34 → /var/www/html/th/
#    rsync /var/www/html/ จาก .29 → /var/www/html/en/
# 4) แก้ DB_HOST ใน wp-config.php → 192.168.92.210
```

## คำสั่งจัดการ

```bash
pct exec 2005 -- systemctl status apache2
pct exec 2005 -- ls /var/www/html/
pct restart 2005
```

## ประวัติการย้าย

- **Zero Downtime:** เครื่องเดิม VM 58 (`192.168.92.34`) และ VM 48 (`192.168.92.29`) ยังเปิดทำงานปกติ 100%
- ใช้ Staging IP `.134` แยกจากเครื่องเดิม — ไม่ชน IP/MAC
- ดูรายละเอียดที่ [[Update Log]] และ [[ESXi Legacy Servers]]

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · Routing: [[Gateway Nginx Setup]] · DB: [[CT 2000 - core-db-mariadb]]*
