---
tags:
  - proxmox
  - vmware-esxi
  - migration
  - infrastructure
  - knowledge
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
---

# Proxmox VE Migration — FTE KMUTNB

> [!summary] สรุปสั้น
> ย้าย VM ระบบงานคณะครุศาสตร์อุตสาหกรรม (มจพ.) จาก **VMware ESXi** ไป **Proxmox VE 9.1.1** เสร็จ 100% — ปัจจุบันมี **8 ระบบงาน / 10 ฐานข้อมูล + Portal กลาง** ใช้ LXC + Centralized MariaDB + Unified Reverse Proxy คืนพื้นที่ storage 766 GB
>
> 🔄 **อัปเดตล่าสุด (15 ต.ค. 2569):** เพิ่ม CT 2005 (เว็บคณะ TH+EN), CT 2006 (วารสาร), **CT 2007 (FTE App Portal `/portal/`)** และรายงานสำรวจ **ESXi 3 โฮสต์ / 46 VM** — ดู [[Update Log]]

## ข้อมูลโปรเจกต์

| รายการ | รายละเอียด |
| :--- | :--- |
| เจ้าของระบบ | คณะครุศาสตร์อุตสาหกรรม มจพ. (FTE KMUTNB) |
| วันที่ส่งมอบ | 15 ตุลาคม 2569 |
| Proxmox Host | `FTEPVE` — Public `202.44.35.40` / Internal `192.168.92.200` (VLAN 266) |
| สเปก Host | Proxmox VE 9.1.1, Xeon Silver 4509Y, RAM 64 GB |
| สถานะ | Production Ready — เปิดบริการครบทุก path |

## สถาปัตยกรรมปัจจุบัน

```
[Client] → Edge Nginx บน Host (202.44.35.40:80/443)
              ├─ / และ /pve → Proxmox GUI (127.0.0.1:8006)
              └─ /openhouse/ /edoc/ /nc/ /ic/ /aiet/ /fte-web/ /jtechee/ /portal/
                     ↓
           CT 1001 core-npm-proxy (192.168.92.211)
              ทำ sub_filter rewrite พาธ/asset รายระบบ
                     ↓
   CT 2001–2006 (Web Apps บน SAS RAID) · CT 2007 (Portal static)
                     ↓ (MariaDB 3306 — LAN เท่านั้น)
           CT 2000 core-db-mariadb (192.168.92.210, NVMe)
```

## ตาราง Container ทั้งหมด

| CTID | ชื่อ                | ระบบงาน                             | IP ภายใน | Storage              | URL ผ่าน Gateway |
| :--: | :------------------ | :---------------------------------- | :------- | :------------------- | :--------------- |
| 2000 | `core-db-mariadb`   | MariaDB 10.11 (DB รวม 10 ฐาน)      | .210     | local-lvm (NVMe 25G) | —                |
| 2001 | `app-openhouse-lxc` | WordPress Open House (PHP 7.2)      | .26      | storage-raid (15G)   | `/openhouse/`    |
| 2002 | `app-edoc-lxc`      | ระบบสารบรรณอิเล็กทรอนิกส์ (PHP 7.4) | .25      | storage-raid (80G)   | `/edoc/`         |
| 2003 | `app-nc-ic-lxc`     | ประชุมวิชาการ NC + IC (PHP 7.2)     | .56      | storage-raid (20G)   | `/nc/` `/ic/`    |
| 2004 | `app-aiet-lxc`      | ศูนย์นวัตกรรม AIET (PHP 8.3)        | .57      | storage-raid (15G)   | `/aiet/`         |
| 2005 | `app-fte-web-lxc`   | เว็บคณะ TH + EN (PHP 7.2)           | .134     | storage-raid (40G)   | `/fte-web/th/` `/fte-web/en/` |
| 2006 | `app-jtechee-lxc`   | วารสาร JTechEE (PHP 7.4)            | .119     | storage-raid (20G)   | `/jtechee/`      |
| 2007 | `app-portal-lxc`    | FTE App Portal (SPA static/Nginx)    | .213     | storage-raid (10G)   | `/portal/`       |
| 1001 | `core-npm-proxy`    | Nginx Proxy Manager (Docker)        | .211     | local-lvm (10G)      | —                |
| 8001 | `tool-debian-gui`   | XFCE4 + XRDP jumpbox                | .212     | local-lvm (15G)      | —                |

## 📂 โน้ตย่อยรายละเอียด

### ⭐ อัปเดตล่าสุด
- [[Update Log]] — สรุปสิ่งที่เพิ่ม/เปลี่ยนรอบล่าสุด (CT 2005–2007, Portal, ESXi Audit, DB 10 ตัว)

### โครงสร้างพื้นฐาน
- [[Host FTEPVE]] — สเปก Host, Storage Pools, คำสั่ง CLI, การเข้าถึง
- [[Gateway Nginx Setup]] — Edge Nginx + Core NPM sub_filter ละเอียดทุก Path
- [[Backup Plan FTE]] — ระบบสำรองข้อมูล 2 ชั้น (mysqldump + VZDump)

### ระบบฐานข้อมูล
- [[CT 2000 - core-db-mariadb]] — MariaDB รวมศูนย์, Grants, Buffer Pool, Cron Dump

### ระบบงาน Web (Application Containers)
- [[CT 2001 - app-openhouse-lxc]] — WordPress Open House
- [[CT 2002 - app-edoc-lxc]] — ระบบสารบรรณอิเล็กทรอนิกส์
- [[CT 2003 - app-nc-ic-lxc]] — ประชุมวิชาการ NC + IC
- [[CT 2004 - app-aiet-lxc]] — ศูนย์นวัตกรรม AIET
- [[CT 2005 - app-fte-web-lxc]] — เว็บคณะ TH + EN
- [[CT 2006 - app-jtechee-lxc]] — วารสาร JTechEE
- [[CT 2007 - app-portal-lxc]] — FTE App Portal (หน้าแรก `/portal/`)
- [[FTE App Portal]] — ตัวแอป Portal (React + GitHub Primer)

### ระบบเดิมบน ESXi
- [[ESXi Cluster Audit - 3 Hosts 46 VMs]] — บัญชีรายชื่อ ESXi ทั้ง 3 โหนด / 46 VM
- [[ESXi Host 2 - Spec & Stack]] — วิเคราะห์เชิงลึก AD/DC, FS7, vNIOS, Net_Monitor, Tech
- [[ESXi Host 2 - Easy Migration List]] — ลิสต์เริ่มต้น (Tech, Net_Monitor)

### เครื่องมือและเครื่องเดิม
- [[CT 1001 - core-npm-proxy]] — Nginx Proxy Manager
- [[CT 8001 - tool-debian-gui]] — Desktop Jumpbox (XFCE4/XRDP)
- [[VM Migration Playbook]] — SOP มาตรฐานการย้าย VM 2 แบบ
- [[ESXi Legacy Servers]] — สถานะเครื่องเดิมบน ESXi

> [!note] หลักการออกแบบ
> Web App ทั้งหมดอยู่บน **SAS HDD RAID** (`storage-raid`) แต่ Database กลางอยู่บน **NVMe SSD** (`local-lvm`) เพื่อ I/O สูงสุด ทุก CT เปิด `onboot=1`

## Storage Pools

| Pool | ชนิด | ขนาด | ใช้เก็บ |
| :--- | :--- | :--- | :--- |
| `local` | Directory | ~94 GB | OS, ISO, Backup VZDump |
| `local-lvm` | LVM-Thin (NVMe) | ~318 GB | CT 2000, 1001, 8001 |
| `storage-raid` | LVM-Thin (MegaRAID MR9460-8i SAS) | 3.56 TB | CT 2001–2007 |

## บทเรียนการย้าย VM: 2 Workflow มาตรฐาน

### 1. Direct Raw Block Stream — เครื่องต้นทางปิดได้
ใช้กับ VM 3001 (Open House), 3002 (Edoc)

1. รวม snapshot บน ESXi: `vmkfstools -i <snapshot>.vmdk -d thin <target-flat>.vmdk`
2. Power off เครื่องต้นทาง
3. สร้าง VM เปล่าบน Proxmox (`qm create ... --scsihw virtio-scsi-pci --net0 vmxnet3,tag=266`)
4. สตรีมตรงเข้า block storage: `ssh root@ESXi "cat <flat>.vmdk" | dd of=/dev/pve/vm-<ID>-disk-0 bs=4M conv=sparse`

### 2. Zero-Downtime Live Migration — เครื่องห้ามดับ
ใช้กับ VM 3003 (NC+IC), 3004 (AIET) — ต้นทางเปิดต่อได้ปกติ

1. สร้าง snapshot ชั่วคราวปลด lock: `vim-cmd vmsvc/snapshot.create <ID> "mig-snap" "migration" 0 0`
2. ดาวน์โหลดผ่าน ESXi Kernel Web Server (75–80 MB/s, เร็วกว่า SSH 2–3 เท่า, resume ได้ด้วย `curl -C -`)
3. รวม chain ด้วย descriptor 2 ไฟล์ (base + snap) แล้ว `qemu-img convert -f vmdk -O raw snap.vmdk /dev/vg-raid/vm-<ID>-disk-0`
4. **แก้ IP/MAC ใน Guest ก่อนบูต** ด้วย `kpartx -av` + mount partition (ป้องกัน collision)
5. Start VM แล้วลบ snapshot ชั่วคราวบน ESXi

### Do & Don't

| ควรทำ ✅ | ห้ามทำ ❌ |
| :--- | :--- |
| ใช้ ESXi Kernel Web Server โอนข้อมูล | ดาวน์โหลด VMDK ไปพักบน Windows ผ่าน Wi-Fi |
| Snapshot ชั่วคราวเสมอถ้าต้นทางยังเปิดอยู่ | อ่าน Active Delta Disk ที่กำลังเขียน (ติด Lock) |
| ตั้ง IP/MAC ใหม่แยกจากของเดิม | ใช้ IP/MAC เดิม VLAN เดียวกัน (เกิด MAC Flapping) |
| ตรวจ UUID ใน `/etc/fstab` ของ Guest เสมอ | เปลี่ยน mount point เป็น `/dev/sda` มั่ว ๆ |
| ล้าง `/mnt/temp-mig` หลัง merge เสร็จ | ปล่อยไฟล์ VMDK ชั่วคราวค้างใน storage |

## แผนสำรองข้อมูล 2 ชั้น

1. **Proxmox VZDump** — ทุกวัน 01:00 น., Mode Snapshot, เก็บ 7 วัน
2. **mysqldump อัตโนมัติใน CT 2000** — ทุกวัน 00:00 น., แยก DB, gzip, เก็บ 14 วัน, ลบอัตโนมัติด้วย `find -mtime +14 -delete`
   - Restore เฉพาะระบบเดียวได้ใน ~30 วินาที

## คำสั่งที่ใช้บ่อย (PVE Host)

```bash
pct list && qm list          # ดู container/VM ทั้งหมด
pct enter <CTID>             # เข้า shell ใน container
pct exec <CTID> -- <cmd>     # รันคำสั่งใน container
lvs vg-raid                  # เช็คพื้นที่ RAID 3.5 TB
systemctl reload nginx       # reload edge proxy บน host
```

เส้นทางคอนฟิกสำคัญ:
- Edge Nginx: `/etc/nginx/sites-available/pve_openhouse.conf`
- Core NPM: `/opt/npm/data/nginx/custom/http.conf` (ใน CT 1001)
- Sync ทั้งหมด: `bash /root/update_proxy_routes.sh`

> [!warning] ความปลอดภัย
> รหัสผ่านและ SSH key **ไม่ได้บันทึกในโน้ตนี้** — ข้อมูล credential ทั้งหมดอยู่ในไฟล์ต้นฉบับ `SYSTEM_HANDOVER_CREDENTIALS.txt` และ `ftesshkey` ในโฟลเดอร์ source (ดูด้านล่าง) เก็บเป็นความลับ ห้ามแชร์ต่อ

## ไฟล์อ้างอิงต้นฉบับ

- `LATEST_UPDATE_NOTES.md` — บันทึกสรุปการอัปเดตล่าสุด (CT 2005–2007 + ESXi survey)
- `ESXI_INFRASTRUCTURE_AUDIT.md` — รายงานสำรวจ ESXi 3 โหนด / 46 VM
- `README.md` — คู่มือเชื่อมต่อและข้อมูลเซิร์ฟเวอร์
- `HANDOFF.md` — เอกสารส่งมอบฉบับสมบูรณ์
- `MIGRATION_PLAYBOOK.md` — SOP การย้าย VM
- `LXC_MIGRATION_AND_DB_PLAN.md` — แผนสถาปัตยกรรม LXC + Central DB
- `SYSTEM_HANDOVER_CREDENTIALS.txt` — ข้อมูลรหัสผ่าน (ลับ)
- ตำแหน่ง: `C:\Users\Newsk\Desktop\proxmox-mrigate`

---

*กลับไปที่: [[00-Home]]*
