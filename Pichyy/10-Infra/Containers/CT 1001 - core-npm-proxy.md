---
tags:
  - proxmox
  - lxc
  - nginx
  - proxy
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
status: production
ctid: 1001
---

# CT 1001 — core-npm-proxy (Nginx Proxy Manager)

> [!info] ภาพรวม
> Reverse Proxy ชั้นในสุด ทำ sub_filter rewrite พาธ/asset/cookie ทุกระบบ ให้อยู่ใต้ sub-path ของตัวเอง

## สเปก

| รายการ | รายละเอียด |
| :--- | :--- |
| CTID | `1001` (เดิม `101`) |
| Hostname | `core-npm-proxy` (เดิม `npm-proxy`) |
| OS / Runtime | Alpine + Docker **Nginx Proxy Manager** |
| สเปก | 2 vCPU / 2 GB RAM / 10 GB NVMe (`local-lvm`) |
| IP | `192.168.92.211/24` (Bridge `vmbr0`, VLAN 266, GW `.1`) |
| Web Admin Port | `81` (default: `admin@example.com` / `changeme`) |
| Config | `/opt/npm/data/nginx/custom/http.conf` |
| Auto-Start | `onboot=1` |

## หน้าที่

- รับทราฟฟิกจาก Edge Nginx บน [[Host FTEPVE]] แล้วกระจายไปยัง Web App CT แต่ละตัว
- ใช้ `sub_filter` แปลง URL สมบูรณ์, `href="/`, `src="/`, `action="/` ให้อยู่ใต้ Prefix แต่ละระบบ
- ป้องกันปัญหาเส้นทางญาติและ Protocol-Relative URLs ไม่ให้ถูกแปลงผิดเพี้ยน
- ใช้ `proxy_redirect` + `proxy_cookie_path` แก้เส้นทาง redirect และ cookies

## เส้นทางที่จัดการ

| Path | ไปที่ | Host Header |
| :--- | :--- | :--- |
| `/openhouse/` | `192.168.92.26` (CT 2001) | `openhouse.fte.kmutnb.ac.th` |
| `/edoc/` | `192.168.92.25` (CT 2002) | `edocument.fte.kmutnb.ac.th` |
| `/nc/` | `192.168.92.56` (CT 2003) | `ncteched.fte.kmutnb.ac.th` |
| `/ic/` | `192.168.92.56` (CT 2003) | `icteched.fte.kmutnb.ac.th` |
| `/aiet/` | `192.168.92.57` (CT 2004) | `aiet.fte.kmutnb.ac.th` |
| `/` (fallback) | `192.168.92.26` (CT 2001) | `openhouse.fte.kmutnb.ac.th` |

รายละเอียด sub_filter ต่อระบบดูที่ [[Gateway Nginx Setup]]

## วิธีเข้า Web Admin

```bash
# SSH Tunnel
ssh -L 8181:192.168.92.211:81 root@202.44.35.40
# แล้วเปิด http://localhost:8181
```

## วิธีอัปเดตคอนฟิก

รันสคริปต์อัตโนมัติบน Host (เขียนคอนฟิกใหม่ + push เข้า CT + test + reload):
```bash
bash /root/update_proxy_routes.sh
```
หรือทำ manual:
```bash
pct push 1001 /tmp/new_http.conf /opt/npm/data/nginx/custom/http.conf
pct exec 1001 -- docker exec npm-app-1 nginx -t
pct exec 1001 -- docker exec npm-app-1 nginx -s reload
```

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · Routing ละเอียด: [[Gateway Nginx Setup]]*
