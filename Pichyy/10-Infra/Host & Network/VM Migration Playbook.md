---
tags:
  - proxmox
  - vmware-esxi
  - migration
  - playbook
  - sop
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
---

# VM Migration Playbook (ESXi → Proxmox)

> [!info] ภาพรวม
> SOP มาตรฐาน 2 แบบสำหรับย้าย VM จาก VMware ESXi ไป Proxmox VE — เลือกใช้ตามว่าเครื่องต้นทางปิดได้หรือไม่

## Workflow 1: Direct Raw Block Stream — เครื่องต้นทางปิดได้

ใช้กับ: [[CT 2001 - app-openhouse-lxc]] (VM 3001), [[CT 2002 - app-edoc-lxc]] (VM 3002)

1. **รวม Snapshot บน ESXi:**
   ```bash
   vmkfstools -i /vmfs/volumes/<DS>/<DIR>/<snapshot>.vmdk \
              -d thin /vmfs/volumes/<DS>/<DIR>/<target-flat>.vmdk
   ```
2. **Power Off เครื่องต้นทาง**
3. **สร้าง VM เปล่าบน Proxmox:**
   ```bash
   qm create <VMID> --name <NAME> --memory <RAM> --cores <CORE> \
     --scsihw virtio-scsi-pci --scsi0 <STORAGE>:<SIZE> \
     --net0 vmxnet3,bridge=vmbr0,tag=266,macaddr=<ORIG_MAC> \
     --boot order=scsi0 --ostype l26
   ```
4. **สตรีมตรงเข้า Block Storage ผ่าน SSH Pipe:**
   ```bash
   ssh root@<ESXi_IP> "cat /vmfs/volumes/.../<flat>.vmdk" | \
     dd of=/dev/pve/vm-<VMID>-disk-0 bs=4M conv=sparse status=progress
   ```

## Workflow 2: Zero-Downtime Live Migration — ต้นทางห้ามดับ

ใช้กับ: [[CT 2003 - app-nc-ic-lxc]] (VM 3003), [[CT 2004 - app-aiet-lxc]] (VM 3004)

1. **สร้าง Snapshot ชั่วคราวปลด Exclusive Lock** (พารามิเตอร์ `0 0` = ไม่รวม memory, ไม่ quiesce — เสร็จใน 1 วิ):
   ```bash
   vim-cmd vmsvc/snapshot.create <ESXi_VMID> "mig-snap" "migration" 0 0
   ```
2. **ดาวน์โหลดผ่าน ESXi Kernel Web Server (75–80 MB/s, เร็วกว่า SSH 2–3 เท่า):**
   ```bash
   # Base Flat (รองรับ resume ด้วย -C -)
   curl -k -C - --retry 5 "https://root:<PASS>@<ESXi_IP>/folder/<DIR>/<flat>.vmdk?dsName=<DS>" \
        -o /mnt/temp-mig/<flat>.vmdk
   # Delta Snapshot
   curl -k -C - --retry 5 "https://root:<PASS>@<ESXi_IP>/folder/<DIR>/<delta>.vmdk?dsName=<DS>" \
        -o /mnt/temp-mig/<delta>.vmdk
   ```
3. **สร้าง Descriptor 2 ไฟล์เชื่อม Parent-Child แล้ว Merge ด้วย qemu-img:**
   ```bash
   # base.vmdk (parentCID=ffffffff) + snap.vmdk (parentFileNameHint=base.vmdk)
   qemu-img convert -p -f vmdk -O raw /mnt/temp-mig/snap.vmdk /dev/vg-raid/vm-<VMID>-disk-0
   ```
4. **แก้ IP/MAC ใน Guest ก่อนบูต (ป้องกัน collision):**
   ```bash
   kpartx -av /dev/vg-raid/vm-<VMID>-disk-0
   mount /dev/mapper/...p2 /mnt/guest_root
   # แก้ /etc/netplan/ หรือ /etc/NetworkManager/system-connections/
   # ฝัง SSH key ของ Proxmox เข้า /root/.ssh/authorized_keys
   umount /mnt/guest_root && kpartx -dv /dev/vg-raid/vm-<VMID>-disk-0
   ```
5. **Start VM แล้วลบ Snapshot ชั่วคราวบน ESXi:**
   ```bash
   qm start <VMID>
   ssh root@<ESXi_IP> "vim-cmd vmsvc/snapshot.remove <ESXi_VMID> <SnapId> 0"
   ```

## Do & Don't

| ควรทำ ✅ | ห้ามทำ ❌ |
| :--- | :--- |
| ใช้ ESXi Kernel Web Server โอนข้อมูล | ดาวน์โหลด VMDK ไปพักบน Windows ผ่าน Wi-Fi |
| Snapshot ชั่วคราวเสมอถ้าต้นทางยังเปิดอยู่ | อ่าน Active Delta Disk ที่กำลังเขียน (ติด Lock / 500 Error) |
| ตั้ง IP/MAC ใหม่แยกจากของเดิม | ใช้ IP/MAC เดิม VLAN เดียวกัน (เกิด MAC Flapping) |
| ตรวจ UUID ใน `/etc/fstab` ของ Guest เสมอ | เปลี่ยน mount point เป็น `/dev/sda` มั่ว ๆ |
| ล้าง `/mnt/temp-mig` หลัง merge เสร็จ | ปล่อยไฟล์ VMDK ชั่วคราวค้างใน storage |

## สคริปต์ช่วยในคลัง (`proxmox-mrigate`)

| สคริปต์ | หน้าที่ |
| :--- | :--- |
| `migrate_aiet.py` | ดาวน์โหลด + สร้าง descriptor + merge VM 3004 อัตโนมัติ |
| `run_finalize.py` | รวม snapshot chain + ย้ายดิสก์ VM 3003 |
| `verify_snap_chain.py` | ตรวจความถูกต้อง VMDK Descriptor Chain ก่อน merge |
| `fast_stream_flat.sh` | stream base flat ผ่าน kernel web server + `dd conv=sparse` |
| `setup_aiet_network.py` | ตั้ง Netplan/NetworkManager ให้ Ubuntu 24.04 |

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · ระบบปลายทาง: [[CT 2001 - app-openhouse-lxc]] [[CT 2002 - app-edoc-lxc]] [[CT 2003 - app-nc-ic-lxc]] [[CT 2004 - app-aiet-lxc]]*
