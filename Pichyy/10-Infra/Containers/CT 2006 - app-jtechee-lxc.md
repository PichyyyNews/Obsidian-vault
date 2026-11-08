---
tags:
  - proxmox
  - lxc
  - wordpress
  - jtechee
  - journal
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
ctid: 2006
---

# CT 2006 — app-jtechee-lxc (วารสาร JTechEE)

> [!info] ภาพรวม
> เว็บวารสาร JTechEE (Journal of Technical Education and Engineering) รัน WordPress บน Apache + PHP 7.4 ย้ายจาก VM 47 (Journal FTE New) แบบ Zero-Downtime

## สเปก

| รายการ | รายละเอียด |
| :--- | :--- |
| CTID | `2006` |
| Hostname | `app-jtechee-lxc` |
| OS / Runtime | Debian 12 + Apache 2.4 + **PHP 7.4** (Ondřej Surý) |
| สเปก | 2 vCPU / 2 GB RAM / 20 GB SAS HDD (`storage-raid`) |
| IP (Staging) | `192.168.92.119/24` (Bridge `vmbr0`, VLAN 266, GW `.1`) |
| DNS | `202.44.32.29`, `202.44.32.30` |
| Features | `nesting=1` |
| Auto-Start | `onboot=1` |
| DocumentRoot | `/var/www/html/` |
| URL ใหม่ | `https://202.44.35.40/jtechee/` (Host `jtechee.fte.kmutnb.ac.th`) |
| สถานะ | Online (200 OK) |

## ฐานข้อมูล (บน [[CT 2000 - core-db-mariadb]])

| Database | Tables | User | Password |
| :--- | :--: | :--- | :--- |
| `web_jtechee` | 12 | `jtech` | `1234Jtech` |

> [!warning] จุดที่ต้องระวัง
> เว็บต้นทาง (`.19`) เดิมใช้ DB ชื่อ `web_en` — ตอนย้ายต้องแก้ `DB_NAME` เป็น `web_jtechee` ด้วย (สคริปต์ `migrate_new_systems_full.py` จัดการผ่าน `find ... -exec sed`)
> Config: `wp-config.php` → `DB_HOST = 192.168.92.210`, `DB_NAME = web_jtechee`

## วิธีตั้งค่า (จากสคริปต์)

```bash
pct create 2006 local:vztmpl/debian-12-standard_*.tar.zst \
  --hostname app-jtechee-lxc --cores 2 --memory 2048 --swap 512 \
  --rootfs storage-raid:20 \
  --net0 name=eth0,bridge=vmbr0,tag=266,ip=192.168.92.119/24,gw=192.168.92.1 \
  --nameserver "202.44.32.29 202.44.32.30" --onboot 1 --unprivileged 0

# ย้ายข้อมูล: dump web_jtechee จาก .19 → import CT 2000 → rsync /var/www/html/
```

## คำสั่งจัดการ

```bash
pct exec 2006 -- systemctl status apache2
pct exec 2006 -- find /var/www/html -name wp-config.php
pct restart 2006
```

## ประวัติการย้าย

- **Zero Downtime:** เครื่องเดิม VM 47 (`192.168.92.19`) ยังเปิดทำงานปกติ 100%
- ใช้ Staging IP `.119` — ไม่ชน IP/MAC
- ดูรายละเอียดที่ [[Update Log]] และ [[ESXi Legacy Servers]]

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · Routing: [[Gateway Nginx Setup]] · DB: [[CT 2000 - core-db-mariadb]]*
