# Retail BI Dashboard – HomeDepo (Demo)

This repository contains the **Power BI portfolio** with included image placeholders.

---

## 📊 Executive Dashboard  
![Executive](images/executive.png)

## 🟦 Branch Performance  
![Branch](images/branch.png)

## 🟨 Product & Category Performance  
![Product](images/product.png)

## 🟥 Stock Insights (DOC & Stock Speed)  
![Stock](images/stock.png)

## 📦 Stock Speed Algorithm

ส่วนนี้อธิบายวิธีคำนวณ **Stock Speed** เพื่อจัดกลุ่มสินค้าเป็น  
`Fast`, `Medium`, `Slow`, `No Sales` จากยอดขายและจำนวนคงเหลือในสต็อก

ใช้สำหรับทำ ABC/XYZ แบบง่าย ๆ และสร้าง KPI เช่น  
- สัดส่วนสินค้าค้างสต็อก (No Sales %)  
- สัดส่วนสินค้าขายเร็ว (Fast %)  
- Days of Cover (สต็อกพอขายได้อีกกี่วัน)

---

### 1. Input Data

ใช้ข้อมูลจาก 2 แหล่งหลัก

1. `stocks`
   - `branch`          – สาขา
   - `product_id`      – รหัสสินค้า
   - `quantity`        – จำนวนคงเหลือในสต็อก (On hand)
   - `total_cost`      – มูลค่าต้นทุนรวม

2. `sales + sale_items`
   - `sale_date`       – วันที่ขาย
   - `branch`          – สาขา
   - `product_id`      – รหัสสินค้า
   - `quantity`        – จำนวนที่ขาย

---

### 2. Metrics ที่คำนวณ

#### 2.1 Sales 90 Days

เราดูยอดขายย้อนหลัง **90 วันล่าสุด** ต่อ `branch, product_id`

```sql
sales_qty_90d = SUM(quantity in last 90 days)
last_sale_date = MAX(sale_date in last 90 days)

---
2.2 Average Daily Sales (90 days)

ค่าเฉลี่ยยอดขายต่อวันในช่วง 90 วัน

avg_daily_sales_90d =
    CASE
        WHEN sales_qty_90d = 0 THEN 0
        ELSE sales_qty_90d / 90.0
    END
2.3 Days of Cover

จำนวน “วัน” ที่สต็อกปัจจุบันจะพอขายได้ ถ้ายอดขายเฉลี่ยยังเท่าเดิม

days_of_cover =
    CASE
        WHEN sales_qty_90d = 0 THEN NULL      -- ไม่เคยขาย → ไม่มีค่า days_of_cover
        ELSE on_hand_qty / avg_daily_sales_90d
    END


Interpretation:

days_of_cover = 10 → สต็อกพอขายอีก 10 วัน

days_of_cover = 45 → พอขาย 1.5 เดือน

days_of_cover > 90 → สต็อกเยอะเมื่อเทียบกับยอดขาย

3. การจัดกลุ่ม Stock Speed

เรานิยาม stock_speed จาก sales_qty_90d และ days_of_cover ตามตารางนี้:

Condition	stock_speed
sales_qty_90d = 0	No Sales
on_hand_qty <= 0	No Stock (optional)
days_of_cover <= 30	Fast
30 < days_of_cover <= 90	Medium
days_of_cover > 90	Slow

## 📁 Project Structure
```
powerbi_portfolio/
│ README.md
└── images/
    ├── executive.png
    ├── branch.png
    ├── product.png
    └── stock.png
```

---

## 🧠 Author
**Thana Pattanaverakit**  
Data Analytics | BI | SQL | AI Automation
