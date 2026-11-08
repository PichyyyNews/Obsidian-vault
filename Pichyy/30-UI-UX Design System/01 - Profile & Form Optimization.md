# Kumo UI & Modern Dashboard Design System - Lessons Learned

> **Project**: Project Tracker (`d:\projecttracker`)  
> **Core Stack**: Next.js 16 (App Router, Turbopack) + TypeScript + Tailwind CSS v4 + Cloudflare Kumo UI (`@cloudflare/kumo`) + Phosphor Icons (`@phosphor-icons/react` weight="thin") + PostgreSQL + MinIO Object Storage.

---

## 1. ปรัชญาการออกแบบหลัก (Design Philosophy & Core Rules)

1. **Clean Typography (ลดข้อความที่รกและซ้ำซ้อน)**
   - ในหน้าของผู้ใช้ (User-facing Pages) ต้องหลีกเลี่ยง **Typography และ Micro-copy ที่มากเกินไป**
   - **ตัดวงเล็บคำอธิบายภาษาไทยซ้ำซ้อนออกทั้งหมด**: เช่น `Full name (ชื่อ-สกุล)`, `Phone number (เบอร์โทรศัพท์)` ให้ใช้คำสากลที่กระชับเพียงคำเดียว เช่น `Full name`, `Phone`, `Student ID`
   - **ตัดข้อความอธิบายเชิงเทคนิคที่ผู้ใช้ไม่จำเป็นต้องรู้**: เช่น *"Upload directly to MinIO Object Storage (PNG, JPG, WebP)"* หรือ *"Dynamic options managed directly by college administrator"* ให้ใช้แค่ปุ่ม Action ที่สื่อความหมายตรงไปตรงมา เช่น `Upload photo`

2. **White Space & Visual Hierarchy (ให้ความสำคัญกับช่องไฟ)**
   - เว้นระยะห่างให้หน้าจอหายใจได้ (Breathing Room) ด้วย `space-y-6` ถึง `space-y-8` หรือ `py-10` ถึง `py-12`
   - ไม่ควรมีข้อความหรือกล่องสรุปข้อมูลที่ซ้ำกับข้อมูลที่ผู้ใช้เพิ่งกรอกไป (เช่น กล่อง Current selection summary ใต้ Dropdown ตัดออกทันที)
   - ขยับตำแหน่งเนื้อหาให้อยู่ในระดับสายตาที่เป็นธรรมชาติ: กล่อง Empty State กลางจอ ให้ชดเชยเยื้องขึ้นด้านบน ~10% (`-translate-y-[10vh]`) เพื่อไม่ให้จมอยู่กลางจอจนดูลอย

3. **Single Cohesive Card Layout**
   - รวบรวมฟอร์มและการตั้งค่าของหน้านั้นๆ ให้อยู่ในการ์ดหลักแผ่นเดียว (`LayerCard` พร้อม `ring ring-kumo-line rounded-xl shadow-sm`)
   - ไม่แยกกล่องเป็นสองท่อนหรือกระจายองค์ประกอบลอยๆ เว้นแต่จะเป็นการแบ่ง Section ใหญ่คนละบริบทอย่างแท้จริง

---

## 2. การจัดการ Avatar & Profile Interactions

### ปัญหาที่พบและวิธีแก้ (Problem & Solution)
* **ปัญหาเดิม**: มีปุ่ม `Upload photo` และ `Remove` แยกอยู่ต่างหาก พร้อมฟิลด์กรอกชื่อและอีเมลในฟอร์ม ทำให้หน้าจอเต็มไปด้วยช่อง Input
* **ทางแก้ใหม่**:
  1. **Direct Avatar Click & Hover Overlay**:
     - ให้คลิกที่ตัวรูปโปรไฟล์โดยตรงเพื่อเปิด File Selector
     - บนเดสก์ท็อป: เมื่อ Hover เหนือรูปภาพ จะมี Dark Backdrop โปร่งแสงพร้อมไอคอนกล้อง (`Camera`) และถังขยะ (`Trash`) สำหรับลบรูป
     - บนจอมือถือ (Mobile / Touch Screen): จอมือถือไม่มีเมาส์สำหรับ Hover จึงต้องมีปุ่มกลมเล็กๆ (Action Badge) ติดอยู่ที่มุมขวาล่างของ Avatar เสมอ
  2. **Prominent Header Name with Edit Modal**:
     - ดึง **ชื่อ-นามสกุล (ขนาด `text-xl font-semibold`)** และ **Email** มาวางเคียงข้าง Avatar ที่ส่วนหัวของการ์ด
     - เมื่อ Hover ที่ชื่อ จะปรากฏไอคอนดินสอ (`PencilSimple`) เมื่อคลิกจะเปิด **Modal แก้ไขชื่อ**
     - วิธีนี้ช่วยตัดช่อง Input ในฟอร์มออกไปได้ทันที 2 ช่อง ทำให้ฟอร์มเหลือเฉพาะข้อมูลสำคัญเพียง 4 ฟิลด์ (Student ID, Phone, Department, Class & Group)

---

## 3. การแจ้งเตือน (Notifications): จาก Inline Banner สู่ Responsive Toast

### กฎสำคัญ:
> **ห้ามวางแถบแจ้งเตือนขนาดใหญ่ (Inline Alert Banner) บนหัวฟอร์ม** เพราะจะดัน Layout ให้กระตุก ขยับตำแหน่ง และทำให้หน้าจอดูรก

* **ทางแก้**: ทำเป็น **Floating Toast Notification** แยก Layer ออกมาจากเนื้อหา (`fixed z-50`):
  * **Desktop (หน้าจอใหญ่)**: ลอยอยู่ที่มุมขวาล่าง (`sm:bottom-6 sm:right-6`) ไม่บดบังปุ่มหรือสายตา
  * **Mobile (หน้าจอมือถือ)**: ลอยอยู่ที่ด้านบนสุดของจอ (`top-4 left-4 right-4`) ขนาดกะทัดรัด
  * **Styling**: ใช้ขอบโปร่งแสงและกระจกฝ้า (`bg-green-500/10 border-green-500/30 backdrop-blur-md shadow-lg rounded-lg text-xs`) พร้อมไอคอน Phosphor และปุ่มปิด ($\times$)

---

## 4. Modal ยืนยันก่อนทำลายข้อมูล (Destructive Action Confirmation)

* ทุกการลบใดๆ ที่ทำให้ข้อมูลหาย เช่น **การลบภาพโปรไฟล์ (Remove Photo)** หรือการลบข้อมูลระบบ:
  * **ต้องมี Confirmation Modal เสมอ**
  * ห้ามลบข้อมูลทิ้งทันทีเมื่อกดปุ่ม
  * ใช้ไอคอนเตือนสีแดง (`Trash` ในวงกลมสีแดงอ่อน) พร้อมปุ่ม `Cancel` (Secondary) และ `Remove photo` (Destructive/แดง)

---

## 5. การตั้งค่าระบบเชื่อมต่อ (System Configurations)

### 1. MinIO Object Storage Configuration
* **พอร์ตมาตรฐาน**:
  * API / S3 Port: `9000`
  * Web Console: `9001`
* **Bucket Policy**:
  * รูปภาพโปรไฟล์ (`avatars`) ต้องเปิด **Anonymous Download Policy** เสมอ เพื่อให้เบราว์เซอร์สามารถดึงรูปมาแสดงผลบน `<img>` tag ได้โดยตรง
  ```bash
  mc alias set local http://localhost:9000 minioadmin minioadmin
  mc mb --ignore-existing local/avatars
  mc anonymous set download local/avatars
  ```

### 2. Branding (Logo & Favicon)
* นำภาพ Logo ประจำระบบวางที่ `public/logo.png`
* คัดลอกไปที่ `src/app/icon.png` และ `public/favicon.ico`
* กำหนดใน Root Layout:
  ```typescript
  export const metadata: Metadata = {
    title: "Project Tracker",
    icons: {
      icon: "/logo.png",
      shortcut: "/logo.png",
      apple: "/logo.png",
    },
  };
  ```

---

## 6. Checklist สำหรับหน้า UI ถัดไป

- [ ] **Text Budget**: ตรวจสอบว่ามีข้อความอธิบายซ้ำซ้อนหรือไม่ ตัดคำฟุ่มเฟือยออก
- [ ] **No Thai Parentheses**: ใช้คำภาษาอังกฤษมาตรฐานแทนการใส่วงเล็บภาษาไทยกำกับ
- [ ] **White Space**: ตรวจสอบว่า Spacing ระหว่างหัวข้อกับเนื้อหาไม่อึดอัด (`leading-relaxed`, `gap-5` หรือ `gap-6`)
- [ ] **Responsive Interactions**: ตรวจสอบว่าพฤติกรรมบนเดสก์ท็อป (Hover) มีทางเลือกสำหรับจอมือถือ (Touch/Icon Badge) หรือไม่
- [ ] **Notification Layer**: ใช้ Toast Notification ที่มุมขวาล่างเสมอ ไม่ใช้แถบกั้นหน้าฟอร์ม
- [ ] **Destructive Safety**: มี Confirmation Modal เสมอก่อนดำเนินการลบข้อมูลใดๆ
