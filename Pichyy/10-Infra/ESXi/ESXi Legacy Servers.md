---
tags:
  - proxmox
  - vmware-esxi
  - backup
  - legacy
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
---

# ESXi Legacy Servers (สถานะเครื่องเดิม)

> ดูรายชื่อครบทั้ง 3 โฮสต์ / 46 VM ได้ที่ [[ESXi Cluster Audit - 3 Hosts 46 VMs]]


> เครื่องเดิมบน VMware ESXi ที่ย้ายข้อมูลมา Proxmox แล้ว — ยังคงสถานะเปิด/ปิดตามข้อตกลง Zero-Downtime

## ESXi Host เดิม

| รายการ | รายละเอียด |
| :--- | :--- |
| IP | `192.168.92.250` |
| Datastore | `datastore43.3` |
| ESXi Version | 6.0 / 6.7 |
| ช่องทางดาวน์โหลด | ESXi Kernel Web Server (HTTPS — เร็วกว่า SSH 2–3 เท่า) |

## สถานะ VM แต่ละตัวหลังย้าย

| VM เดิม | ระบบ | สถานะบน ESXi | หมายเหตุ |
| :--: | :--- | :--- | :--- |
| VM 53 | NC + IC | ** Powered ON ** | ยังให้บริการปกติที่ `192.168.92.55` — ไม่ชนกับ [[CT 2003 - app-nc-ic-lxc]] (ใช้ `.56`) |
| VM 61 | AIET | ** Powered ON ** | ยังให้บริการปกติที่ `192.168.92.18` — ไม่ชนกับ [[CT 2004 - app-aiet-lxc]] (ใช้ `.57`) |
| VM 45 | Open House | ** Powered OFF ** | ปิดสำรองไว้บน `datastore43.3` |
| VM 32 | Edoc | ** Powered OFF ** | ปิดสำรองไว้บน `datastore43.3` |
| VM 58 | `webfte` (เว็บคณะ TH) | ** Powered ON ** | ยังให้บริการที่ `192.168.92.34` — ปลายทาง [[CT 2005 - app-fte-web-lxc]] (ใช้ `.134`) |
| VM 48 | `test web` (เว็บคณะ EN) | ** Powered ON ** | ยังให้บริการที่ `192.168.92.29` — ปลายทาง [[CT 2005 - app-fte-web-lxc]] |
| VM 47 | `Journal FTE New` (JTechEE) | ** Powered ON ** | ยังให้บริการที่ `192.168.92.19` — ปลายทาง [[CT 2006 - app-jtechee-lxc]] (ใช้ `.119`) |

> [!note] รอบอัปเดต 15 ต.ค. 2569
> เพิ่มการย้าย 3 เครื่อง (VM 58, 48, 47) แบบ **Zero Downtime** — เครื่องต้นทางทั้งหมดยังเปิดทำงาน 100% (ดู [[Update Log]])

> [!note] ทำไมไม่ชน IP/MAC
> เครื่องใหม่บน Proxmox ใช้ IP ชุดสำรอง (`.56`, `.57`) และกำหนด MAC Address อิสระใหม่ จึงไม่เกิด IP Collision หรือ MAC Flapping บน Switch ของมหาวิทยาลัย

## การลบ Snapshot ชั่วคราวหลังย้ายเสร็จ

```bash
# ตรวจสอบ snapshot ที่ค้างอยู่
ssh root@192.168.92.250 "vim-cmd vmsvc/getallvms"
ssh root@192.168.92.250 "vim-cmd vmsvc/snapshot.get <VMID>"

# ลบ snapshot mig-snap ที่สร้างไว้ตอนย้าย
ssh root@192.168.92.250 "vim-cmd vmsvc/snapshot.remove <VMID> <SnapId> 0"
```

## ก่อนปิด ESXi ตัวเดิมถาวร

- [ ] ยืนยันว่าทุกระบบบน [[Host FTEPVE]] ทำงานปกติ 100%
- [ ] สำรอง Golden Backup (VZDump) ของทุก CT
- [ ] ตรวจสอบข้อมูลล่าสุดใน [[CT 2000 - core-db-mariadb]] ตรงกับของจริง
- [ ] ตกลงกับผู้เกี่ยวข้องก่อนปิด VM 53 / VM 61
- [ ] อ่านรายงานครบถ้วน [[ESXi Cluster Audit - 3 Hosts 46 VMs]]



---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · วิธีย้าย: [[VM Migration Playbook]]*
