---
tags:
  - proxmox
  - nginx
  - reverse-proxy
  - gateway
created: 2026-10-15
source: C:\Users\Newsk\Desktop\proxmox-mrigate
---

# Gateway Nginx Setup (Edge + Core Proxy)

> [!info] ภาพรวม
> Reverse Proxy 2 ชั้น — Edge Nginx บน Host รับทราฟฟิกนอก แล้วส่งต่อให้ CT 1001 (NPM) แปลงพาธ/asset ก่อนกระจายไป Web App แต่ละตัว

## สถาปัตยกรรม 2 ชั้น

```
[Client] → Edge Nginx (Host FTEPVE :80/:443)
              │ / และ /pve → 127.0.0.1:8006 (Proxmox GUI)
              │ /openhouse/ /edoc/ /nc/ /ic/ /aiet/ → CT 1001 (192.168.92.211)
              ▼
         CT 1001 core-npm-proxy (NPM + sub_filter)
              │ /openhouse/ → CT 2001 (.26)
              │ /edoc/      → CT 2002 (.25)
              │ /nc/ /ic/   → CT 2003 (.56 :80/web_nc/ และ :80/web_ic/)
              │ /aiet/      → CT 2004 (.57)
              │ /fte-web/th/ /fte-web/en/ → CT 2005 (.134)
              │ /jtechee/   → CT 2006 (.119)
              │ /portal/    → CT 2007 (.213)
              │ / (fallback)→ CT 2001 (.26)
```

## ชั้นที่ 1: Edge Nginx บน Host

- ไฟล์: `/etc/nginx/sites-available/pve_openhouse.conf`
- พอร์ต 80 (HTTP) + 443 (HTTPS) + 8006 (เฉพาะ internal)
- SSL ใช้ Self-Signed ของ PVE: `/etc/pve/local/pve-ssl.pem` + `pve-ssl.key`
- `error_page 497 =301 https://$http_host$request_uri` — ดัก HTTP ที่เข้ามาทาง HTTPS port
- `port_in_redirect off` + `absolute_redirect off` กัน redirect หลุด port

เส้นทาง Edge:
| Path | ส่งไป |
| :--- | :--- |
| `/`, `/pve/` | `https://127.0.0.1:8006` (Proxmox GUI) |
| `/openhouse/ /edoc/ /nc/ /ic/ /aiet/` | `http://192.168.92.211/<path>` (CT 1001) |
| `/fte-web/` | `http://192.168.92.211/fte-web/` (CT 1001 → CT 2005) |
| `/jtechee/` | `http://192.168.92.211/jtechee/` (CT 1001 → CT 2006) |
| `/portal/`  | `http://192.168.92.211/portal/`  (CT 1001 → CT 2007) |

> [!note] อัปเดต 15 ต.ค. 2569
> `server_name` เพิ่ม `fte.kmutnb.ac.th`, `fte-en.kmutnb.ac.th`, `jtechee.fte.kmutnb.ac.th`
> Path ใหม่: `/fte-web/th/`, `/fte-web/en/` (→ CT 2005), `/jtechee/` (→ CT 2006)
> NC/IC เปลี่ยน `proxy_pass` เป็น `:80/web_nc/` และ `:80/web_ic/` (ดู [[CT 2003 - app-nc-ic-lxc]])
> Deploy ด้วยสคริปต์ `deploy_new_routes_and_backups.py`

## ชั้นที่ 2: Core NPM ใน CT 1001

- ไฟล์: `/opt/npm/data/nginx/custom/http.conf`
- `server_name` รวมทุกโดเมนจริง + IP: `202.44.35.40 192.168.92.211 ... openhouse.fte... edocument.fte... ncteched.fte... icteched.fte... aiet.fte...`

### ตัวอย่าง sub_filter รายระบบ (ดึงจาก `update_proxy_routes.sh`)

**openhouse:**
```nginx
sub_filter 'http://openhouse.fte.kmutnb.ac.th/' '/openhouse/';
sub_filter 'href="/' 'href="/openhouse/';
sub_filter 'src="/'  'src="/openhouse/';
sub_filter 'action="/' 'action="/openhouse/';
```

**edoc (ซับซ้อนสุด — มี dashboard):**
```nginx
sub_filter '../../dashboard/' '/edoc/dashboard/';
sub_filter 'href="../' 'href="/edoc/';
sub_filter "window.location.href = 'dashboard'" "window.location.href = '/edoc/dashboard/';
proxy_cookie_path / /edoc/;
```

**nc / ic / aiet:**
```nginx
sub_filter 'https://ncteched.fte.kmutnb.ac.th/' '/nc/';   # ทั้ง http และ https
sub_filter 'href="/' 'href="/nc/';
# ปล่อย protocol-relative ไว้: sub_filter 'href="//' 'href="//'; (identity — ห้ามแปลง)
```

> [!warning] กติกา sub_filter
> - `sub_filter_once off` = แทนที่ทุก occurrence
> - `sub_filter_types` ต้องครอบคลุม `text/html text/css text/xml application/javascript text/plain`
> - ห้ามแปลง protocol-relative URLs (`//fonts.googleapis.com/...`) — ใส่ sub_filter เดียวกันซ้ำ (identity) เพื่อกันไม่ให้ `href="/` ไปจับผิด

## วิธีอัปเดตคอนฟิกทั้งระบบ

```bash
bash /root/update_proxy_routes.sh
```
สคริปต์จะเขียนคอนฟิกใหม่ทั้ง 2 ชั้น → `pct push` เข้า CT 1001 → `nginx -t` → reload ทั้งคู่

## คำสั่งตรวจสอบ

```bash
nginx -t                        # เช็ค syntax บน Host
systemctl reload nginx          # reload Edge
pct exec 1001 -- docker exec npm-app-1 nginx -t
pct exec 1001 -- docker exec npm-app-1 nginx -s reload
curl -I https://202.44.35.40/openhouse/   # ทดสอบ 200 OK
```

---
*ดูภาพรวม: [[Proxmox VE Migration FTE]] · Host: [[Host FTEPVE]] · Proxy CT: [[CT 1001 - core-npm-proxy]]*
