# Beyond MMR — ระบบ Data Pipeline & Analytics (Python + Postgres + Google Sheets)

โปรเจกต์นี้เป็น **End-to-End Data Pipeline** สำหรับวิเคราะห์ “ราคาขายรถมือสองเทียบกับ MMR” โดยทำงานตั้งแต่ ingest ไฟล์ CSV → ทำความสะอาด/สร้างฟีเจอร์ใน Postgres → ส่งตารางที่พร้อมใช้ขึ้น Google Sheets → ต่อ Looker Studio ทำ Dashboard

---

## จุดเด่นสำหรับพอร์ต (Portfolio Highlights)
- สร้าง **ETL Pipeline ครบวงจร** ด้วย **Dockerized Postgres** (แยกชั้น raw → production)
- ทำ **Data Cleaning + Feature Engineering** เพื่อเปรียบเทียบ `sellingprice` vs `mmr`
- ส่งตารางผลลัพธ์ที่พร้อมทำ BI ขึ้น **Google Sheets** (ต่อ Looker Studio ได้ทันที)
- ออกแบบผลลัพธ์ให้วิเคราะห์ได้ทั้ง **รายเวลา** และ **แยกตามกลุ่ม** (ยี่ห้อ/รัฐ/รายเดือน)

---

## ภาพรวมโปรเจกต์
**เป้าหมาย:** เปลี่ยนข้อมูลขายรถมือสองดิบให้เป็นตารางที่ “พร้อมวิเคราะห์” เพื่อวัดผลว่าราคาขายจริง **สูง/ต่ำกว่า MMR** มากน้อยแค่ไหน และมีแพทเทิร์นอะไรตามช่วงเวลา/ภูมิภาค/สภาพรถ

**คำถามหลัก:**
- รถขาย **สูงกว่า/ต่ำกว่า MMR** เท่าไหร่?
- สัดส่วน “Above MMR” แตกต่างกันไหมตาม **ยี่ห้อ (make)** / **รัฐ (state)** / **เวลา (month)** / **สภาพรถ (condition)**

---

## โครงสร้างการทำงาน (Architecture)
```text
car_prices.csv
  → Postgres (raw_data.vehicle_sales)
  → Postgres (production.* ตารางพร้อมใช้ + ฟีเจอร์ที่สร้างเพิ่ม)
  → Google Sheets (ตารางสำหรับ BI)
  → Looker Studio (Dashboard)
เครื่องมือที่ใช้ (Tech Stack)
Python (สคริปต์ ETL)

PostgreSQL (เก็บข้อมูล + แปลงข้อมูล)

Docker / Docker Compose (รันฐานข้อมูลแบบ local)

Google Sheets API (ชั้น publish)

Looker Studio (ชั้น visualize)

วิธีรันแบบเร็ว (Quick Start)
1) ติดตั้ง dependencies
bash
Copy code
python -m pip install -r requirements.txt
2) เปิด Postgres ด้วย Docker
bash
Copy code
docker compose up -d
docker compose ps
3) ตั้งค่า environment variables
คัดลอก .env.example → .env แล้วกรอกค่าตามจริง

Database

DB_USER=car_user

DB_PASSWORD=car_password

DB_NAME=car_db

DB_HOST=localhost

DB_PORT=5432

Google Sheets

GOOGLE_SHEETS_SPREADSHEET_ID=YOUR_SHEET_ID

Credentials (เลือกอย่างใดอย่างหนึ่ง):

GOOGLE_APPLICATION_CREDENTIALS=./service_account.json

หรือ GOOGLE_CREDENTIALS_JSON=YOUR_JSON_STRING

ห้ามอัปขึ้น GitHub: .env และไฟล์ credential จริง

4) รันทั้ง pipeline (ingest → transform → publish)
bash
Copy code
python -u run_pipeline.py
5) รันเฉพาะ publish (กรณีมีตาราง production แล้ว)
bash
Copy code
python -u publish.py
6) ปิดบริการ
bash
Copy code
docker compose down
ตรรกะการทำความสะอาด/สร้างฟีเจอร์ (Data & Transform Logic)
คอลัมน์หลักที่ต้องมี
vin, year, state, saledate, sellingprice, mmr, odometer, condition

กติกาการทำความสะอาด (ตัวอย่าง)
sellingprice > 0 และ mmr > 0

year อยู่ในช่วงที่สมเหตุสมผล (เช่น 1950 → ปีปัจจุบัน+1)

odometer อยู่ในช่วงที่เป็นไปได้

condition อยู่ในสเกลที่กำหนด (เช่น 0–5)

แปลง saledate ให้เป็น datetime ที่ใช้งานได้ และจัดการ timezone ให้สม่ำเสมอ

ฟีเจอร์ที่สร้างเพิ่ม (ตัวอย่าง)
price_diff = sellingprice - mmr

price_diff_pct = price_diff / mmr

is_above_mmr (true/false)

vehicle_age

time buckets: sale_year, sale_month, sale_quarter

odometer_per_year

ผลลัพธ์ (Outputs)
ตาราง Production ใน Postgres
production.vehicle_sales_enriched — ข้อมูลที่ clean + มีฟีเจอร์เพิ่ม

production.sales_summary_by_make_month — สรุปยอด/ค่าเฉลี่ยรายเดือนตาม make

production.sales_summary_by_state_month — สรุปยอด/ค่าเฉลี่ยรายเดือนตาม state

ส่งขึ้น Google Sheets
ส่งตารางที่จำเป็นขึ้น spreadsheet เพื่อเชื่อม BI

แนะนำรูปแบบวันที่ YYYY-MM-DD เพื่อ group/filter ใน Looker Studio ง่าย

โครงสร้างไฟล์ (Project Structure)
text
Copy code
.
├── ingest.py              # CSV → raw_data.vehicle_sales
├── transform.py           # raw_data → production tables + features
├── publish.py             # production → Google Sheets
├── run_pipeline.py        # รัน ingest → transform → publish
├── docker-compose.yml     # ตั้งค่า Postgres
├── requirements.txt       # รายการ dependencies
├── car_prices.csv         # ชุดข้อมูลต้นทาง
├── .env.example           # ตัวอย่างไฟล์ตั้งค่า
├── README.md
└── .gitignore
ลิงก์ Dashboard (ใส่เพิ่มได้)
Looker Studio: [ใส่ลิงก์ที่นี่]

Google Sheet: [ใส่ลิงก์ที่นี่]

ตัวอย่างมุมมองที่แนะนำ:

เปรียบเทียบ MMR vs ราคาขายจริง (distribution / avg diff)

% ขายสูงกว่า MMR แยกตาม make/state/month

แนวโน้ม price_diff_pct ตามเวลา

ผลกระทบของ condition/odometer ต่อ price_diff

แก้ปัญหาเบื้องต้น (Troubleshooting)
Docker ไม่รัน: เปิด Docker Desktop แล้วสั่ง docker compose up -d

credential Google ผิด: ตรวจ path GOOGLE_APPLICATION_CREDENTIALS หรือค่า JSON env

ส่งข้อมูลขึ้น Sheets แล้วชน limit: ส่งเฉพาะ summary ก่อน หรือปรับให้ publish แบบแบ่งชุดใน publish.py

::contentReference[oaicite:0]{index=0}
