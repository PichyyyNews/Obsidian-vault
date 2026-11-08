---
tags:
  - proxmox
  - infrastructure
  - host
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
---

# Host FTEPVE (Proxmox VE)

> [!info] ภาพรวม
> เครื่องแม่ข่าย Proxmox VE 9.1.1 ตัวเดียวของคณะ รันบทบาท Edge Gateway + Hypervisor + Backup Storage

## สเปกฮาร์ดแวร์/ระบบ

| รายการ | รายละเอียด |
| :--- | :--- |
| Hostname | `FTEPVE` |
| OS | Proxmox VE 9.1.1 (Debian 13 Trixie) |
| Kernel | `6.17.2-1-pve` |
| CPU | Intel Xeon Silver 4509Y |
| RAM | 64 GB (ใช้ได้ ~62 GiB) |
| Public IP | `202.44.35.40` |
| Internal IP | `192.168.92.200/24` (Interface `vmbr0.266`, VLAN 266) |
| Gateway | `192.168.92.1` |
| SSH | Port 22 |
| Web GUI | Port 8006 (HTTPS, Self-Signed) |

## ช่องทางเข้าถึง

```bash
# SSH (Password อยู่ใน SYSTEM_HANDOVER_CREDENTIALS.txt)
ssh root@202.44.35.40

# SSH ด้วย Key (ไม่ต้องกรอกรหัสผ่าน)
ssh -i ftesshkey root@202.44.35.40

# Web GUI (เบราว์เซอร์จะเตือน Self-Signed — กด Advanced → Proceed)
https://202.44.35.40:8006/
```

> [!tip] VS Code Remote-SSH
> เพิ่มใน `~/.ssh/config`:
> ```
> Host Proxmox-FTE
>     HostName 202.44.35.40
>     User root
>     Port 22
>     IdentityFile "c:/Users/Newsk/OneDrive/Desktop/proxmox-fte/ftesshkey"
> ```

## Storage Pools ทั้งหมด 3 Pool

| Pool | ชนิด | ขนาด | Used หลังย้าย | เก็บอะไร |
| :--- | :--- | :--- | :--- | :--- |
| `local` | Directory | ~100 GB | 4.45% (ว่าง 85 GB) | OS, ISO, Backup VZDump |
| `local-lvm` | LVM-Thin (NVMe SSD) | 318.5 GB | 2.31% (ว่าง 311 GB) | CT 2000 (DB), CT 1001, CT 8001 |
| `storage-raid` | LVM-Thin (MegaRAID MR9460-8i SAS) | 3.32–3.56 TB | 1.92% (ว่าง 3.33 TB) | CT 2001–2004 (Web Apps) |

> [!note] ผลลัพธ์การคืนพื้นที่
> หลังลบ VM KVM เดิม 3001–3004 + Temporary Volumes คืนพื้นที่ได้รวม **+766 GB** (NVMe +301 GB, SAS RAID +465 GB)

## คำสั่ง CLI ที่ใช้บ่อย

```bash
pveversion -v              # ดูเวอร์ชัน PVE และ sub-packages
pvesm status               # สถานะ Storage Pools
qm list                    # รายชื่อ VM ทั้งหมด
pct list                   # รายชื่อ LXC ทั้งหมด
pct enter <CTID>           # เข้า shell ใน container
pct exec <CTID> -- <cmd>   # รันคำสั่งใน container
pct restart <CTID>         # restart container
lvs vg-raid                # เช็คพื้นที่ RAID 3.5 TB
systemctl status pveproxy  # สถานะ Web UI service
systemctl reload nginx     # reload edge nginx หลังแก้คอนฟิก
ip a && df -h && free -h   # เช็ค network / disk / ram
```

## Network & Security

- Firewall ส่งต่อเฉพาะ port **22 (SSH)** และ **8006 (Web GUI)**
- ถ้าจะเปิด Web ออกภายนอก ให้เปิด Forward port 80/443 ไปที่ CT 1001 (`192.168.92.211`) แล้วให้ NPM จัดการ SSL Let's Encrypt
- แนะนำ: SSH Key-based Auth + Fail2ban

## ไฟล์คอนฟิกที่สำคัญบน Host

- Edge Nginx: `/etc/nginx/sites-available/pve_openhouse.conf` (รายละเอียดที่ [[Gateway Nginx Setup]])
- Backup Job: `/etc/pve/jobs.cfg` (รายละเอียดที่ [[Backup Plan FTE]])
- SSL ของ PVE: `/etc/pve/local/pve-ssl.pem` + `pve-ssl.key`

---
*ดูภาพรวมโปรเจกต์ทั้งหมด: [[Proxmox VE Migration FTE]]*
