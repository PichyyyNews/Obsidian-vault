# UI / UX Design System & Case Studies Index

เอกสารรวบรวม Best Practices, Design Decisions และ Case Studies ทั้งหมดสำหรับโปรเจกต์ โดยอิงตามมาตรฐาน **Cloudflare Kumo UI**, **Tailwind CSS**, และ **Phosphor Icons (weight="thin")**

---

## 📂 หมวดหมู่เอกสาร (Documentation Structure)

### 1. Forms & Profile Interaction
* **[[01 - Profile & Form Optimization]]**: 
  - การจัดการ Avatar (Desktop Hover vs Mobile Touch Action Badge)
  - การตัดฟิลด์ที่ไม่จำเป็นออกด้วย Header Prominent Text + Edit Modal
  - กฎการลบข้อความภาษาไทยในวงเล็บและ Clean Typography
  - การลดขนาดฟอร์มให้เหลือเฉพาะข้อมูลสำคัญ

### 2. Notifications & Feedback
* **[[02 - Notifications & Toasts]]**: 
  - การเปลี่ยนจาก Inline Banner ที่ดัน Layout มาเป็น Floating Toast Notification
  - Responsive Behavior (ขวาล่างสำหรับ Desktop, ลอยบนสุดสำหรับ Mobile)
  - Auto-dismiss และสไตล์ Frosted Glass / Backdrop Blur

### 3. Modals & Safety Guidelines
* **[[03 - Modals & Destructive Actions]]**: 
  - กฎการทำ Confirmation Modal ก่อนลบข้อมูลใดๆ (ห้ามลบทันที)
  - รูปแบบปุ่ม Destructive vs Secondary
  - Backdrop Blur และ Focus Trap

### 4. Layout, Spacing & Visual Balance
* **[[04 - Layout, Spacing & Visual Hierarchy]]**: 
  - การชดเชยระดับสายตา (Visual Balance Compensation) เช่น การยกการ์ดขึ้น 10% (`-translate-y-[10vh]`)
  - การจัดการระยะบรรทัด (Leading & Line-height) และ Spacing ไม่ให้ข้อความเกาะกันเป็นก้อน
  - Single Cohesive Card vs Split Cards

### 5. Media & Storage Integrations
* **[[05 - Media, MinIO & Branding]]**: 
  - การตั้งค่า MinIO S3 Object Storage (Port 9000 API / 9001 Console)
  - การกำหนด Anonymous Download Policy บน Bucket เพื่อให้แสดงรูปโปรไฟล์ได้
  - การตั้งค่า App Icon, Favicon และ Logo ใน Next.js App Router

---

## 📌 กฎเหล็กที่ต้องจำ (Quick Golden Rules)
1. **No Thai Parentheses in Labels**: ใช้ชื่อภาษาอังกฤษสากลที่กระชับ ไม่ใส่วงเล็บซ้ำซ้อน
2. **Generous White Space**: อย่าให้ Element เบียดกัน เพิ่ม Breathing Room ด้วย `space-y-6`, `gap-6`
3. **Desktop Hover Must Have Mobile Alternative**: ทุกฟังก์ชันที่ใช้เมาส์ Hover ต้องมีปุ่มหรือทางเลือกให้จอมือถือแตะได้เสมอ
4. **Toast Over Banner**: แจ้งเตือนสถานะด้วย Toast เสมอ เพื่อไม่ให้ Layout ในหน้าจอกระตุก
5. **Confirm Before Destroy**: ทุก Action การลบ ต้องผ่าน Confirmation Modal

---

*กลับไปที่: [[00-Home]]*
