---
tags:
  - proxmox
  - lxc
  - wordpress
  - openhouse
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
ctid: 2001
---

# CT 2001 — app-openhouse-lxc (Open House FTE)

> [!info] ภาพรวม
> เว็บ Open House ของคณะ รัน WordPress CMS บน Apache + PHP 7.2 ย้ายมาจาก VM 3001 (เดิมบน ESXi)

## สเปก

| รายการ | รายละเอียด |
| :--- | :--- |
| CTID | `2001` |
| Hostname | `app-openhouse-lxc` |
| OS / Runtime | Debian 12 + Apache 2.4 + **PHP 7.2** (Ondřej Surý) |
| สเปก | 2 vCPU / 2 GB RAM / 15 GB SAS HDD (`storage-raid`) |
| IP | `192.168.92.26/24` (GW `.1`, DNS `202.44.32.29`) |
| DocumentRoot | `/var/www/html/openhouse/` |
| Database | `fte_openhouse` ที่ [[CT 2000 - core-db-mariadb]] |
| URL เดิม | `http://openhouse.fte.kmutnb.ac.th/` |
| URL ใหม่ | `https://202.44.35.40/openhouse/` |
| สถานะ | Online (200 OK) |

## รายละเอียดระบบ

- **CMS:** WordPress (`fte_openhouse`)
- **Config DB:** `/var/www/html/openhouse/wp-config.php` → `DB_HOST = 192.168.92.210`
- ขนาดข้อมูลจริง: เว็บ ~3.7 GB + DB ~312 MB (~4 GB รวม)
- เดิมจองดิสก์ 150 GB แต่ข้อมูลจริงน้อยมาก → เหลือพื้นที่ RAID คืนเยอะ

## URL Routing (ผ่าน [[CT 1001 - core-npm-proxy]])

```
/openhouse/ → proxy_pass http://192.168.92.26:80/
Host header ถูก set เป็น openhouse.fte.kmutnb.ac.th
sub_filter แปลง href="/, src="/, action="/ ให้อยู่ใต้ /openhouse/
```

## คำสั่งจัดการ

```bash
pct exec 2001 -- systemctl status apache2
pct exec 2001 -- php -v
pct restart 2001
```

## ประวัติการย้าย

- ย้ายด้วย **Workflow 1: Direct Raw Block Stream** (ปิดเครื่องต้นทาง → รวม snapshot บน ESXi → `dd` ตรงเข้า block storage)
- เครื่องเดิม VM 45 บน ESXi ถูกปิดสำรองไว้ที่ `datastore43.3`

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · วิธีย้าย: [[VM Migration Playbook]] · Routing: [[Gateway Nginx Setup]]*
