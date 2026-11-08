---
tags:
  - esxi
  - inventory
  - audit
  - migration
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate\ESXI_INFRASTRUCTURE_AUDIT.md
status: complete
---

# ESXi Cluster Audit — 3 โฮสต์ / 46 Virtual Machines

> [!summary] สรุป
> รายงานสำรวจเครื่องแม่ข่าย **VMware ESXi ทั้ง 3 โหนด** ของคณะครุศาสตร์อุตสาหกรรม มจพ. เข้าถึงและบันทึกข้อมูลครบ **100% รวม 46 VM** (เปิด 21 / ปิด 25) — ใช้เป็นบัญชีรายชื่อระบบเดิมก่อน/หลังย้ายเข้า [[Proxmox VE Migration FTE]]

> [!warning] ข้อมูลลับ
> **รหัสผ่าน root ของ ESXi ทั้ง 3 โหนด ไม่ถูกเก็บไว้ใน vault นี้** (sync ผ่าน Obsidian Sync) ให้ดูที่ `SYSTEM_HANDOVER_CREDENTIALS.txt` และ `ESXI_INFRASTRUCTURE_AUDIT.md` ในโฟลเดอร์ต้นทางเท่านั้น

## 1. สรุป 3 โหนด

| รายการ | โหนด 1 (Main) | โหนด 2 | โหนด 3 |
| :--- | :--- | :--- | :--- |
| IP จัดการ (สาธารณะ) | `202.44.35.3` | `202.44.35.4` | `202.44.35.5` |
| IP ภายใน | `192.168.92.250` | ผ่าน VLAN 102 | ผ่าน VLAN 102 |
| VLAN Management | VLAN 102 | VLAN 102 | VLAN 102 |
| VMware ESXi | **6.5.0** | **6.0.0** (build-6921384) | **5.5.0** (build-1331820) |
| พอร์ตเปิด | 22, 80, 443, 902 | 80, 443, 902, 8000, 8100, 8300 | 80, 443, 902, 8000, 8100 |
| จำนวน VM | **17** (เปิด 10 / ปิด 7) | **18** (เปิด 6 / ปิด 12) | **11** (เปิด 5 / ปิด 6) |
| บทบาท | ต้นทางย้ายเว็บคณะ → Proxmox | AD, File Server, Network Infra | ระบบ Legacy (Web/DB รุ่นแรก) |

> [!note] โหนด 3 เข้าถึงผ่าน **TLS 1.0 / Legacy Ciphers** (ESXi 5.5)

## 2. โหนด 1 — `202.44.35.3` (17 VM)

- **OS:** ESXi 6.5.0 · **Datastore:** `datastore43.3` (7.44 TB / ว่าง 5.16 TB)
- เป็นโฮสต์ต้นทางที่ย้ายเว็บขึ้น LXC แล้ว (CT 2001–2006)

| VMID | ชื่อ VM | Power | IP | รายละเอียด / สถานะย้าย |
| :--: | :--- | :--: | :--- | :--- |
| 58 | `webfte 2024` | 🟢 ON | `192.168.92.34` | เว็บคณะ TH/EN → [[CT 2005 - app-fte-web-lxc]] (คู่ขนาน, Zero Downtime) |
| 48 | `202.44.35.29-web-en` | 🟢 ON | `192.168.92.29` | เว็บอังกฤษเดิม → รวมใน CT 2005 `/fte-web/en/` |
| 47 | `Journal FTE New` | 🟢 ON | `192.168.92.19` | วารสาร JTechEE → [[CT 2006 - app-jtechee-lxc]] |
| 53 | `NC + IC 202.44.35.24` | 🟢 ON | `192.168.92.24` | ประชุมวิชาการ → [[CT 2003 - app-nc-ic-lxc]] |
| 61 | `aiet fte` | 🟢 ON | `192.168.92.57` | AIET → [[CT 2004 - app-aiet-lxc]] |
| 36 | `vijai 2019 FTE` | 🟢 ON | `192.168.92.55` | ระบบงานวิจัยคณะ **(ยังอยู่บน ESXi)** |
| 30 | `Ubuntu 18.0 karupan New` | 🟢 ON | `192.168.92.18` | พัสดุ/ครุภัณฑ์ **(ยังอยู่บน ESXi)** |
| 60 | `IPR System` | 🟢 ON | `192.168.92.17` | ทรัพย์สินทางปัญญา **(ยังอยู่บน ESXi)** |
| 66 | `Conference` | 🟢 ON | `192.168.92.14` | ลงทะเบียน/จัดประชุม **(ยังอยู่บน ESXi)** |
| 62 | `Fte Web` | 🟢 ON | — | เว็บคณะเวอร์ชันสำรอง |
| 45 | `Open House FTE` | ⚪ OFF | `192.168.92.26` | ย้ายแล้ว → [[CT 2001 - app-openhouse-lxc]] (ปิดเครื่องเดิม) |
| 32 | `Web EDoc New` | ⚪ OFF | `192.168.92.25` | ย้ายแล้ว → [[CT 2002 - app-edoc-lxc]] (ปิดเครื่องเดิม) |
| 3 | `Win Server 2012 DB Edoc` | ⚪ OFF | `192.168.92.20` | DB ย้ายเข้า [[CT 2000 - core-db-mariadb]] |
| 17 | `Ubuntu web TMEDU 101` | ⚪ OFF | — | เว็บภาควิชาเครื่องกล (เลิกใช้) |
| 64 | `Tech EDu` | ⚪ OFF | — | ระบบเดิม (ปิดใช้งาน) |
| 2 | `Windows 2012 R12_Cloud` | ⚪ OFF | — | เครื่องทดสอบ |
| 37 | `vNIOS` | ⚪ OFF | — | Infoblox vNIOS (ปิดใช้งาน) |

## 3. โหนด 2 — `202.44.35.4` (18 VM)

- **OS:** ESXi 6.0.0 · ระบบเด่น: **Active Directory, File Server, Network Infra**

| # | ชื่อ VM | Power | IP | วัตถุประสงค์ |
| :-: | :--- | :--: | :--- | :--- |
| 1 | `windows server 2019 AD35.28` | 🟢 ON | `202.44.35.28` | **AD Domain Controller (Primary)** — สิทธิ์/บัญชีคณะ |
| 2 | `Windows server2016 AD` | 🟢 ON | ภายใน | AD DC สำรอง (2016) |
| 3 | `Windown2008 R2_FS7` | 🟢 ON | ภายใน | **File Server (FS7)** ไฟล์แชร์กลาง |
| 4 | `vNIOS` | 🟢 ON | `192.168.2.21` | Infoblox — DNS/DHCP/IPAM |
| 5 | `Net_Monitor` | 🟢 ON | `192.168.2.107` | Network Monitoring |
| 6 | `Tech` | 🟢 ON | ภายใน | สนับสนุนงานเทคนิคสารสนเทศ |
| 7 | `Windows Server Solewinds` | ⚪ OFF | — | SolarWinds NPM |
| 8 | `android os for pc` | ⚪ OFF | — | Android x86 ทดสอบ |
| 9 | `windows Domian Do` | ⚪ OFF | — | เครื่องทดสอบโดเมน |
| 10 | `windows 2012R2 Domian Do` | ⚪ OFF | — | เครื่องทดสอบโดเมน 2012 R2 |
| 11 | `windows7 userDo` | ⚪ OFF | — | ไคลเอนต์จำลอง Win7 |
| 12 | `Windows Server 2012R2 AIET` | ⚪ OFF | — | AIET เดิม (ปลดระวาง) |
| 13 | `Windows Server 2019AD fte5d` | ⚪ OFF | — | เครื่องทดสอบ AD 2019 |
| 14 | `Windows XP` | ⚪ OFF | — | Legacy XP |
| 15 | `Windown 10 Unifi` | ⚪ OFF | — | UniFi เดิมบน Win10 |
| 16 | `UniFi Network` | ⚪ OFF | — | UniFi Controller Wi-Fi |
| 17 | `Windown7_CCTV` | ⚪ OFF | — | มอนิเตอร์ CCTV |
| 18 | `Windown7_CCTV Edu` | ⚪ OFF | — | CCTV อาคารเรียน |

## 4. โหนด 3 — `202.44.35.5` (11 VM)

- **OS:** ESXi 5.5.0 · ระบบเด่น: **Legacy Web/DB รุ่นดั้งเดิม**

| # | ชื่อ VM | Power | IP | วัตถุประสงค์ |
| :-: | :--- | :--: | :--- | :--- |
| 1 | `Server- Web FTE-35.11` | 🟢 ON | `202.44.35.11` | เว็บคณะยุคแรกสุด |
| 2 | `Win Server 2012 DB EDoc` | 🟢 ON | ภายใน | Legacy EDoc DB (Windows 2012) |
| 3 | `Win2008R2 EDoc Web` | 🟢 ON | ภายใน | Legacy EDoc Web (IIS/2008 R2) |
| 4 | `jtechee` | 🟢 ON | ภายใน | วารสาร JTechEE รุ่นแรก |
| 5 | `Windows 7` | 🟢 ON | ภายใน | Client/Service เฉพาะกิจ |
| 6 | `VMware vCenter Server Appliance` | ⚪ OFF | — | vCenter (ปิดอยู่) |
| 7 | `win 2008 R2 Domain14` | ⚪ OFF | — | โดเมนคอนโทรลเลอร์เก่า |
| 8 | `Windows XP -1` | ⚪ OFF | — | Legacy XP |
| 9 | `Fte` | ⚪ OFF | — | เครื่องทดสอบ |
| 10 | `Elementary OS` | ⚪ OFF | — | Linux Desktop ทดสอบ |
| 11 | `PFsense GW2` | ⚪ OFF | — | pfSense สำรอง |

## 5. ย้ายแล้ว vs ยังอยู่บน ESXi

### ✅ ขึ้น Proxmox แล้ว (Production 8 รายการ)
1. เว็บคณะ TH/EN → [[CT 2005 - app-fte-web-lxc]] · `/fte-web/th/`, `/fte-web/en/`
2. Edoc → [[CT 2002 - app-edoc-lxc]] · `/edoc/`
3. Open House → [[CT 2001 - app-openhouse-lxc]] · `/openhouse/`
4. NC + IC → [[CT 2003 - app-nc-ic-lxc]] · `/nc/`, `/ic/`
5. AIET → [[CT 2004 - app-aiet-lxc]] · `/aiet/`
6. JTechEE → [[CT 2006 - app-jtechee-lxc]] · `/jtechee/`
7. DB รวม 10 ฐาน → [[CT 2000 - core-db-mariadb]] · `192.168.92.210:3306`
8. Portal รวมลิงก์ → [[CT 2007 - app-portal-lxc]] · `/portal/`

### ⏳ แผนระยะถัดไป (ยังไม่ย้าย)
| ระบบ | VM | เสนอแนะ |
| :--- | :--- | :--- |
| Active Directory + DC | โหนด 2 (`202.44.35.28`, AD2016) | Backup แล้ว P2V/V2V ขึ้น KVM บน Proxmox |
| พัสดุ/ครุภัณฑ์ (Karupan) | VMID 30 · `192.168.92.18` | แปลงเป็น Debian/Ubuntu LXC หรือย้าย Zero Downtime |
| ระบบงานวิจัย (Vijai FTE) | VMID 36 · `192.168.92.55` | วางแผนย้าย |
| ทรัพย์สินทางปัญญา (IPR) | VMID 60 · `192.168.92.17` | วางแผนย้าย |
| File Server FS7 | โหนด 2 | ย้ายขึ้น ZFS/SAS RAID หรือสร้าง Samba/Nextcloud LXC |

## 6. เครื่องมือสำรวจที่ใช้ (Python)

สคริปต์ในโฟลเดอร์ต้นทาง: `test_esxi_passwords.py`, `probe_esxi_all.py`, `probe_banners.py`, `get_esxi_vm_states.py`, `check_esxi_vlans.py`, `scan_all_ports.py`, `scan_hosts_ports.py`, `sweep_92.py`, `run_fast_scan.py`, `run_probe_from_esxi.py`, `query_tunnels.py`, `test_port_902.py`, `test_port_8000.py`, `test_host5.py`, `test_esxi_ping.py`

## 7. แหล่งอ้างอิง

- ฉบับเต็ม: `C:\Users\Newsk\Desktop\proxmox-mrigate\ESXI_INFRASTRUCTURE_AUDIT.md`
- บันทึกอัปเดต: `LATEST_UPDATE_NOTES.md` (หมวด 8)
- รหัสผ่าน: `SYSTEM_HANDOVER_CREDENTIALS.txt` (หมวด 1.2–1.4) ⚠️ **ห้ามคัดลอกเข้า vault**
- สถานะย้ายโดยย่อ: [[ESXi Legacy Servers]] · บันทึกภาพรวม: [[Update Log]]

---
*กลับไปที่: [[Proxmox VE Migration FTE]]*
