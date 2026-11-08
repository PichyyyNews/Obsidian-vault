---
tags:
  - proxmox
  - lxc
  - aiet
  - wordpress
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
ctid: 2004
---

# CT 2004 — app-aiet-lxc (ศูนย์นวัตกรรม AIET)

> [!info] ภาพรวม
> เว็บศูนย์นวัตกรรมและเทคโนโลยี AIET รัน WordPress บน Apache + PHP 8.3 ย้ายมาจาก VM 3004 แบบ Zero-Downtime

## สเปก

| รายการ | รายละเอียด |
| :--- | :--- |
| CTID | `2004` |
| Hostname | `app-aiet-lxc` |
| OS / Runtime | Debian 12 + Apache 2.4 + **PHP 8.3** (Ondřej Surý) |
| สเปก | 2 vCPU / 2 GB RAM / 15 GB SAS HDD (`storage-raid`) |
| IP | `192.168.92.57/24` (GW `.1`, DNS `202.44.32.29`) |
| DocumentRoot | `/var/www/html/` |
| Database | `aiet` ที่ [[CT 2000 - core-db-mariadb]] |
| URL เดิม | `http://aiet.fte.kmutnb.ac.th/` |
| URL ใหม่ | `https://202.44.35.40/aiet/` |
| สถานะ | Online (200 OK) |

## รายละเอียดระบบ

- **CMS:** WordPress — ศูนย์นวัตกรรมและเทคโนโลยี (สมาคมครุศาสตร์อุตสาหกรรม)
- **Config DB:** `/var/www/html/wp-config.php` → `DB_HOST = 192.168.92.210`
- ข้อมูลจริง ~3.7 GB (เว็บ 2.6 GB + DB 1.1 GB) — เดิมจอง 100 GB

## URL Routing (ผ่าน [[CT 1001 - core-npm-proxy]])

```
/aiet/ → proxy_pass http://192.168.92.57:80/  (Host: aiet.fte.kmutnb.ac.th)
sub_filter แปลง https://aiet... http://aiet... → /aiet/
href="/ src="/ action="/ → /aiet/
ปล่อย protocol-relative (href="//) ไว้ ห้ามแปลง
```

## คำสั่งจัดการ

```bash
pct exec 2004 -- systemctl status apache2
pct exec 2004 -- php -v
pct restart 2004
```

## ประวัติการย้าย

- ย้ายด้วย **Workflow 2: Zero-Downtime Live Migration** (ใช้สคริปต์ `migrate_aiet.py` อัตโนมัติ)
- **ต้นทาง VM 61 บน ESXi ยังคงเปิดทำงานปกติ** (IP `192.168.92.18`) — ใช้ IP ใหม่ `.57` + MAC ใหม่ `BC:24:11:30:04:19`
- Network ตั้งค่าด้วย `setup_aiet_network.py` (Netplan / NetworkManager ของ Ubuntu 24.04)

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · วิธีย้าย: [[VM Migration Playbook]] · Routing: [[Gateway Nginx Setup]] · เครื่องเดิม: [[ESXi Legacy Servers]]*
