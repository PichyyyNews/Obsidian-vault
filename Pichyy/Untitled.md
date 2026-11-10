Analyzed SKILL.md:L1-83
Ran inventory all pichyy.qzz.io occurrences in repo
Ran check live cloudflared config on Proxmox host · Exited with code 1
Analyzed ct.js:L350-420
Created check_cloudflared_inventory.mjs
Analyzed preview_proxy.js:L1-120
Ran inspect live cloudflared config and listening ports on Proxmox host

# แผนและเช็กลิสต์: ย้ายระบบจาก Cloudflare Tunnel (`pichyy.qzz.io`) ไปยัง VPS Reverse Proxy (`techwork.in.th`)

เอกสารสำหรับผู้ดูแลระบบและทีมพัฒนา ใช้เป็นคู่มือวางแผน แก้โค้ด และตรวจเช็กลำดับการย้ายโดเมนและทราฟฟิกทั้งหมดของวิทยาลัยจาก Cloudflare Tunnel ไปสู่ VPS ท่อส่งต่อ

---

## เป้าหมายและโครงสร้างระบบหลังย้าย

- **รูปแบบทราฟฟิกเว็บ (HTTP/HTTPS/WSS):** `ผู้ใช้ → Cloudflare DNS Only (เมฆเทา) → VPS (TLS Wildcard *.techwork.in.th + Nginx/Caddy) → Tunnel (WireGuard หรือ frp/rathole) → Proxmox Host / CT 108`
- **รูปแบบทราฟฟิก TCP ที่ไม่ใช่เว็บ (MQTT 1883, SSH):** `อุปกรณ์/ผู้ใช้ → VPS (DNAT) → Tunnel → Server/CT`
- **หลักสำคัญของระบบ IoT Portal (`CT 108`):** เว็บโครงงานนักศึกษา (`<studentId>.techwork.in.th`) ใช้ Regex Proxy แบบ Wildcard ใน [nginx.conf](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-portal/nginx.conf#L78-L99) และ [preview_proxy.js](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-backend/services/preview_proxy.js) อยู่แล้ว **จึงไม่ต้องแก้ระบบให้สร้างไฟล์ config รายคนตอนนักศึกษาขอ CT** เพียงแก้ base domain ในระบบและให้ VPS ส่ง Wildcard `*.techwork.in.th` เข้ามาที่ `10.10.10.108:3000`

---

## รายการ Domain / Subdomain และพอร์ตทั้งหมดที่ต้องย้าย (จากระบบจริง)

จากการตรวจ `/etc/cloudflared/config.yml` บน Proxmox Host (`100.125.250.85`) และซอร์สโค้ดปัจจุบัน มีบริการทั้งหมดที่ต้องย้ายดังนี้:

| กลุ่มบริการ | โดเมนเดิม (`pichyy.qzz.io`) | โดเมนใหม่ (`techwork.in.th`) | ปลายทางภายใน (Upstream) | หมายเหตุพิเศษ |
| :--- | :--- | :--- | :--- | :--- |
| **IoT Portal หลัก + API** | `iot.pichyy.qzz.io` | `iot.techwork.in.th` | `10.10.10.108:3000` | WebSocket terminal `/api/ct/terminal/*` |
| **AI Chat (Open WebUI)** | `ai.pichyy.qzz.io` | `ai.techwork.in.th` | `10.10.10.108:3000` | พึ่งพา Cookie SSO ข้ามโดเมน `.techwork.in.th` |
| **AI Gateway (LiteLLM)** | `ai-api.pichyy.qzz.io` | `ai-api.techwork.in.th` | `10.10.10.108:3000` *(หรือ `:4000`)* | ควรชี้เข้า `:3000` เพื่อได้ HTTPS redirect `/ui` |
| **MQTT WebSocket (WSS)** | `mqtt.pichyy.qzz.io` | `mqtt.techwork.in.th` | `10.10.10.108:8083` | พอร์ต `443` ผ่าน Reverse Proxy Path `/mqtt` |
| **MQTT Native TCP** | *(เดิมใช้ Tailscale/IP)* | `mqtt.techwork.in.th:1883` | `10.10.10.108:1883` | **ต้องเพิ่ม map พอร์ต `1883:1883` ใน Docker** |
| **EMQX Admin UI** | `emqx.pichyy.qzz.io` | `emqx.techwork.in.th` | `10.10.10.108:18083` | หน้าจัดการ Broker |
| **InfluxDB v2 UI & API** | `influx.pichyy.qzz.io` | `influx.techwork.in.th` | `10.10.10.108:8086` | API เขียน/อ่านข้อมูลของนักศึกษา |
| **เว็บโครงงานนักศึกษา** | `*.pichyy.qzz.io` | `*.techwork.in.th` | `10.10.10.108:3000` | Regex `<studentId>.techwork.in.th` |
| **SSH Bastion Jump Host** | `techniccom.tail59415d.ts.net:10000` | `ssh.techwork.in.th:10000` *(หรือ VPS:10000)* | `10.10.10.108:2222` | DNAT ผ่าน VPS แทน Tailscale Funnel |
| **Proxmox VE Host** | `techniccom-pve.pichyy.qzz.io` | `pve.techwork.in.th` | `https://10.10.10.1:8006` | Self-signed TLS + WebSocket noVNC |
| **Hardware & Fan Monitor** | `hardware.pichyy.qzz.io` | `hardware.techwork.in.th` | `http://10.10.10.1:8088` | มี SSE stream `/api/...` ห้าม buffer |
| **Web Server CT 100** | `aas.pichyy.qzz.io` | `aas.techwork.in.th` | `10.10.10.100:80` | เช็ก CORS/Base URL ในแอป |
| **App CT 100 (:3000)** | `techsar.pichyy.qzz.io` | `techsar.techwork.in.th` | `10.10.10.100:3000` | เช็ก Base URL ในแอป |
| **CloudPanel Admin (VM 103)** | `techniccom-cp.pichyy.qzz.io` | `cp.techwork.in.th` | `https://10.10.10.103:8443` | Self-signed TLS backend |
| **Lab Teaching Site (VM 103)** | `lab.pichyy.qzz.io` | `lab.techwork.in.th` | `10.10.10.103:80` | ไซต์การสอนของนักศึกษา |
| **Nextcloud (CT 104)** | `cloud.pichyy.qzz.io` | `cloud.techwork.in.th` | `10.10.10.104:80` | ต้องแก้ `trusted_domains` + `client_max_body_size 0` |
| **Homepage Hub (CT 105)** | `homepage.pichyy.qzz.io` | `homepage.techwork.in.th` | `10.10.10.105:3000` | อัปเดตลิงก์ในการ์ดบริการ |

---

## จุดที่ต้องแก้ในซอร์สโค้ดของระบบ IoT (ก่อนสลับ DNS)

ระบบปัจจุบันฝังชื่อ `pichyy.qzz.io` ไว้หลายไฟล์ ควรรวมศูนย์เป็นตัวแปรกลาง (เช่น `VITE_BASE_DOMAIN` และ `BASE_DOMAIN`) และรองรับช่วงเปลี่ยนผ่านทั้ง 2 โดเมน:

1. **Nginx บน CT 108 ([nginx.conf](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-portal/nginx.conf#L18-L110)):**
   - รับ `server_name` ทั้งโดเมนใหม่และโดเมนเก่าคู่กันในช่วงย้าย (เช่น `ai.techwork.in.th ai.pichyy.qzz.io`, `iot.techwork.in.th iot.pichyy.qzz.io`)
   - แก้ Regex ซับโดเมนนักศึกษาให้รองรับทั้ง `\.techwork\.in\.th$` และ `\.pichyy\.qzz\.io$`
   - เพิ่ม `set_real_ip_from` สำหรับ IP ของ VPS/Tunnel และรักษา `X-Forwarded-Proto` สำหรับ Cloudflare/VPS
2. **SSO Cookie ข้าม Subdomain ([App.tsx](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-portal/src/App.tsx#L733-L756)):**
   - แก้ Logic `cookieDomain` จากเดิมที่เช็ก `.includes('pichyy.qzz.io')` ให้ดึง Root Domain แบบ Dynamic (หรือรองรับทั้ง `.techwork.in.th` และ `.pichyy.qzz.io`) ไม่เช่นนั้น Open WebUI SSO จะล็อกอินข้ามโดเมนไม่ได้
3. **Frontend URLs และโค้ดตัวอย่าง ([App.tsx](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-portal/src/App.tsx), [AdminPanel.tsx](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-portal/src/components/AdminPanel.tsx#L823-L845)):**
   - เปลี่ยนค่า URL ใน `App.tsx` (โค้ดตัวอย่าง ESP32/ESP8266/Web JS/Python, Influx cURL, AI Starter Codes, MCP config, AI Prompts Hub, iframe Open WebUI, ซับโดเมนนักศึกษา, ปุ่ม `llms.txt`) และลิงก์คอนโซลใน `AdminPanel.tsx` ให้ดึงจากตัวแปร `BASE_DOMAIN` เดียว
4. **Backend Services และเอกสาร AI:**
   - [ai.js](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-backend/routes/ai.js#L11-L12): `PUBLIC_API_BASE` (`https://ai-api.techwork.in.th/v1`) และ `PUBLIC_CHAT_URL` (`https://ai.techwork.in.th`)
   - [ct.js](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-backend/routes/ct.js#L392): ปรับค่า `subdomain` (ปัจจุบันมี `.iot.pichyy.qzz.io`) ให้ตรงกับ Pattern จริง `${studentId}.techwork.in.th`
   - [mcp.js](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-backend/routes/mcp.js#L85-L247): เปลี่ยน MCP endpoint URL, MQTT host, Influx host
   - [ssh_bastion.js](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-backend/services/ssh_bastion.js#L96): ข้อความแนะนำ Portal URL
   - [llms.txt](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-portal/public/llms.txt) และ [robots.txt](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-portal/public/robots.txt)
5. **Docker Compose บน CT 108 ([docker-compose.iot.yml](file:///c:/Users/Newsk/Downloads/lbtechserver/docker-compose.iot.yml#L5-L20)):**
   - หากต้องการเปิด `mqtt.techwork.in.th:1883` ผ่าน DNAT ต้องเพิ่ม `"1883:1883"` ให้ service `emqx` ด้วย (ปัจจุบันเปิดเฉพาะ `8083` และ `18083`)

---

## แผนการดำเนินงานเป็นเฟส

### A. เตรียมข้อมูลและเช็กนโยบายก่อนลงมือ
- [ ] สอบถามฝ่ายเครือข่ายวิทยาลัยว่า Proxmox Host (`100.125.250.85` / วงแลนวิทยาลัย) ออกเน็ตขาออกได้ทุกพอร์ตหรือไม่ และบล็อกโปรโตคอล UDP หรือไม่
- [ ] วัดความเร็ว Upload/Download จริงจาก Proxmox Host ไปยัง VPS เป้าหมายในช่วงเวลาที่มีนักศึกษาใช้งาน
- [ ] ตรวจสเปก VPS: Static Public IPv4, Traffic quota/เดือน, เปิด `net.ipv4.ip_forward=1` ได้, และตั้ง SSH Key-only + Firewall
- [ ] สำรองไฟล์ `/etc/cloudflared/config.yml`, Credentials Tunnel (`ae270dc3-cd44-46cb-abd6-3a409f94d423.json`) และ Export DNS Records ของ `pichyy.qzz.io` จาก Cloudflare

### B. ตั้ง DNS โดเมนใหม่ (`techwork.in.th`) บน Cloudflare
- [ ] ปิด DNSSEC ที่ผู้ให้บริการจดโดเมน `techwork.in.th` (ถ้าเปิดอยู่) ก่อนเปลี่ยน Nameserver
- [ ] เพิ่มโซน `techwork.in.th` เข้า Cloudflare แผน Free และเปลี่ยน Nameserver ตามที่ Cloudflare กำหนดจนสถานะเป็น Active
- [ ] สร้าง A Record สำหรับ `techwork.in.th` และ `*.techwork.in.th` ชี้ไปยัง Static IP ของ VPS โดยปิด Proxy เป็น **DNS only (เมฆเทา)** และตั้ง TTL ต่ำ (`60-300` วินาที)
- [ ] สร้าง Cloudflare API Token แบบจำกัดสิทธิ์เฉพาะ `Zone:DNS:Edit` ของโซน `techwork.in.th` (และเพิ่มโซน `pichyy.qzz.io` ด้วย หากจะออก Cert ให้โดเมนเก่าบน VPS ด้วย)

### C. ตั้ง Tunnel เชื่อมระหว่าง Server วิทยาลัยกับ VPS
- [ ] ติดตั้ง WireGuard บน VPS (Server mode) และ Proxmox Host (Client เชื่อมออก พร้อม `PersistentKeepalive = 25`)
- [ ] ทดสอบ Ping, MTU และ Throughput ผ่านท่อ WireGuard หากพบว่า UDP ถูกบล็อกในเครือข่ายวิทยาลัย ให้สลับเป็น `frp` หรือ `rathole` บน TCP `443` (หรือแยกพอร์ต `8443`) แทน
- [ ] ตั้ง `systemd` service ให้ต่อใหม่อัตโนมัติเมื่อหลุดและทำงานอัตโนมัติหลัง Reboot ทั้งฝั่ง VPS และ Proxmox Host

### D. ตั้ง Reverse Proxy, Wildcard TLS และ DNAT บน VPS (ตั้งครั้งเดียว)
- [ ] ติดตั้ง Reverse Proxy (เช่น Nginx + Certbot/acme.sh หรือ Caddy DNS-01) และเก็บ Cloudflare API Token ในไฟล์ Environment ที่จำกัดสิทธิ์ `0600`
- [ ] ขอ Wildcard Certificate สำหรับ `techwork.in.th` + `*.techwork.in.th` ด้วยวิธี DNS-01 และทดสอบต่ออายุอัตโนมัติ (`--dry-run`)
- [ ] ตั้ง Virtual Host ส่งทราฟฟิก HTTP/HTTPS จาก VPS ผ่าน IP ในท่อ Tunnel ไปยังปลายทางแต่ละ CT/Host (`10.10.10.108:3000`, `10.10.10.100:80`, `10.10.10.104:80`, `10.10.10.1:8088` ฯลฯ) พร้อมส่ง `Host`, `X-Real-IP`, `X-Forwarded-For`, `X-Forwarded-Proto https`, รองรับ WebSocket (`Upgrade`/`Connection`), ปิด buffering สำหรับ SSE และตั้ง `client_max_body_size 0` สำหรับ Nextcloud
- [ ] ตั้ง Proxy สำหรับ Backend ที่ใช้ HTTPS Self-signed (`pve.techwork.in.th -> :8006`, `cp.techwork.in.th -> :8443`) โดยปิด verify TLS เฉพาะเส้นทางภายในท่อ Tunnel และเปิด WebSocket (จำเป็นสำหรับ noVNC/Console)
- [ ] ตั้ง iptables DNAT บน VPS เฉพาะพอร์ตที่ไม่ใช่ `80/443`: เช่น TCP `1883` (`mqtt`), TCP `10000` (`SSH Bastion -> 10.10.10.108:2222`), และพอร์ต CT อื่นตามตกลง พร้อมบันทึก Rule ให้ถาวร (`netfilter-persistent`)
- [ ] ตั้งค่า Security บน VPS: Firewall (`ufw`/`nftables`) เปิดเฉพาะ `22, 80, 443, WireGuard/Tunnel port, 1883, 10000`, ติดตั้ง `fail2ban` และจำกัด SSH เป็น Key-only

### E. ตั้งค่ารับทราฟฟิกและ Routing ฝั่ง Server วิทยาลัย
- [ ] ตั้ง Routing/Firewall บน Proxmox Host ให้ทราฟฟิกที่มาจาก IP ของ VPS ในท่อ Tunnel สามารถเข้าถึงวง `10.10.10.0/24` (`vmbr1`) ได้ และเปิด `set_real_ip_from` เฉพาะ IP ของ VPS ในท่อ Tunnel เท่านั้น
- [ ] เปิดพอร์ต `1883:1883` ใน `docker-compose.iot.yml` สำหรับ `iot-emqx`
- [ ] เพิ่ม `cloud.techwork.in.th` ใน `trusted_domains` ของ Nextcloud (CT 104) และอัปเดต `trusted_proxies` ให้เชื่อ IP ในท่อ Tunnel
- [ ] *(กรณีเตรียมขยายในอนาคต)* หากภายหลังจะเปิดให้ Panel สร้าง LXC/VM รายตัวบน Proxmox ให้เพิ่มสคริปต์ฝั่ง Backend เขียนไฟล์ proxy แยกตาม `ct<VMID>.techwork.in.th` พร้อมตรวจ `nginx -t` ก่อน Reload ทุกครั้ง (สำหรับระบบ Docker container ของนักศึกษาปัจจุบันไม่ต้องแก้ส่วนนี้)

### F. แก้ไขโค้ดระบบ IoT Portal ให้รองรับ `techwork.in.th` (โดยยังรับโดเมนเดิมคู่กัน)
- [ ] ย้ายค่าโดเมนใน `App.tsx`, `AdminPanel.tsx`, `ai.js`, `ct.js`, `mcp.js`, `ssh_bastion.js`, `llms.txt`, `robots.txt` ให้ใช้ตัวแปร `BASE_DOMAIN` เดียว (`techwork.in.th`)
- [ ] แก้ Cookie SSO ใน `App.tsx` ให้ตั้ง `domain=.techwork.in.th` เมื่อเข้าผ่าน `techwork.in.th` (และ `.pichyy.qzz.io` เมื่อเข้าผ่านโดเมนเก่า)
- [ ] อัปเดต `iot-portal/nginx.conf` ให้รับทั้งโดเมนใหม่และโดเมนเก่าคู่กันชั่วคราว ก่อนทดสอบ Build และ Deploy บน CT 108

### G. ทดสอบเต็มระบบก่อนเปิดใช้งานจริง (ยังไม่กระทบของเดิม)
- [ ] ทดสอบยิงตรงไปที่ IP ของ VPS ด้วย `curl --resolve <subdomain>.techwork.in.th:443:<VPS_IP> https://<subdomain>.techwork.in.th` ครบทุกโดเมนใน Inventory
- [ ] ทดสอบเข้า `https://iot.techwork.in.th`: ล็อกอิน, เปิดหน้าเว็บโครงงาน `<studentId>.techwork.in.th`, เปิด Web Terminal (`wss://`), เปิด Open WebUI (`ai.techwork.in.th` ว่า SSO ซิงค์ชื่อได้จริง), และยิง API ผ่าน `ai-api.techwork.in.th/v1`
- [ ] ทดสอบ WebSocket MQTT (`wss://mqtt.techwork.in.th/mqtt`) และ Native TCP MQTT (`mqtt.techwork.in.th:1883`), InfluxDB write/query, Nextcloud upload ไฟล์ใหญ่ และ SSH Bastion พอร์ต `10000`
- [ ] ตรวจ Access Log ใน CT 108 และ CT 100/104 ว่าเห็น IP จริงของผู้ใช้ ไม่ใช่ IP ของ VPS
- [ ] ทดสอบตัดการเชื่อมต่อท่อ Tunnel แล้วต่อใหม่ รวมถึง Reboot เครื่องจำลองเพื่อดูเวลาการกู้คืนอัตโนมัติ

### H. สลับใช้งานจริงแบบค่อยเป็นค่อยไปและปิดของเก่า
- [ ] ย้ายบริการที่ผลกระทบน้อยก่อน (`hardware`, `homepage`, `lab`, `aas`, `techsar`, `cloud`, `pve`, `cp`) ตามด้วยบริการหลักของนักศึกษา (`iot`, `ai`, `ai-api`, `emqx`, `influx`, `mqtt`, `*`)
- [ ] คง Cloudflare Tunnel (`pichyy.qzz.io`) ทำงานคู่ขนานไว้ `1–2` สัปดาห์ (หรือจนจบเทอมสำหรับบอร์ด ESP32/ESP8266 ที่ฝัง `mqtt.pichyy.qzz.io` ใน Firmware ไปแล้ว) และตั้ง `301 Redirect` สำหรับหน้าเว็บฝั่งคนเปิดเบราว์เซอร์ (`iot`, `ai`, `cloud`, `homepage`)
- [ ] ติดตาม Log ของ `cloudflared` บน Proxmox Host เมื่อไม่มีทราฟฟิกของบอร์ดหรือ API วิ่งเข้า `pichyy.qzz.io` แล้ว จึงหยุด service `cloudflared`, ลบ CNAME เดิม และลบ Tunnel ใน Cloudflare Dashboard
- [ ] จัดทำเอกสารสรุปส่งมอบ: IP ใน Tunnel, พอร์ตที่เปิดบน VPS/Firewall, ตำแหน่งไฟล์ Config, และขั้นตอนกู้คืนเมื่อ VPS หรือ Tunnel ล่ม

---

## ข้อที่ควรให้ผู้เชี่ยวชาญเช็กเป็นพิเศษ

- **เฟส A & C (นโยบายเครือข่ายวิทยาลัยและการทะลุ NAT):** ยืนยันว่าวิทยาลัยเปิดทางให้ WireGuard (UDP) เชื่อมออกได้หรือไม่ หากบล็อก UDP ต้องเปลี่ยนท่อเป็น `frp`/`rathole` บน TCP `443` ตั้งแต่ต้น
- **เฟส D (ความปลอดภัยของ VPS เมื่อไม่มี Cloudflare บังหน้า):** เมื่อตั้ง DNS Only (เมฆเทา) IP จริงของ VPS จะออกสู่สาธารณะและรับ DDoS/Bot Scan โดยตรง ต้องตรวจ Firewall rules, `fail2ban`, rate limit บน Nginx และไม่เปิดพอร์ตบริการภายใน (เช่น PostgreSQL `5432`, Proxmox `8006` ตรงๆ) ออกสู่เน็ต
- **เฟส D & E (Proxmox WebSocket & Nextcloud Proxy):** `pve.techwork.in.th` (`:8006`) และ `cp.techwork.in.th` (`:8443`) ใช้ HTTPS ภายในและ WebSocket สำหรับ Console ต้องตั้ง TLS upstream + timeout ให้ถูกต้อง ส่วน Nextcloud ต้องตั้ง `trusted_domains` และ `trusted_proxies` เพื่อไม่ให้ขึ้น Warning หรือ Error

---

## ความเสี่ยงที่ต้องรู้

1. **บอร์ด ESP32/ESP8266 ของนักศึกษาที่แฟลช Firmware เก่าไปแล้ว:** อุปกรณ์ที่ฮาร์ดโค้ด `mqtt.pichyy.qzz.io` หรือ `iot.pichyy.qzz.io` จะหลุดทันทีหากปิด `pichyy.qzz.io` ก่อนแก้โค้ดบนบอร์ด (ทางแก้: คง DNS เก่าหรือตั้ง Redirect/Tunnel คู่ขนานไว้ก่อนจนกว่านักศึกษาจะอัปเดตครบ)
2. **Nextcloud และ Open WebUI SSO พังชั่วคราวหากลืมแก้ Domain Config:** ถ้าไม่เพิ่มโดเมนใหม่ใน `trusted_domains` ของ Nextcloud หรือลืมแก้ `cookieDomain` ใน [App.tsx](file:///c:/Users/Newsk/Downloads/lbtechserver/iot-portal/src/App.tsx#L734) ผู้ใช้จะเข้า Nextcloud ไม่ได้และ Open WebUI จะล็อกอินเป็น `guest@lbtech.ac.th` แทนชื่อนักศึกษา
3. **Cloudflare Tunnel เดิมมี `cloudflared` รันอยู่ 2 Process:** จากการตรวจ Host พบโปรเซส `/usr/local/bin/cloudflared` และ `/usr/bin/cloudflared` รันซ้อนกันอยู่ ตอนปิดของเก่าในเฟส H ต้องตรวจเช็กให้ครบทั้ง 2 ตัว (รวมถึงใน CT อื่นถ้ามีรันไว้) เพื่อไม่ให้ค้างในระบบ

---

หากต้องการเริ่มลงมือในส่วนซอร์สโค้ด (เช่น รวมศูนย์ตัวแปร `BASE_DOMAIN` ใน Frontend/Backend และตั้ง Nginx ให้รับได้ทั้ง 2 โดเมนเตรียมไว้ก่อน) แจ้งได้เลยครับ สามารถนำไปวางเป็นไฟล์แผนหรือเช็กลิสต์ส่งต่อให้ทีม/ผู้เชี่ยวชาญตรวจได้ทันทีครับ