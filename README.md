# Beyond MMR — Data Pipeline & Analytics (Python + Postgres + Google Sheets)

End-to-end data pipeline for used-car pricing analysis: ingest CSV → clean/feature engineer in Postgres → publish to Google Sheets → visualize in Looker Studio.

---

## Portfolio Highlights
- Built an end-to-end **ETL pipeline** with **Dockerized Postgres** (raw → production)
- Implemented **data cleaning + feature engineering** to compare `sellingprice` vs `mmr`
- Published analytics-ready tables to **Google Sheets** (Looker Studio–friendly)
- Designed outputs for **time-series + segmentation** analysis (make/state/month)

---

## Project Overview
**Goal:** Transform raw used-car sales data into analytics-ready tables to evaluate pricing performance **beyond MMR** (Manheim Market Report).

**Key question:** Are vehicles sold **above or below MMR**, by how much, and what patterns exist across time, brand, region, and vehicle condition?

---

## Architecture
```text
car_prices.csv
  → Postgres (raw_data.vehicle_sales)
  → Postgres (production.* tables + engineered features)
  → Google Sheets (curated tables)
  → Looker Studio (dashboard)
Tech Stack
Python (ETL scripts)

PostgreSQL (storage + transformation layer)

Docker / Docker Compose (local database)

Google Sheets API (publishing layer)

Looker Studio (visualization)

Quick Start
1) Install dependencies
bash
Copy code
python -m pip install -r requirements.txt
2) Start Postgres (Docker)
bash
Copy code
docker compose up -d
docker compose ps
3) Configure environment variables
Copy .env.example → .env and fill values.

Database

DB_USER=car_user

DB_PASSWORD=car_password

DB_NAME=car_db

DB_HOST=localhost

DB_PORT=5432

Google Sheets

GOOGLE_SHEETS_SPREADSHEET_ID=YOUR_SHEET_ID

Credentials (choose one):

GOOGLE_APPLICATION_CREDENTIALS=./service_account.json

or GOOGLE_CREDENTIALS_JSON=YOUR_JSON_STRING

Do not commit .env or credential files to GitHub.

4) Run the full pipeline (ingest → transform → publish)
bash
Copy code
python -u run_pipeline.py
5) Publish only (if production tables already exist)
bash
Copy code
python -u publish.py
6) Stop services
bash
Copy code
docker compose down
Data & Transform Logic
Required input columns
vin, year, state, saledate, sellingprice, mmr, odometer, condition

Validation / cleaning rules
sellingprice > 0, mmr > 0

year within a reasonable range (e.g., 1950 → current_year+1)

odometer within realistic bounds

condition within expected scale (e.g., 0–5)

Convert saledate to a usable datetime and ensure consistent timezone handling

Engineered features (examples)
price_diff = sellingprice - mmr

price_diff_pct = price_diff / mmr

is_above_mmr (boolean)

vehicle_age

Time buckets: sale_year, sale_month, sale_quarter

odometer_per_year

Outputs
Postgres production tables
production.vehicle_sales_enriched — cleaned + engineered dataset

production.sales_summary_by_make_month — monthly summary by make

production.sales_summary_by_state_month — monthly summary by state

Google Sheets publishing
Publishes curated tables to a spreadsheet for BI connection

Recommended date format: YYYY-MM-DD for grouping/filtering in Looker Studio

Project Structure
text
Copy code
.
├── ingest.py              # CSV → raw_data.vehicle_sales
├── transform.py           # raw_data → production tables + features
├── publish.py             # production → Google Sheets
├── run_pipeline.py        # run ingest → transform → publish
├── docker-compose.yml     # Postgres service
├── requirements.txt       # Python dependencies
├── car_prices.csv         # Source dataset
├── .env.example           # Environment template
├── README.md
└── .gitignore
Dashboard Links (Optional)
Looker Studio: [ADD_LINK_HERE]

Google Sheet: [ADD_LINK_HERE]

Suggested views:

MMR vs Selling Price (distribution, average diff)

% Above MMR by make/state/month

Trend of price_diff_pct over time

Condition/odometer impact on price_diff

Troubleshooting
Docker not running: open Docker Desktop then docker compose up -d

Google credential error: verify GOOGLE_APPLICATION_CREDENTIALS path or JSON env

Large exports / sheet limits: publish summaries first or chunk publishing logic in publish.py
