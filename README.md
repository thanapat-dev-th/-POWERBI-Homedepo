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

stock speed algorithm
CREATE OR REPLACE VIEW vw_stock_speed AS
WITH stock_base AS (
    SELECT
        st.warehouse_id,
        st.branch,
        st.product_id,
        st.quantity          AS on_hand_qty,
        st.total_cost,
        CASE 
            WHEN st.quantity = 0 THEN NULL
            ELSE st.total_cost::numeric / st.quantity
        END                  AS unit_cost
    FROM stocks st
),

sales_90d AS (
    SELECT
        si.product_id,
        s.branch,
        SUM(si.quantity)          AS sales_qty_90d,
        MAX(s.sale_date)          AS last_sale_date
    FROM sale_items si
    JOIN sales s ON s.sale_id = si.sale_id
    WHERE s.sale_date >= current_date - 90
    GROUP BY si.product_id, s.branch
)

SELECT
    sb.warehouse_id,
    sb.branch,
    sb.product_id,
    sb.on_hand_qty,
    COALESCE(s90.sales_qty_90d, 0) AS sales_qty_90d,
    
    -- Avg daily sales in last 90 days
    CASE 
        WHEN COALESCE(s90.sales_qty_90d, 0) = 0 THEN 0
        ELSE COALESCE(s90.sales_qty_90d, 0)::numeric / 90.0
    END AS avg_daily_sales_90d,

    -- Days of cover = on hand / avg daily
    CASE 
        WHEN COALESCE(s90.sales_qty_90d, 0) = 0 THEN NULL
        ELSE sb.on_hand_qty::numeric / (COALESCE(s90.sales_qty_90d, 0)::numeric / 90.0)
    END AS days_of_cover,

    -- Stock speed bucket
    CASE
        WHEN COALESCE(s90.sales_qty_90d, 0) = 0 THEN 'No Sales'
        WHEN sb.on_hand_qty <= 0 THEN 'No Stock'
        WHEN (sb.on_hand_qty::numeric / (COALESCE(s90.sales_qty_90d, 0)::numeric / 90.0)) <= 30 
            THEN 'Fast'
        WHEN (sb.on_hand_qty::numeric / (COALESCE(s90.sales_qty_90d, 0)::numeric / 90.0)) <= 90 
            THEN 'Medium'
        ELSE 'Slow'
    END AS stock_speed,

    s90.last_sale_date,
    sb.unit_cost
FROM stock_base sb
LEFT JOIN sales_90d s90
  ON s90.product_id = sb.product_id
 AND s90.branch     = sb.branch;


---

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
