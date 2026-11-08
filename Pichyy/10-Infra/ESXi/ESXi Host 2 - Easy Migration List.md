---
tags:
  - esxi
  - migration
  - host2
  - easy
  - proxmox
created: 2026-10-15
source: D:\Obsidian-vault\Pichyy\10-Infra\ESXi\ESXi Host 2 - Spec & Stack.md
status: analysis
---

# ESXi Host 2 — ระบบที่ย้ายง่ายและปัญหาน้อย

> [!info] สรุปจากการวิเคราะห์
> จาก 6 ระบบเปิดใช้งานบน **ESXi Host 2 (`202.44.35.4`)** มี **2 ระบบ** ที่เหมาะสมจะย้ายขึ้น Proxmox VE เป็นอันดับแรก เพราะง่าย ปัญหาน้อย และไม่ต้องแก้ปัญหาพื้นที่ใหญ่

## 1. อันดับที่ 1 — ย้ายง่ายที่สุด

| ระบบ                                 | ขนาดข้อมูลจริง | vCPU/RAM      | VLAN/IP                 | วิธีย้าย                                       | เหตุผล                                                                                                                       |
| :----------------------------------- | :------------- | :------------ | :---------------------- | :--------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------- |
| **`Tech`** (IT Support/Utility)      | **31.9 GB**    | 1 vCPU / 4 GB | VLAN101 (ภายใน)         | **ESXi Import Tool** (`pve-esxi-import-tools`) | ระบบเครื่องมือของ IT เล็กมาก ไม่ขึ้นกับ AD/Infoblox โดยตรง แค่ import → mount disk OK. ความเสี่ยงต่ำมาก                      |
| **`Net_Monitor`** (Ubuntu 20.04 LTS) | **66.14 GB**   | 4 vCPU / 8 GB | VLAN103 `192.168.2.107` | **ESXi Import Tool** (`qemu-img/virtio`)       | Generic Linux VM, ใช้ SSH+SNMP ธรรมดา ไม่ได้เป็น DC/DNS/DHCP core จัดการง่าย. ตรวจแค่ network mapping + SNMP config หลังย้าย |

## 2. อันดับที่ 2 — ย้ายได้แต่ต้องระวังเล็กน้อย

| ระบบ | ขนาดข้อมูลจริง | vCPU/RAM | VLAN/IP | วิธีย้าย | หมายเหตุ |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`vNIOS`** (Infoblox DNS/DHCP/IPAM) | **107.0 GB** | 4 vCPU / 32 GB | VLAN103 `192.168.2.21` (4 vNICs) | Import + **คง MAC ทุก vNIC** + สำรอง Grid config | ถึงแม้เป็น Linux แต่เป็น **Core Infrastructure** (DNS/DHCP/IPAM). ต้องสำรอง Infoblox Grid backup, คง MAC ให้ครบ 4 พอร์ต และตรวจ Grid/HA/leases หลังย้าย แนะนำทำ**เฟส 2** |

## 3. ระบบที่ย้ายยากกว่าหรือมีเงื่อนไข

| ระบบ | สาเหตุ | แนวทาง |
| :--- | :--- | :--- |
| **`windows server 2019 AD35.28`** (Primary DC) | เป็น Domain Controller หลัก (FSMO) | แนะนำสร้าง DC ใหม่บน Proxmox แล้วโอน FSMO มากกว่ารวม disk raw |
| **`Windows server2016 AD`** (Secondary DC) | ต้องคงการ Replication | Sync-join/promote DC ใหม่บน Proxmox หรือ import แล้วตรวจ replication ระมัดระวัง |
| **`Windown2008 R2_FS7`** (File Server) | **ขนาดใหญ่ ~4.17 TB** แต่ Proxmox ว่าง ~3.45 TB (**ขาด ~1.06 TB**) + legacy OS (2008 R2) | ต้องแก้ปัญหาพื้นที่ก่อน: เพิ่ม storage หรือใช้ **Robocopy** (เก็บ ACLs) ไป File Server VM ใหม่บน Proxmox — **ถือเป็น Blocker** |

## 4. ข้อตกลงสถาปัตยกรรมและการตัดสินใจจากการสัมภาษณ์ (/grill-me Alignment)

จากการ Alignment ทางสถาปัตยกรรมร่วมกัน สรุปมติและแนวทางการย้าย **เฟส 1** ดังนี้:

1. **เปลี่ยนรูปแบบจาก KVM VM เป็น LXC Container:**
   * เนื่องจากทั้ง `Tech` และ `Net_Monitor` เป็น **Ubuntu Linux 64-bit** (ไม่ได้ขึ้นกับ Windows Kernel)
   * ข้อมูลจริงของ `Tech` มีเพียง ~15-30 GB และ `Net_Monitor` มีข้อมูล RootFS เพียง **14.7 GB**
   * การทำเป็น LXC จะประหยัด RAM/CPU เกือบ 100% ไม่ต้องเสีย Overhead ให้ Hypervisor และบูตเสร็จภายใน 2 วินาที

2. **การจัดหมวดและตั้งรหัสตามกฎของระบบ (Naming & ID Convention):**
   * จัดอยู่ในหมวด **Core Infrastructure (ซีรีส์ 1000)** ระดับเดียวกับ Reverse Proxy (`CT 1001 core-npm-proxy`):
     * **`CT 1003`** ➔ **`core-tech-lxc`** (ระบบ IT Technical Utility)
     * **`CT 1002`** ➔ **`core-netmon-lxc`** (ระบบ Network Monitoring)

3. **Storage & Network Placement:**
   * **Storage Pool:** วางไว้บน **`storage-raid` (SAS HDD RAID)** ซึ่งมีพื้นที่ว่าง 3.45 TB เหลือเฟือ ปลอดภัย
   * **Network & MAC:** 
     * `CT 1003` (Tech): Bridge `vmbr0`, VLAN Tag `101`, คงค่า MAC `00:0c:29:13:e4:07`
     * `CT 1002` (Net_Monitor): Bridge `vmbr0`, VLAN Tag `103`, IP `192.168.2.107`, คงค่า MAC `00:0c:29:94:2e:a5`

4. **วิธีการแปลงและลำดับการทำ (Migration Execution):**
   * ใช้วิธี **VMDK RootFS Extraction:** ดึง Virtual Disk ของ VM มาแตกไฟล์ RootFS เข้า LXC Container โดยตรง (ไม่ต้องกังวลเรื่องรหัสผ่าน root เดิม สามารถรีเซ็ตรหัสผ่านใหม่ได้ทันทีผ่าน `pct exec`)
   * **ลำดับ Pilot:** เริ่มต้นจาก **`Tech` (`CT 1003`)** ก่อนทันทีเพื่อยืนยัน Pipeline เมื่อเสร็จสมบูรณ์จึงย้าย **`Net_Monitor` (`CT 1002`)** ต่อเนื่อง


## 5. ผลการดำเนินการ Pilot เครื่องแรก: `Tech` ➔ `CT 1003 (core-tech-lxc)` (สำเร็จสมบูรณ์)

* **วันเวลาที่ดำเนินการ:** 15 ตุลาคม 2026 (เวลา 23:38 น.)
* **สถานะปัจจุบัน:** 🟢 **Online & Running (Active)**
* **ข้อมูลเชิงสถาปัตยกรรม:**
  * **CT ID:** `1003` | **Hostname:** `core-tech-lxc`
  * **OS:** Ubuntu 24.04.3 LTS (Noble Numbat)
  * **Resource Allocation:** 2 vCPU, 4096 MB RAM, Swap 1024 MB
  * **Storage Allocation:** `storage-raid:vm-1003-disk-0` ขนาด Provision 30 GB (ขนาดข้อมูลจริง **8.3 GB** ประหยัดพื้นที่จาก VM เดิม 200 GB ไปถึง 95.8%!)
  * **Network Configuration:** `eth0` บน Bridge `vmbr0` (VLAN Tag 266, IP `192.168.92.33/24`, Gateway `192.168.92.1`, MAC `00:0c:29:13:e4:07`)
  * **Active Services ภายใน:** Apache2 Web Server (Port 80), OpenSSH (Port 22), CUPS Print Server (Port 631)
  * **Systemd Health:** 0 failed units (ปิด Desktop GUI/Snap services ที่ไม่จำเป็นใน Container เรียบร้อย)
  * **Network Connectivity:** Gateway & Internet (8.8.8.8) Ping ผ่าน 100% (0% packet loss, latency ~5.5 ms)
  * **เวลาที่ใช้ในการโอนย้าย RootFS:** **169 วินาที (< 3 นาที)** ผ่านเครือข่ายความเร็วสูงตรงระหว่าง ESXi 2 และ Proxmox VE
  * **สิทธิ์ผู้ดูแลระบบ:** รหัสผ่าน root / techfte ถูกตั้งค่าเป็น `e88j8*@o`

---

## 6. ผลการดำเนินการเครื่องที่ 2: `Net_Monitor` ➔ `CT 1002 (core-netmon-lxc)` (สำเร็จสมบูรณ์)

* **วันเวลาที่ดำเนินการ:** 16 ตุลาคม 2026 (เวลา 00:41 น.)
* **สถานะปัจจุบัน:** 🟢 **Online & Running (Active)**
* **ข้อมูลเชิงสถาปัตยกรรม:**
  * **CT ID:** `1002` | **Hostname:** `core-netmon-lxc`
  * **OS:** Ubuntu 20.04.1 LTS (Focal Fossa)
  * **Resource Allocation:** 4 vCPU, 8192 MB RAM, Swap 1024 MB
  * **Storage Allocation:** `storage-raid:vm-1002-disk-0` ขนาด Provision 30 GB (ขนาดข้อมูลจริง **6.6 GB** ประหยัดพื้นที่จาก VM เดิม 128 GB ไปถึง **94.8%**!)
  * **Network Configuration:** `eth0` บน Bridge `vmbr0` (VLAN Tag 103, IP `192.168.2.107/24`, Gateway `192.168.2.1`, MAC `00:0c:29:94:2e:a5`)
  * **Active Services ภายใน:** OpenSSH Server (Port 22), System Monitoring Daemons
  * **Systemd Health:** **0 failed units** (Mask บริการ snap / open-vm-tools ที่ไม่จำเป็นใน LXC เรียบร้อย)
  * **Network Connectivity:** Gateway `192.168.2.1` Ping ผ่าน 100% (Latency 0.22 ms) และ Internet (8.8.8.8) Ping ผ่าน 100% (Latency 7.0 ms)
  * **Direct SSH Access Test:** ทดสอบผ่าน `ssh root@192.168.2.107` ผ่าน 100%
  * **สิทธิ์ผู้ดูแลระบบ:** รหัสผ่าน root / weerayuth ถูกตั้งค่ามาตรฐานเป็น `e88j8*@o`

---

## 7. ผลการดำเนินการเครื่องที่ 3: `vNIOS` ➔ `VM 1004 (core-vnios-vm)` (สำเร็จสมบูรณ์)

* **วันเวลาที่ดำเนินการ:** 16 ตุลาคม 2026 (เวลา 02:56 น.)
* **สถานะปัจจุบัน:** 🟢 **Online & Running (Active)**
* **ข้อมูลเชิงสถาปัตยกรรม:**
  * **VM ID:** `1004` | **Hostname:** `dns.fte.kmutnb.ac.th` (`core-vnios-vm`)
  * **Appliance OS:** Infoblox NIOS 8.4.4-386831 (Alpine Linux 3.6 / Granville Kernel)
  * **Resource Optimization:** 4 vCPU, **16 GB RAM** (ปรับลดจากเดิม 32 GB ประหยัด RAM ให้ Proxmox ถึง 16 GB แต่ทำงานได้ลื่นไหลสมบูรณ์ 100%)
  * **Storage Allocation:** `storage-raid:vm-1004-disk-0` ขนาด Provision 250 GB (ขนาดข้อมูลจริง **88.7 GB** บน LVM-thin)
  * **Disk Bus & Controller:** SATA Emulation (`sata0`) รองรับ AHCI บน Linux Kernel โดยตรง
  * **Network Interfaces (VirtIO):** คงค่า MAC Address เดิมครบทั้ง 4 พอร์ตบน VLAN 103:
    * `net0`: VirtIO MAC `00:0c:29:c7:77:62` (LAN1 / Mgmt)
    * `net1`: VirtIO MAC `00:0c:29:c7:77:6c` (LAN2)
    * `net2`: VirtIO MAC `00:0c:29:c7:77:76` (HA)
    * `net3`: VirtIO MAC `00:0c:29:c7:77:80` (MGMT)
  * **Hardware Fingerprint Preservation (License Safeguard):**
    * บังคับใช้ BIOS UUID เดิม: `smbios1: uuid=564d28ad-bad8-153b-5df4-25674fc77762`
    * Hardware Type: `IB-KVM`
  * **Network & Service Connectivity:**
    * Ping `192.168.2.21` ผ่าน 100% (Latency **0.37 ms**)
    * พอร์ต 22 (SSH), 80 (HTTP), 443 (HTTPS) เปิดพร้อมใช้งาน
    * **Web UI:** Infoblox Grid Manager ตอบรับ `HTTP 302 -> https://192.168.2.21/ui` พร้อมใช้งาน
  * **Rollback Safeguard:** VM 10 บน ESXi ปิดเครื่องไว้อย่างสมบูรณ์ ไฟล์ต้นฉบับปลอดภัย 100%

---

## 8. อ้างอิง

- [[ESXi Host 2 - Spec & Stack]] — วิเคราะห์สเปก/stack เต็มรูปแบบ
- [[ESXi Cluster Audit - 3 Hosts 46 VMs]] — รายชื่อครบทั้งคลัสเตอร์
- [[Proxmox VE Migration FTE]] — ภาพรวมโครงการ
- [[VM Migration Playbook]] — SOP การย้าย VM

