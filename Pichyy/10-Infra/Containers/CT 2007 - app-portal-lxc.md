---
tags:
  - proxmox
  - lxc
  - portal
  - nginx
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
ctid: 2007
---

# CT 2007 — app-portal-lxc (FTE App Portal)

> [!info] ภาพรวม
> Container สำหรับเสิร์ฟ **FTE Application Portal** — หน้าเว็บกลางรวมลิงก์ระบบงานทั้งหมดของคณะ เป็น static SPA (React build) เสิร์ฟด้วย Nginx

## สเปก

| รายการ | รายละเอียด |
| :--- | :--- |
| CTID | `2007` (สร้างใหม่ล่าสุด) |
| Hostname | `app-portal-lxc` |
| OS | Debian 12 Bookworm (Minimal) |
| Web Server | Nginx 1.22.1 (SPA Static Serving) |
| สเปก | 2 vCPU / 1 GB RAM / 10 GB SAS HDD (`storage-raid`) |
| IP | `192.168.92.213/24` (Bridge `vmbr0`, VLAN 266, GW `.1`) |
| DNS | `202.44.32.29`, `202.44.32.30` |
| Features | `nesting=1`, `onboot=1` |
| Root path | `/var/www/html/portal/` |
| URL | `https://202.44.35.40/portal/` |

## Nginx config (ใน CT 2007)

```nginx
server {
    listen 80 default_server;
    root /var/www/html;
    index index.html index.htm;

    location /portal/ {
        alias /var/www/html/portal/;
        try_files $uri $uri/ /portal/index.html;   # SPA fallback
    }

    location = / { return 301 /portal/; }
}
```

## การ Deploy (สคริปต์)

```bash
# 1) สร้าง CT + ติดตั้ง Nginx
python provision_portal_ct.py

# 2) build แล้ว deploy ตัวแอป (bundle dist เป็น tar อัตโนมัติ)
python deploy_portal.py
```

`deploy_portal.py` จะ:
1. tar โฟลเดอร์ `fte-app-portal/dist` → `portal_dist.tar.gz`
2. upload ขึ้น PVE host → `pct push` เข้า CT 2007 → `tar -xzf` ที่ `/var/www/html/portal/`
3. ตั้ง Nginx config + `nginx -t` + reload
4. เพิ่ม block `/portal/` เข้า NPM (CT 1001) และ Edge Nginx (Host) อัตโนมัติ
5. verify ด้วย `curl -skI https://202.44.35.40/portal/`

## Routing (ผ่าน [[CT 1001 - core-npm-proxy]])

```
/portal/ → proxy_pass http://192.168.92.213:80/portal/   (เพิ่ม block อัตโนมัติ)
/portal  → 301 → /portal/
```

## คำสั่งจัดการ

```bash
pct exec 2007 -- systemctl status nginx
pct exec 2007 -- ls -la /var/www/html/portal/
pct restart 2007
```

> [!note] Backup
> CT 2007 ถูกรวมใน VZDump รายวันแล้ว (เป้าหมายเป็น 8 CT) — ดู [[Backup Plan FTE]]

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · ตัวแอป: [[FTE App Portal]] · Routing: [[Gateway Nginx Setup]]*
