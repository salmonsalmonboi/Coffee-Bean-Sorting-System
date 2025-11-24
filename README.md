# Automated Coffee Bean Sorting System (YOLO + Arduino + Pneumatic)

โปรเจกต์นี้เป็นระบบคัดแยกเมล็ดกาแฟอัตโนมัติ
ใช้โมเดล YOLO ตรวจจับและจำแนกเมล็ดดี/เมล็ดเสียแบบเรียลไทม์
แล้วส่งคำสั่งไปยัง Arduino เพื่อสั่งงานกระบอกสูบลม (Pneumatic Cylinder)
ให้ดีดเมล็ดเสียออกจากสายพาน

> โปรเจกต์นี้พัฒนาสำหรับใช้เป็นโครงงานวิศวกรรม และใช้เป็นตัวอย่างผลงาน (Portfolio) ด้าน Machine Learning + Embedded System

---

## Features

- ใช้ **YOLO** (Ultralytics) สำหรับจำแนกเมล็ดกาแฟหลายคลาส  
  - GOOD: `A, AA, AAA, B, Dry, Honey, Pea berry, Wash`  
  - BAD: `Black, Chipped, Elephant ear, Faded, Split, Triangle, Weevil-infested`
- มีกลไก **Frame Stability + Majority Voting**:
  - ต้องเห็นวัตถุหลายเฟรม (`FRAMES_TO_CONFIRM_STABILITY`)
  - ใช้ประวัติ class หลายค่า (`CLASS_VOTING_HISTORY`) แล้วโหวต
- ระบบ **Arming Zone + Lead Time**:
  - กำหนดโซน X บนภาพที่ใช้เป็นจุด “เตรียมยิง” (arming zone)
  - มีเวลาเตรียม (`LEAD_TIME_MS`) ก่อนที่เมล็ดจะถึงจุดยิงจริง
- เชื่อมต่อกับ **Arduino** ผ่าน Serial:
  - Python ส่งคำสั่ง `'A'` ไปยัง Arduino เพื่อ “ARM” การยิง
  - Arduino รอ trigger จากเซ็นเซอร์ → ยิงกระบอกสูบ → ส่ง status กลับ
- มี **ระบบ Logging**:
  - บันทึก log ข้อความลงไฟล์ + แสดงบนหน้าจอ
  - บันทึก event ลง `events.csv` (เช่น confirm class, eject, cleanup)
  - เซฟภาพเฟรม / ครอปเมล็ดตอน confirm และตอน eject แยกตามคลาส

---
## System Overview

1) กล้อง Webcam จับภาพเมล็ดบนสายพานแบบเรียลไทม์

2) YOLO ตรวจจับเมล็ดและระบุตัวตนแต่ละเมล็ดด้วย ID (object tracking)

3) สำหรับแต่ละ ID:

  - เก็บประวัติ class หลายเฟรม → majority vote → ได้ confirmed_class

  - นับสถิติเมล็ดดี / เมล็ดเสีย

4) ถ้าเป็น เมล็ดเสีย และเข้ามาใน Arming Zone:

  - เมื่อเวลาตรงกับ LEAD_TIME_MS → Python ส่งคำสั่ง 'A' ไป Arduino

5) ฝั่ง Arduino:

  - เปิดสถานะ isArmedForEjection = true

  - รอเซ็นเซอร์ตรวจเจอเมล็ด (บนตำแหน่งยิง)

  - ถ้า Armed → สั่งรีเลย์ให้กระบอกสูบยืด/หด ตาม CYLINDER_EXTEND_DURATION_MS / CYLINDER_RETRACT_DURATION_MS

  - ส่งข้อความสถานะกลับ Python เช่น: STATUS:BAD_SEED_EJECTED STATUS:GOOD_SEED_PASSED

6) Python แสดงผลบนหน้าจอ + บันทึก log / ภาพสำหรับวิเคราะห์ภายหลัง

## Communication Protocol (Python ↔ Arduino)

- **Python → Arduino**

  - ส่ง byte 'A' เพื่อ ARM เมล็ดที่เป็น defect (เตรียมยิง)

- **Arduino → Python**

  - ข้อความ startup: "Enhanced Coffee Sorter - Arduino Ready"

- **ขณะทำงาน:**

  - "System ARMED. Waiting for sensor trigger..."

  - "STATUS:BAD_SEED_EJECTED" เมื่อยิงเมล็ดเสียแล้ว

  - "STATUS:GOOD_SEED_PASSED" เมื่อเมล็ดดีผ่านไป

  - Heartbeat / log อื่น ๆ เช่น
  `"Heartbeat - Armed: ... | Ready: ... | Total: ..."`

## Project Structure

```
.
├── src/
│   └── Coffee-Bean-Classification-System.py           # โค้ด Python: YOLO + Webcam + Tracking + Serial
│       └── requirements.txt                           # Python dependencies
├── arduino/
│   └── coffee_sorter_controller/
│       └── coffee_sorter_controller.ino               # โค้ด Arduino: คุมรีเลย์, เซ็นเซอร์, กระบอกสูบ
├── docs/
│   └── ...                                            # รูปเครื่อง, diagram, ฯลฯ
├── .gitignore
└── README.md

```
## Author
A. Chetsada jinamoon, B.Supakorn Sirimueangmoon , and C. Audsadakorn Jeerahut, D. Kwanchai Eurviriyanukul, G.  Pranote pookkapund
> Department of Computer Engineering, Faculty of Engineering, Rajamangala University of Technology Lanna Chiang Mai, Thailand 

