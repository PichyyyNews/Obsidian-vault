---
title: "แบบฝึกหัด — ระบบควบคุมหลอดไฟ 4 ห้องผ่านแอปพลิเคชัน Blynk และโปรแกรมจำลอง Wokwi"
aliases:
  - ใบงาน Blynk 4 ห้อง LDR
  - แบบฝึกหัด IoT Interface Blynk
  - ระบบควบคุมหลอดไฟ 4 ห้อง Wokwi
  - 31909-2002 แบบฝึกหัดที่ 1
tags:
  - การสอน
  - แบบฝึกหัด
  - ใบงาน
  - IoT
  - blynk
  - wokwi
  - esp32
  - ปวส
  - วิทยาลัยเทคนิคลพบุรี
course_code: 31909-2002
course_title: การออกแบบส่วนต่อประสานสำหรับงาน IoT
unit: 1
week: 5
type: แบบฝึกหัด / ใบงานปฏิบัติการ
score: 20
created: 2026-11-17
updated: 2026-11-17
status: พร้อมแจกและปฏิบัติการ
---

# แบบฝึกหัดและใบงานปฏิบัติการ: ระบบควบคุมหลอดไฟ 4 ห้องผ่าน Blynk IoT และ Wokwi

> [!NOTE]
> **ข้อมูลใบงานปฏิบัติการ**
>
> | รายการ | รายละเอียด |
> | :--- | :--- |
> | รายวิชา | `31909-2002` การออกแบบส่วนต่อประสานสำหรับงาน IoT |
> | ระดับชั้น | ปวส. สาขาวิชาเทคโนโลยีคอมพิวเตอร์ |
> | สัปดาห์ / หน่วยการเรียน | สัปดาห์ที่ 5 / หน่วยที่ 1 (ส่วนต่อประสานสำหรับงาน IoT และ Blynk Platform) |
> | เวลาปฏิบัติการ | 3 ชั่วโมง (150 นาที) |
> | คะแนนเต็ม | **20 คะแนน** (เกณฑ์ประเมินทักษะปฏิบัติ 5 มิติ) และคะแนนโบนัส 3 คะแนน |
> | เครื่องมือที่ใช้ | Wokwi Simulator, Blynk.Cloud และสมาร์ตโฟน (Blynk IoT App) |
> | เนื้อหาอ้างอิง | [[บทที่ 1 - พื้นฐานส่วนต่อประสาน IoT Protocols และแอปพลิเคชัน Blynk]] |

---

## 1. จุดประสงค์เชิงปฏิบัติการ

เมื่อนักศึกษาทำแบบฝึกหัดปฏิบัติการชุดนี้สำเร็จ จะต้องสามารถ:
- [ ] ออกแบบและกำหนด Datastreams (Virtual Pins) บนแพลตฟอร์ม Blynk Cloud เพื่อใช้เป็นส่วนต่อประสานส่งผ่านข้อมูลระหว่างฮาร์ดแวร์กับผู้ใช้ได้อย่างถูกต้อง
- [ ] ออกแบบส่วนต่อประสานผู้ใช้แบบกราฟิก (GUI Dashboard) บนมือถือและเว็บบราวเซอร์ โดยจัดวางวิตเจ็ต (Switch, Slider, Gauge, LED) ได้อย่างเป็นระเบียบ ชัดเจน และใช้งานง่าย
- [ ] ต่อวงจรจำลองบน Wokwi Simulator โดยเชื่อมต่อไมโครคอนโทรลเลอร์ ESP32 ร่วมกับตัวต้านทานปรับค่าตามแสง (LDR Photoresistor) และหลอด LED แสดงผล 4 หลอดได้อย่างถูกต้องตามหลักไฟฟ้า
- [ ] เขียนโปรแกรมภาษา C++ ควบคุมการทำงานแบบ 2 โหมด ได้แก่ โหมดสั่งการด้วยมือ (Manual Mode) และโหมดอัตโนมัติตามแสง (Auto Mode)
- [ ] ปรับแต่งค่าเกณฑ์ระดับความสว่าง (Threshold) แบบ Dynamic จากสไลเดอร์บนหน้าจอมือถือ แล้วนำค่านั้นมาตัดสินใจเปิด-ปิดหลอดไฟตามสภาพแวดล้อมได้แบบ Real-Time
- [ ] ประยุกต์ใช้คำสั่ง `BlynkTimer` ในการจัดตารางเวลาอ่านเซนเซอร์โดยไม่ใช้ `delay()` เพื่อรักษาเสถียรภาพการเชื่อมต่อของระบบ IoT

---

## 2. สถานการณ์จำลองและสถาปัตยกรรมระบบ

ระบบไฟฟ้าส่องสว่างอัจฉริยะสำหรับอาคารพักอาศัย ครอบคลุมพื้นที่ 4 ห้องหลัก:
1. **ห้องนั่งเล่น (Living Room)**
2. **ห้องนอน (Bedroom)**
3. **ห้องครัว (Kitchen)**
4. **ห้องทำงาน (Study Room)**

ระบบมีเซนเซอร์ตรวจวัดความเข้มแสงแดดธรรมชาติ (LDR Sensor) ติดตั้งภายนอกอาคาร 1 ตัว เพื่อตรวจสอบระดับความสว่างในแต่ละช่วงเวลา

```mermaid
flowchart TD
    App["สมาร์ตโฟน (Blynk App)<br/>- สวิตช์สลับโหมด Manual / Auto (V0)<br/>- ปุ่มเปิด/ปิดไฟแยก 4 ห้อง (V1-V4)<br/>- เกจแสดงค่าความเข้มแสงแดด (V5)<br/>- แถบสไลเดอร์ตั้งค่า Threshold แสง (V6)"]
    <-->|"Wi-Fi Internet Connection"| ESP["บอร์ด ESP32 (จำลองบน Wokwi)"]
    
    ESP -->|"อ่านค่าแรงดันแอนะล็อก (GPIO 34)"| LDR["เซนเซอร์วัดแสง LDR"]
    ESP -->|"ขับสัญญาณดิจิทัล OUTPUT"| LEDs["หลอดไฟ LED ประจำ 4 ห้อง<br/>- ห้องนั่งเล่น (GPIO 18)<br/>- ห้องนอน (GPIO 19)<br/>- ห้องครัว (GPIO 21)<br/>- ห้องทำงาน (GPIO 22)"]
```

### ข้อกำหนดการทำงานของระบบ

1. **โหมดควบคุมเองด้วยมือ (Manual Mode):**
   * ผู้ใช้สามารถกดสวิตช์เปิดหรือปิดไฟของแต่ละห้องได้อย่างอิสระ ผ่านปุ่มกดทั้ง 4 ปุ่มบนแอปพลิเคชัน Blynk
   * ระบบจะไม่นำค่าจากเซนเซอร์ LDR มาสั่งเปิดหรือปิดไฟ
2. **โหมดเปิดปิดไฟอัตโนมัติตามแสง (Auto Mode):**
   * บอร์ด ESP32 อ่านค่าความสว่างจากเซนเซอร์ LDR ต่อเนื่อง และส่งค่าขึ้นไปแสดงผลบนเข็มเกจบนแอป
   * หาก **ค่าความเข้มแสง น้อยกว่าค่าเกณฑ์ (LDR < Threshold)** ซึ่งแสดงว่าสภาพแวดล้อมมืด ระบบจะสั่งเปิดไฟทั้ง 4 ห้องโดยอัตโนมัติ พร้อมส่งสัญญาณไฟ LED บนแอปว่าระบบไฟอัตโนมัติกำลังทำงาน
   * หาก **ค่าความเข้มแสง มากกว่าหรือเท่ากับเกณฑ์ (LDR >= Threshold)** ซึ่งแสดงว่ามีแสงสว่างเพียงพอ ระบบจะสั่งปิดไฟทั้ง 4 ห้องโดยอัตโนมัติ เพื่อประหยัดพลังงาน
3. **การตั้งค่าเกณฑ์แสงแบบไดนามิก (Dynamic Threshold Tuning):**
   * ห้ามกำหนดค่าตัวเลขเกณฑ์แสงแบบคงที่ (Hard-code) ในโค้ดโปรแกรม
   * ผู้ใช้ต้องสามารถเลื่อน Slider Widget บนแอป Blynk เพื่อปรับเปลี่ยนค่าเกณฑ์ความไวแสงได้ตามต้องการในทันที

---

## 3. แผนผังการต่อวงจรใน Wokwi

> [!NOTE]
> **ตารางการต่อขาฮาร์ดแวร์ (Hardware Pin Assignment)**
>
> | อุปกรณ์ | ขาบนอุปกรณ์ | ขาบน ESP32 | หน้าที่การทำงาน |
> | :--- | :--- | :--- | :--- |
> | **LED 1 (สีเขียว)** | ขา Anode (+) ผ่าน R 220 โอห์ม / Cathode ลง GND | **GPIO 18** | ตัวแทนหลอดไฟ: ห้องนั่งเล่น |
> | **LED 2 (สีแดง)** | ขา Anode (+) ผ่าน R 220 โอห์ม / Cathode ลง GND | **GPIO 19** | ตัวแทนหลอดไฟ: ห้องนอน |
> | **LED 3 (สีเหลือง)** | ขา Anode (+) ผ่าน R 220 โอห์ม / Cathode ลง GND | **GPIO 21** | ตัวแทนหลอดไฟ: ห้องครัว |
> | **LED 4 (สีน้ำเงิน)** | ขา Anode (+) ผ่าน R 220 โอห์ม / Cathode ลง GND | **GPIO 22** | ตัวแทนหลอดไฟ: ห้องทำงาน |
> | **LDR Module** | ขา VCC (ต่อเข้า 3.3V), GND (ลง GND)<br>ขา Signal / AO (สัญญาณแอนะล็อก) | **GPIO 34** (ADC1_CH6) | เซนเซอร์ตรวจวัดความเข้มแสง (0–4095) |

> [!TIP]
> **โครงสร้างไฟล์ `diagram.json` ของ Wokwi (สำหรับตรวจสอบการต่อสาย)**
> ```json
> {
>   "version": 1,
>   "author": "Pichyy IoT Course",
>   "editor": "wokwi",
>   "parts": [
>     { "type": "board-esp32-devkit-c-v4", "id": "esp", "top": 0, "left": 0, "attrs": {} },
>     { "type": "wokwi-led", "id": "led1", "top": -100, "left": 100, "attrs": { "color": "green" } },
>     { "type": "wokwi-led", "id": "led2", "top": -50, "left": 100, "attrs": { "color": "red" } },
>     { "type": "wokwi-led", "id": "led3", "top": 0, "left": 100, "attrs": { "color": "yellow" } },
>     { "type": "wokwi-led", "id": "led4", "top": 50, "left": 100, "attrs": { "color": "blue" } },
>     { "type": "wokwi-resistor", "id": "r1", "top": -90, "left": 160, "attrs": { "value": "220" } },
>     { "type": "wokwi-resistor", "id": "r2", "top": -40, "left": 160, "attrs": { "value": "220" } },
>     { "type": "wokwi-resistor", "id": "r3", "top": 10, "left": 160, "attrs": { "value": "220" } },
>     { "type": "wokwi-resistor", "id": "r4", "top": 60, "left": 160, "attrs": { "value": "220" } },
>     { "type": "wokwi-photoresistor-sensor", "id": "ldr", "top": -100, "left": -120, "attrs": {} }
>   ],
>   "connections": [
>     [ "esp:TX", "$serialMonitor:RX", "", [] ],
>     [ "esp:RX", "$serialMonitor:TX", "", [] ],
>     [ "esp:18", "led1:A", "green", [ "v0" ] ],
>     [ "led1:C", "r1:1", "black", [ "v0" ] ],
>     [ "r1:2", "esp:GND.1", "black", [ "v0" ] ],
>     [ "esp:19", "led2:A", "red", [ "v0" ] ],
>     [ "led2:C", "r2:1", "black", [ "v0" ] ],
>     [ "r2:2", "esp:GND.1", "black", [ "v0" ] ],
>     [ "esp:21", "led3:A", "gold", [ "v0" ] ],
>     [ "led3:C", "r3:1", "black", [ "v0" ] ],
>     [ "r3:2", "esp:GND.1", "black", [ "v0" ] ],
>     [ "esp:22", "led4:A", "blue", [ "v0" ] ],
>     [ "led4:C", "r4:1", "black", [ "v0" ] ],
>     [ "r4:2", "esp:GND.1", "black", [ "v0" ] ],
>     [ "esp:3V3", "ldr:VCC", "red", [ "v0" ] ],
>     [ "esp:GND.2", "ldr:GND", "black", [ "v0" ] ],
>     [ "esp:34", "ldr:AO", "orange", [ "v0" ] ]
>   ]
> }
> ```

---

## 4. ข้อกำหนดการออกแบบส่วนต่อประสาน (Blynk Datastreams)

นักศึกษาต้องสร้าง Template ใหม่บน **Blynk.Cloud** ชื่อ `Smart Home Lighting 4-Room` โดยกำหนดตาราง Datastreams ตามตารางนี้:

> [!IMPORTANT]
> **ตารางกำหนดข้อตกลง Interface (Virtual Pin Datastream Map)**
>
> | Virtual Pin | ชื่อ Datastream | ชนิดข้อมูล | ช่วงค่า | หน้าที่และความหมาย | ชนิด Widget บนหน้าจอ |
> | :---: | :--- | :---: | :---: | :--- | :--- |
> | `V0` | `System_Mode` | Integer | `0` ถึง `1` | `0` = โหมด Manual, `1` = โหมด Auto | **Segmented Switch** หรือ **Switch** |
> | `V1` | `Light_Room1` | Integer | `0` ถึง `1` | ควบคุมไฟห้องนั่งเล่น (`0`=ปิด, `1`=เปิด) | **Button** (โหมด Switch) |
> | `V2` | `Light_Room2` | Integer | `0` ถึง `1` | ควบคุมไฟห้องนอน (`0`=ปิด, `1`=เปิด) | **Button** (โหมด Switch) |
> | `V3` | `Light_Room3` | Integer | `0` ถึง `1` | ควบคุมไฟห้องครัว (`0`=ปิด, `1`=เปิด) | **Button** (โหมด Switch) |
> | `V4` | `Light_Room4` | Integer | `0` ถึง `1` | ควบคุมไฟห้องทำงาน (`0`=ปิด, `1`=เปิด) | **Button** (โหมด Switch) |
> | `V5` | `LDR_Light_Value` | Integer | `0` ถึง `4095` | ค่าความเข้มแสงสดจากเซนเซอร์ LDR | **Gauge** หรือ **Value Display** |
> | `V6` | `Light_Threshold` | Integer | `0` ถึง `4095` | เกณฑ์ความสว่างสำหรับสั่งไฟติดอัตโนมัติ | **Slider** (ปรับค่าได้ 0-4095) |
> | `V7` | `Auto_Light_Status` | Integer | `0` ถึง `1` | สถานะแจ้งเตือนว่าขณะนี้ระบบ Auto เปิดไฟอยู่ | **LED Widget** บนหน้าจอ |

---

## 5. ผังงานการทำงานของระบบ (System Logic Flowchart)

```mermaid
flowchart TD
    Start["เริ่มต้นระบบ (ESP32 Boot)"] --> InitHardware["กำหนด pinMode GPIO 18, 19, 21, 22 เป็น OUTPUT<br/>และ GPIO 34 เป็น INPUT"]
    InitHardware --> ConnectBlynk["เชื่อมต่อ Wi-Fi และเข้าสู่ Blynk.Cloud"]
    ConnectBlynk --> SetTimer["ตั้งเวลา BlynkTimer ให้เรียก checkSensors ทุก 1 วินาที"]
    
    SetTimer --> Loop["วนรอบ loop() -> เรียก Blynk.run() และ timer.run()"]
    
    Loop -.->|"ครบกำหนดทุก 1 วินาที (Timer Trigger)"| ReadLDR["อ่านค่าความเข้มแสงจาก GPIO 34 (analogRead)"]
    ReadLDR --> SendV5["ส่งค่าแสงสดขึ้น Blynk ด้วย Blynk.virtualWrite(V5, ldrValue)"]
    
    SendV5 --> CheckMode{"โหมดปัจจุบันคือ Auto?<br/>(isAutoMode == true)"}
    
    CheckMode -- "ไม่ใช่ (Manual Mode)" --> IdleMode["ไม่ควบคุมไฟอัตโนมัติ<br/>ไฟติด/ดับตามคำสั่งปุ่ม V1-V4 บนแอป"]
    CheckMode -- "ใช่ (Auto Mode)" --> CompareThreshold{"ldrValue < lightThreshold ?<br/>(แสงมืดกว่าเกณฑ์ที่ตั้งไว้)"}
    
    CompareThreshold -- "ใช่ (สภาพแสงมืด)" --> TurnOnAll["สั่งเปิดไฟทั้ง 4 ห้อง (HIGH)<br/>อัปเดตสวิตช์ V1-V4 เป็น 1 บนแอป<br/>เปิดไฟ LED สถานะ V7 เป็น 1 บนแอป"]
    CompareThreshold -- "ไม่ใช่ (สภาพแสงสว่าง)" --> TurnOffAll["สั่งปิดไฟทั้ง 4 ห้อง (LOW)<br/>อัปเดตสวิตช์ V1-V4 เป็น 0 บนแอป<br/>ปิดไฟ LED สถานะ V7 เป็น 0 บนแอป"]
    
    TurnOnAll --> Loop
    TurnOffAll --> Loop
    IdleMode --> Loop
```

---

## 6. ซอร์สโค้ดโปรแกรมต้นแบบ

```cpp
/*
 * =========================================================================
 * รายวิชา: 31909-2002 การออกแบบส่วนต่อประสานสำหรับงาน IoT
 * ใบงานปฏิบัติการ: ระบบควบคุมหลอดไฟ 4 ห้องผ่าน Blynk และ Wokwi
 * สาขาวิชาเทคโนโลยีคอมพิวเตอร์ วิทยาลัยเทคนิคลพบุรี
 * =========================================================================
 */

// 1. ใส่ข้อมูล Template และ Auth Token จากบัญชี Blynk.Cloud ของนักศึกษา
#define BLYNK_TEMPLATE_ID   "TMPLxxxxxx"
#define BLYNK_TEMPLATE_NAME "Smart Home Lighting 4-Room"
#define BLYNK_AUTH_TOKEN    "Your_Auth_Token_Here"

// 2. เรียกใช้งานไลบรารีที่จำเป็น
#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>

// 3. กำหนดข้อมูลการเชื่อมต่อ Wi-Fi สำหรับโปรแกรมจำลอง Wokwi
char ssid[] = "Wokwi-GUEST";
char pass[] = "";

// 4. กำหนดหมายเลขขาฮาร์ดแวร์ (GPIO Assignment)
const int PIN_LED_ROOM1 = 18;  // ห้องนั่งเล่น (เขียว)
const int PIN_LED_ROOM2 = 19;  // ห้องนอน (แดง)
const int PIN_LED_ROOM3 = 21;  // ห้องครัว (เหลือง)
const int PIN_LED_ROOM4 = 22;  // ห้องทำงาน (น้ำเงิน)
const int PIN_LDR_SENSOR = 34; // เซนเซอร์วัดแสง LDR (Analog Input)

// 5. ตัวแปรเก็บสถานะของระบบ
bool isAutoMode = false;       // สถานะโหมด: false = Manual, true = Auto
int lightThreshold = 1500;     // ค่าเกณฑ์ความสว่างเริ่มต้น (ปรับได้จาก Slider V6)
int currentLightValue = 0;     // ตัวแปรเก็บค่าแสงสดที่อ่านได้จากเซนเซอร์

// 6. ประกาศอ็อบเจกต์ตัวจับเวลา (ป้องกันการใช้ delay)
BlynkTimer timer;

// =========================================================================
// ฟังก์ชันจัดการเหตุการณ์เมื่อมีการสั่งงานจาก Blynk App (Virtual Pins Callback)
// =========================================================================

// V0: สวิตช์สลับโหมด Manual (0) / Auto (1)
BLYNK_WRITE(V0) {
    int modeVal = param.asInt();
    isAutoMode = (modeVal == 1);
    
    Serial.print("[System Mode Changed]: ");
    if (isAutoMode) {
        Serial.println("โหมดอัตโนมัติ (AUTO MODE) เปิดทำงาน");
    } else {
        Serial.println("โหมดควบคุมด้วยมือ (MANUAL MODE) เปิดทำงาน");
        Blynk.virtualWrite(V7, 0); // ดับไฟเตือน Auto Status
    }
}

// V1: ควบคุมไฟห้องที่ 1 (เฉพาะเมื่ออยู่ในโหมด Manual)
BLYNK_WRITE(V1) {
    if (!isAutoMode) {
        int state = param.asInt();
        digitalWrite(PIN_LED_ROOM1, state);
        Serial.printf("[Manual] ห้องนั่งเล่น: %s\n", state ? "เปิดไฟ" : "ปิดไฟ");
    }
}

// V2: ควบคุมไฟห้องที่ 2 (เฉพาะเมื่ออยู่ในโหมด Manual)
BLYNK_WRITE(V2) {
    if (!isAutoMode) {
        int state = param.asInt();
        digitalWrite(PIN_LED_ROOM2, state);
        Serial.printf("[Manual] ห้องนอน: %s\n", state ? "เปิดไฟ" : "ปิดไฟ");
    }
}

// V3: ควบคุมไฟห้องที่ 3 (เฉพาะเมื่ออยู่ในโหมด Manual)
BLYNK_WRITE(V3) {
    if (!isAutoMode) {
        int state = param.asInt();
        digitalWrite(PIN_LED_ROOM3, state);
        Serial.printf("[Manual] ห้องครัว: %s\n", state ? "เปิดไฟ" : "ปิดไฟ");
    }
}

// V4: ควบคุมไฟห้องที่ 4 (เฉพาะเมื่ออยู่ในโหมด Manual)
BLYNK_WRITE(V4) {
    if (!isAutoMode) {
        int state = param.asInt();
        digitalWrite(PIN_LED_ROOM4, state);
        Serial.printf("[Manual] ห้องทำงาน: %s\n", state ? "เปิดไฟ" : "ปิดไฟ");
    }
}

// V6: สไลเดอร์ปรับค่าเกณฑ์แสง (Threshold) แบบ Dynamic
BLYNK_WRITE(V6) {
    lightThreshold = param.asInt();
    Serial.printf("[Threshold Updated] ค่าเกณฑ์แสงใหม่: %d\n", lightThreshold);
}

// =========================================================================
// ฟังก์ชันอ่านเซนเซอร์และตัดสินใจตรรกะอัตโนมัติ (รันทุกๆ 1 วินาที)
// =========================================================================
void checkLightAndProcessLogic() {
    // 1. อ่านค่าความสว่างจากเซนเซอร์ LDR (0 - 4095 บน ESP32 ADC)
    currentLightValue = analogRead(PIN_LDR_SENSOR);
    
    // 2. ส่งค่าแสงสดขึ้นไปแสดงบนเกจหน้าจอแอป (Virtual Pin V5)
    Blynk.virtualWrite(V5, currentLightValue);

    // 3. ตรวจสอบเงื่อนไขหากกำลังทำงานในโหมด Auto
    if (isAutoMode) {
        // หากค่าแสงน้อยกว่า Threshold หมายความว่าสภาพแวดล้อมมืด -> สั่งเปิดไฟ
        if (currentLightValue < lightThreshold) {
            digitalWrite(PIN_LED_ROOM1, HIGH);
            digitalWrite(PIN_LED_ROOM2, HIGH);
            digitalWrite(PIN_LED_ROOM3, HIGH);
            digitalWrite(PIN_LED_ROOM4, HIGH);

            // ซิงโครไนซ์อัปเดตสวิตช์บนหน้าจอแอปให้แสดงว่าเปิดไฟอยู่
            Blynk.virtualWrite(V1, 1);
            Blynk.virtualWrite(V2, 1);
            Blynk.virtualWrite(V3, 1);
            Blynk.virtualWrite(V4, 1);
            
            // เปิดไฟ LED สถานะ Auto บนหน้าจอแอป
            Blynk.virtualWrite(V7, 255);
            Serial.printf("[Auto Active - DARK] แสง (%d) < เกณฑ์ (%d) -> เปิดไฟทั้ง 4 ห้อง\n", 
                          currentLightValue, lightThreshold);
        } else {
            digitalWrite(PIN_LED_ROOM1, LOW);
            digitalWrite(PIN_LED_ROOM2, LOW);
            digitalWrite(PIN_LED_ROOM3, LOW);
            digitalWrite(PIN_LED_ROOM4, LOW);

            // ซิงโครไนซ์อัปเดตสวิตช์บนหน้าจอแอปให้แสดงว่าปิดไฟ
            Blynk.virtualWrite(V1, 0);
            Blynk.virtualWrite(V2, 0);
            Blynk.virtualWrite(V3, 0);
            Blynk.virtualWrite(V4, 0);

            // ดับไฟ LED สถานะ Auto บนหน้าจอแอป
            Blynk.virtualWrite(V7, 0);
            Serial.printf("[Auto Active - BRIGHT] แสง (%d) >= เกณฑ์ (%d) -> ปิดไฟเพื่อประหยัดพลังงาน\n", 
                          currentLightValue, lightThreshold);
        }
    }
}

// =========================================================================
// ฟังก์ชันเริ่มต้นระบบ (Setup)
// =========================================================================
void setup() {
    Serial.begin(115200);
    Serial.println("\n--- กำลังเริ่มระบบ Smart Home Lighting System ---");

    // กำหนดโหมดการทำงานของขาขาออก (LEDs)
    pinMode(PIN_LED_ROOM1, OUTPUT);
    pinMode(PIN_LED_ROOM2, OUTPUT);
    pinMode(PIN_LED_ROOM3, OUTPUT);
    pinMode(PIN_LED_ROOM4, OUTPUT);

    // กำหนดขาเข้าสำหรับเซนเซอร์ LDR
    pinMode(PIN_LDR_SENSOR, INPUT);

    // เริ่มต้นให้ไฟทุกดวงดับ
    digitalWrite(PIN_LED_ROOM1, LOW);
    digitalWrite(PIN_LED_ROOM2, LOW);
    digitalWrite(PIN_LED_ROOM3, LOW);
    digitalWrite(PIN_LED_ROOM4, LOW);

    // เชื่อมต่อเข้าสู่ระบบ Blynk ผ่าน Wi-Fi
    Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

    // กำหนดให้เรียกฟังก์ชัน checkLightAndProcessLogic ทุก 1000 มิลลิวินาที (1 วินาที)
    timer.setInterval(1000L, checkLightAndProcessLogic);
    
    // กำหนดค่าเริ่มต้นให้กับ Slider บนแอป
    Blynk.virtualWrite(V6, lightThreshold);
    Blynk.virtualWrite(V0, 0); // เริ่มต้นที่โหมด Manual
}

// =========================================================================
// ฟังก์ชันลูปหลัก (Main Loop)
// =========================================================================
void loop() {
    Blynk.run();  // ดูแลรักษาการเชื่อมต่อและรับส่งคำสั่งกับ Blynk Cloud
    timer.run();  // สั่งให้ตัวจับเวลาทำงานตามรอบเวลาที่กำหนด
}
```

---

## 7. ขั้นตอนการลงมือปฏิบัติ

```mermaid
flowchart TD
    S1["ขั้นที่ 1: สร้าง Blynk Template และกำหนด 8 Datastreams"]
    --> S2["ขั้นที่ 2: ออกแบบ Dashboard (Web Console และ Mobile App)"]
    --> S3["ขั้นที่ 3: เปิด Wokwi ต่อวงจร ESP32, LED 4 หลอด และ LDR"]
    --> S4["ขั้นที่ 4: ติดตั้งไลบรารี Blynk ใน Wokwi และนำ Auth Token ใส่ในโค้ด"]
    --> S5["ขั้นที่ 5: เริ่มการจำลอง (Start Simulation) และทดสอบจากสมาร์ตโฟน"]
```

### ขั้นที่ 1: เตรียมการบน Blynk.Cloud
1. เข้าเว็บไซต์ blynk.cloud ล็อกอินเข้าสู่ระบบ
2. ไปที่เมนู **Developer Zone -> Templates** กดปุ่ม **+ New Template**
   * Name: `Smart Home Lighting 4-Room`
   * Hardware: `ESP32`
   * Connection Type: `WiFi`
3. ไปที่แท็บ **Datastreams** แล้วกด **+ New Datastream -> Virtual Pin** สร้างครบทั้ง 8 ขา (`V0` ถึง `V7`) ตามตารางในหัวข้อที่ 4

### ขั้นที่ 2: วางหน้าจอควบคุม (Dashboard UI)
1. ไปที่แท็บ **Web Dashboard** ใน Template ลากวางวิตเจ็ต:
   * วิตเจ็ต **Switch** สำหรับ `V0` (ตั้งชื่อ: "Manual / Auto Mode")
   * วิตเจ็ต **Switch** 4 ตัวสำหรับ `V1`, `V2`, `V3`, `V4` (ระบุชื่อห้องแต่ละห้อง)
   * วิตเจ็ต **Gauge** สำหรับ `V5` (ตั้งค่า Min: 0, Max: 4095, ชื่อ: "ความสว่างแสงแดด")
   * วิตเจ็ต **Slider** สำหรับ `V6` (ตั้งค่า Min: 0, Max: 4095, ชื่อ: "ตั้งเกณฑ์แสงอัตโนมัติ")
   * วิตเจ็ต **LED** สำหรับ `V7` (ชื่อ: "สถานะเปิดไฟอัตโนมัติ")
2. เปิดแอป **Blynk IoT** บนสมาร์ตโฟน เข้า Template เดียวกัน แล้วจัดวาง Mobile Dashboard ให้สวยงาม

### ขั้นที่ 3: จำลองการทำงานบน Wokwi
1. เข้าเว็บไซต์ wokwi.com เลือกบอร์ด **ESP32**
2. ในแท็บ **Library Manager** ค้นหาและกดติดตั้งไลบรารี **`Blynk`**
3. ต่อสายวงจรหลอด LED 4 สี พร้อมตัวต้านทาน 220 โอห์ม และเซนเซอร์วัดแสง LDR ตามแผนผังในหัวข้อที่ 3
4. คัดลอก Auth Token จากหน้า **Devices** บน Blynk.Cloud มาใส่ในโค้ด
5. กดปุ่ม **"Start the simulation"** เพื่อเริ่มทดสอบ

---

## 8. ตารางกรณีทดสอบ (Test Cases & Verification Matrix)

> [!NOTE]
> **ตารางบันทึกผลการทดสอบระบบ**
>
> | กรณีทดสอบ | การตั้งค่าบน Blynk App | สภาพแวดล้อมบน Wokwi (LDR) | ผลลัพธ์ที่คาดหวัง | ผลการทดลองจริง |
> | :---: | :--- | :--- | :--- | :---: |
> | **TC-1: ทดสอบ Manual Mode** | ปรับโหมด `V0` = **Manual**<br>กดเปิดสวิตช์ `V1` และ `V3`<br>กดปิดสวิตช์ `V2` และ `V4` | เลื่อนค่าแสง LDR เปลี่ยนแปลงไปมา | หลอด LED 1 (เขียว) และ 3 (เหลือง) สว่าง<br>หลอด LED 2 (แดง) และ 4 (น้ำเงิน) ดับ<br>การเปลี่ยนแปลงแสง LDR ต้องไม่มีผลกระทบต่อหลอดไฟ | Pass / Fail |
> | **TC-2: Auto Mode เมื่อสภาพแสงมืด** | ปรับโหมด `V0` = **Auto**<br>ตั้ง Slider `V6` (เกณฑ์) = **2000** | เลื่อนเซนเซอร์ LDR ให้มีค่าแสงต่ำกว่า 2000 | หลอด LED ทั้ง 4 หลอดต้อง **ติดสว่างทันที**<br>สวิตช์ V1-V4 บนแอปอัปเดตเป็น ON<br>ไฟ LED Widget V7 บนแอปสว่างขึ้น | Pass / Fail |
> | **TC-3: Auto Mode เมื่อสภาพแสงสว่าง** | คงโหมด `V0` = **Auto**<br>เกณฑ์ `V6` คงเดิม = **2000** | เลื่อนเซนเซอร์ LDR ให้สว่างขึ้น มีค่าแสงสูงกว่า 2000 | หลอด LED ทั้ง 4 หลอดต้อง **ดับลงพร้อมกัน**<br>สวิตช์ V1-V4 บนแอปอัปเดตเป็น OFF<br>ไฟ LED Widget V7 บนแอปดับลง | Pass / Fail |
> | **TC-4: การปรับค่า Threshold แบบ Dynamic** | สภาพแสงปัจจุบันอยู่ที่ระดับปานกลาง | ค่าแสงคงที่ | เมื่อใช้นิ้วเลื่อน Slider บนแอปปรับเกณฑ์ตัดผ่านค่าแสงปัจจุบัน สถานะหลอดไฟต้องเปลี่ยนแปลงทันทีโดยไม่ต้องรีสตาร์ตบอร์ด | Pass / Fail |

---

## 9. เกณฑ์การประเมินผล (Evaluation Rubric: 20 คะแนน)

> [!IMPORTANT]
> **เกณฑ์การให้คะแนนแบบละเอียด**
>
> | หัวข้อการประเมิน | รายละเอียดเกณฑ์ (คะแนนเต็ม) | คะแนนที่ได้ |
> | :--- | :--- | :---: |
> | **1. การต่อวงจรฮาร์ดแวร์บน Wokwi (4 คะแนน)** | ต่อ LED 4 ดวงถูกต้องตามขามีกราวด์และตัวต้านทานครบ (2 คะแนน)<br>ต่อเซนเซอร์ LDR เข้าขา Analog ADC1 ถูกต้อง (2 คะแนน) | ..... / 4 |
> | **2. การออกแบบส่วนต่อประสาน Blynk (5 คะแนน)** | กำหนด Virtual Pins และ Datastreams V0-V7 ถูกต้องตรงตามสเปก (2 คะแนน)<br>จัดวาง Dashboard หน้าจอสวยงาม มีชื่อกำกับชัดเจน ใช้งานง่าย (3 คะแนน) | ..... / 5 |
> | **3. ความถูกต้องของตรรกะโปรแกรม (6 คะแนน)** | โหมด Manual สั่งเปิด-ปิดไฟแยก 4 ห้องได้อิสระ (2 คะแนน)<br>โหมด Auto ตรวจจับค่าแสงเปิด-ปิดไฟได้ตรงตามเงื่อนไข (2 คะแนน)<br>แถบ Slider ปรับ Threshold แบบ Dynamic ส่งผลต่อการทำงานทันที (2 คะแนน) | ..... / 6 |
> | **4. คุณภาพของโค้ดและเสถียรภาพ (3 คะแนน)** | ใช้ `BlynkTimer` จัดการงาน ไม่มีคำสั่ง `delay()` ใน `loop()` (2 คะแนน)<br>มีคอมเมนต์ภาษาไทยอธิบายโค้ดชัดเจนเป็นระเบียบ (1 คะแนน) | ..... / 3 |
> | **5. การส่งงานและการทดสอบ (2 คะแนน)** | แนบลิงก์ Wokwi ที่ใช้งานได้จริง และมีผลบันทึก Test Cases ครบ (2 คะแนน) | ..... / 2 |
> | **รวมคะแนนทั้งสิ้น** | | **..... / 20** |

---

## 10. สิ่งที่นักศึกษาต้องส่ง

1. **ลิงก์โครงการ Wokwi:** ที่ตั้งค่าแชร์เป็นสาธารณะ (Shareable Link)
2. **ภาพถ่ายหน้าจอ (Screenshots):**
   * ภาพหน้าจอ Dashboard บนมือถือ/คอมพิวเตอร์ ขณะอยู่ในโหมด Manual และ Auto
   * ภาพวงจรบน Wokwi ขณะหลอดไฟสว่างครบทั้ง 4 ดวง
3. **ไฟล์ซอร์สโค้ด:** ไฟล์ `.ino` ที่เขียนคอมเมนต์ชื่อ-สกุล และรหัสนักศึกษาไว้ที่หัวโปรแกรม

---

## 11. ภารกิจท้าทายเพื่อคะแนนพิเศษ (Bonus Challenge +3 คะแนน)

> [!TIP]
> **ภารกิจ: ระบบป้องกันหลอดไฟกระพริบรัว (Hysteresis Deadband)**
> ในการใช้งานจริง แสงอาทิตย์เวลาพลบค่ำมักจะมีความผันผวนเล็กน้อย หากเราตั้งเกณฑ์ไว้ที่ `1500` แล้วค่าแสงแกว่งไปมาระหว่าง `1499` กับ `1501` หลอดไฟจะกระพริบติดๆ ดับๆ สร้างความเสียหายให้กับหลอดไฟและรบกวนผู้อยู่อาศัย
> 
> **โจทย์โบนัส:** ให้นักศึกษาปรับปรุงตรรกะในฟังก์ชัน `checkLightAndProcessLogic()` โดยเพิ่มช่วง **Hysteresis Deadband (±100 หน่วย)** กล่าวคือ:
> - ไฟจะเปิด ก็ต่อเมื่อ แสงมืดลงจนต่ำกว่า `Threshold - 100` (เช่น < 1400)
> - ไฟจะปิด ก็ต่อเมื่อ แสงสว่างขึ้นจนเกินกว่า `Threshold + 100` (เช่น > 1600)
> 
> นักศึกษาที่ทำฟังก์ชันนี้สำเร็จและอธิบายหลักการทำงานได้ รับคะแนนพิเศษเพิ่มทันที +3 คะแนน
