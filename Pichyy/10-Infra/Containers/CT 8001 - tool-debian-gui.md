---
tags:
  - proxmox
  - lxc
  - jumpbox
  - xrdp
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
ctid: 8001
---

# CT 8001 — tool-debian-gui (Desktop Jumpbox)

> [!info] ภาพรวม
> Container เดสก์ท็อป XFCE4 + XRDP ใช้เป็น Jumpbox เปิด Web UI หรือจัดการระบบภายในเครือข่าย

## สเปก

| รายการ | รายละเอียด |
| :--- | :--- |
| CTID | `8001` (เดิม `102`) |
| Hostname | `tool-debian-gui` (เดิม `debian-gui`) |
| OS | Debian 12 + **XFCE4** (Lightweight Desktop) |
| Remote | **XRDP** port `3389` / **noVNC** port `6080` |
| สเปก | 2 vCPU / 4 GB RAM / 15 GB NVMe (`local-lvm`) |
| IP | `192.168.92.212/24` (GW `.1`) |
| Auto-Start | `onboot=1` |
| ติดตั้งเพิ่ม | Firefox ESR, Thai/CJK Fonts, curl, wget, git, net-tools |

## บัญชีเข้าใช้งาน

| บัญชี | Username | สิทธิ์ |
| :--- | :--- | :--- |
| ผู้ใช้หลัก (GUI) | `debian` | sudo (full root) |
| ผู้ดูแลระบบ | `root` | root |

> [!warning] รหัสผ่าน
> ดูรหัสผ่านใน `SYSTEM_HANDOVER_CREDENTIALS.txt` (โฟลเดอร์ต้นฉบับ)

## วิธีเข้าใช้งาน

### วิธี 1: noVNC ผ่าน Web Browser (สะดวกที่สุด)
```bash
ssh -L 6080:192.168.92.212:6080 root@202.44.35.40
# แล้วเปิด http://localhost:6080/ → กด Connect → ใส่รหัสผ่าน
```
*(อยู่ในวงใน/VPN เปิด `http://192.168.92.212:6080/` ได้ตรง)*

### วิธี 2: Remote Desktop (mstsc / XRDP)
```bash
ssh -L 3389:192.168.92.212:3389 root@202.44.35.40
```
แล้วเปิด `mstsc` → Computer: `localhost:3389` → Session: `Xorg` → login `debian`

## คำสั่งจัดการจาก Host

```bash
pct status 8001
pct enter 8001
pct exec 8001 -- systemctl status xrdp
pct exec 8001 -- systemctl restart xrdp
```

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · เจ้าของ Host: [[Host FTEPVE]]*
