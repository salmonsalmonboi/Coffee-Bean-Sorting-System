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

## Project Structure

```
.
├── src/
│   └── Coffee-Bean-Classification-System.py           # โค้ด Python: YOLO + Webcam + Tracking + Serial
├── arduino/
│   └── coffee_sorter_controller/
│       └── coffee_sorter_controller.ino               # โค้ด Arduino: คุมรีเลย์, เซ็นเซอร์, กระบอกสูบ
├── docs/
│   └── ...                                            # รูปเครื่อง, diagram, ฯลฯ
├── requirements.txt                                   # Python dependencies
├── .gitignore
└── README.md

```