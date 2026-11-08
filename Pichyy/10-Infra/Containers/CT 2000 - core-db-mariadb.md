---
tags:
  - proxmox
  - lxc
  - database
  - mariadb
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
ctid: 2000
---

# CT 2000 — core-db-mariadb (ฐานข้อมูลรวมศูนย์)

> [!info] ภาพรวม
> MariaDB 10.11 Server รวมศูนย์ทุกระบบไว้ที่เดียว อยู่บน NVMe SSD เพื่อ I/O สูงสุด ใช้ร่วมกับ [[CT 2001 - app-openhouse-lxc]], [[CT 2002 - app-edoc-lxc]], [[CT 2003 - app-nc-ic-lxc]], [[CT 2004 - app-aiet-lxc]]

## สเปก

| รายการ | รายละเอียด |
| :--- | :--- |
| CTID | `2000` |
| Hostname | `core-db-mariadb` |
| OS | Debian 12 |
| สเปก | 4 vCPU / 8 GB RAM / 25 GB NVMe SSD (`local-lvm`) |
| IP | `192.168.92.210/24` (Gateway `.1`, DNS `202.44.32.29`) |
| Engine | MariaDB 10.11.19 |
| Listen | `0.0.0.0:3306` (รับเฉพาะวง `192.168.92.%` + localhost) |
| Buffer Pool | InnoDB `4 GB` |
| Max Connections | `500` |

## ฐานข้อมูลและบัญชีผู้ใช้

| Database | User | เชื่อมต่อจาก | ระบบเจ้าของ |
| :--- | :--- | :--- | :--- |
| `fte_openhouse` | `admin` | `192.168.92.%`, localhost | [[CT 2001 - app-openhouse-lxc]] |
| `FTE_Edoc` | `adminEdoc` | `192.168.92.%`, localhost | [[CT 2002 - app-edoc-lxc]] |
| `web_nc`, `web_ic`, `conference_db`, `web_student` | `fte` | `192.168.92.%`, localhost | [[CT 2003 - app-nc-ic-lxc]] |
| `aiet` | `aiet` | `192.168.92.%`, localhost | [[CT 2004 - app-aiet-lxc]] |
| `km_fte` | `phai` | `192.168.92.%`, localhost | [[CT 2005 - app-fte-web-lxc]] (เว็บคณะ TH) |
| `web_en` | `admin_en` | `192.168.92.%`, localhost | [[CT 2005 - app-fte-web-lxc]] (เว็บคณะ EN) |
| `web_jtechee` | `jtech` | `192.168.92.%`, localhost | [[CT 2006 - app-jtechee-lxc]] |

> [!note] นับรวมทั้งหมด **10 ฐานข้อมูล** (เพิ่ม 3 ตัวจากรอบอัปเดต 15 ต.ค. 2569 — ดู [[Update Log]])

> [!warning] รหัสผ่าน
> รหัสผ่าน DB users ทั้งหมดอยู่ใน `SYSTEM_HANDOVER_CREDENTIALS.txt` (โฟลเดอร์ต้นฉบับ) — ไม่เก็บในโน้ตนี้

## ตั้งค่า my.cnf สำคัญ

```ini
bind-address = 0.0.0.0
max_connections = 500
innodb_buffer_pool_size = 4G
```

## สำรองข้อมูลอัตโนมัติ (Cron 00:00 น.)

- สคริปต์: `/usr/local/bin/backup_databases.sh`
- เก็บที่: `/var/backups/mariadb/`
- แยกรายฐาน → `.sql.gz` → เก็บ 14 วัน (ลบอัตโนมัติ)
- ครอบคลุม **10 ฐานข้อมูล** (เพิ่มจาก 5 เป็น 10 ในรอบอัปเดต 15 ต.ค. 2569)

```bash
DBS=("fte_openhouse" "FTE_Edoc" "web_nc" "web_ic" "conference_db" "web_student" "aiet" "km_fte" "web_en" "web_jtechee")
for DB in "${DBS[@]}"; do
  mariadb-dump -u root -p<ROOT_PW> --single-transaction --quick --routines --triggers "$DB" | \
    gzip > /var/backups/mariadb/${DB}_$(date +%Y%m%d_%H%M%S).sql.gz
done
find /var/backups/mariadb -type f -name "*.sql.gz" -mtime +14 -exec rm {} +
```

Restore เฉพาะระบบเดียวได้ใน ~30 วินาที (ดูรายละเอียด [[Backup Plan FTE]])

## คำสั่งจัดการ

```bash
pct exec 2000 -- systemctl status mariadb
pct exec 2000 -- mariadb -u root -p
pct exec 2000 -- df -h /var/lib/mysql
```

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · เจ้าของโครงสร้าง Backup: [[Backup Plan FTE]]*
