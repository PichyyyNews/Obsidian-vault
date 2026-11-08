---
tags:
  - proxmox
  - backup
  - mariadb
  - vzdump
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
---

# Backup Plan FTE (สำรองข้อมูล 2 ชั้น)

> [!info] ภาพรวม
> ระบบสำรองข้อมูลอัตโนมัติ 2 ระดับ — Logical Dump รายฐานข้อมูล + Container Snapshot ระดับ Host

## ชั้นที่ 1: Logical Database Dumps (CT 2000)

| รายการ | รายละเอียด |
| :--- | :--- |
| ระดับ | รายฐานข้อมูล (Table-Level Granular Recovery) |
| ที่ | [[CT 2000 - core-db-mariadb]] |
| สคริปต์ | `/usr/local/bin/backup_databases.sh` |
| เวลา | ทุกวัน **00:00 น.** (Cron) |
| เก็บที่ | `/var/backups/mariadb/` |
| รูปแบบ | `.sql.gz` แยกรายฐาน |
| Retention | **14 วัน** (ลบอัตโนมัติ `find -mtime +14 -delete`) |

```bash
DBS=("fte_openhouse" "FTE_Edoc" "web_nc" "web_ic" "conference_db" "web_student" "aiet" "km_fte" "web_en" "web_jtechee")
for DB in "${DBS[@]}"; do
  mariadb-dump -u root -p<ROOT_PW> --single-transaction --quick --routines --triggers "$DB" | \
    gzip > /var/backups/mariadb/${DB}_$(date +%Y%m%d_%H%M%S).sql.gz
done
find /var/backups/mariadb -type f -name "*.sql.gz" -mtime +14 -exec rm {} +
```

**ประโยชน์:** ผู้ใช้เผลอลบ/แก้ข้อมูลผิดในระบบใดระบบหนึ่ง → restore เฉพาะ `.sql.gz` ของระบบนั้นใน ~30 วินาที ไม่ต้องย้อนระบบอื่น

> [!note] อัปเดต 15 ต.ค. 2569
> จำนวนฐานข้อมูลที่ dump เพิ่มจาก 5 → **10 ฐาน** และเปลี่ยนจาก `mysqldump` เป็น `mariadb-dump` (ดู [[Update Log]])

## ชั้นที่ 2: Container Snapshot VZDump (Host Level)

| รายการ | รายละเอียด |
| :--- | :--- |
| ระดับ | Full Container (Disaster Recovery) |
| ที่ | [[Host FTEPVE]] |
| คอนฟิก | `/etc/pve/jobs.cfg` |
| เวลา | ทุกวัน **01:00 น.** |
| Mode | **Snapshot** (ทำงานขณะระบบเปิด — ไม่ดับ ไม่สะดุด) |
| Compression | `zstd` (Ultra-fast) |
| เป้าหมาย | CT **2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007** (8 ตัว) |
| Retention | **7 วัน** ล่าสุด |
| Storage | `local` หรือ External |

## ลำดับเวลาสำรองรายวัน

```
00:00 → mysqldump รายฐาน (CT 2000) → /var/backups/mariadb/*.sql.gz
01:00 → VZDump Snapshot ทุก CT (Host) → /var/lib/vz/dump/
```

## คำสั่ง Manual Backup / Restore

```bash
# Backup CT 2000 ด้วยมือ
vzdump 2000 --storage local --mode snapshot --compress zstd

# Restore จากไฟล์ .sql.gz
gunzip < fte_openhouse_20261015_000000.sql.gz | mysql -u admin -p fte_openhouse

# ดูไฟล์ backup ที่มี
ls -lh /var/lib/vz/dump/
ls -lh /var/backups/mariadb/
```

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · DB Server: [[CT 2000 - core-db-mariadb]] · Host: [[Host FTEPVE]]*
