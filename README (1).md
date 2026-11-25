#  Pizza Sales Analysis — SQL + Power BI

An end-to-end sales analysis of a pizza restaurant for **Jan 2015 – Dec 2015**.
SQL ( SQL Server) is used to calculate the business KPIs and trends, and
Power BI is used to turn the same numbers into an interactive 2-page dashboard.

---

## 📌 Table of Contents
1. [Project Overview](#-project-overview)
2. [Tools Used](#-tools-used)
3. [Dataset](#-dataset)
4. [Dashboard Pages](#-dashboard-pages)
5. [KPIs](#-kpis)
6. [SQL Queries](#-sql-queries)
7. [SQL Query to Dashboard Mapping](#-sql-query-to-dashboard-mapping)
8. [Key Insights](#-key-insights)
9. [Repository Structure](#-repository-structure)
10. [How to Use](#-how-to-use)
11. [Author](#-author)

---

## 📖 Project Overview

The goal of this project is to answer simple business questions about a pizza
restaurant:

- How much revenue did we make, and how many orders and pizzas did we sell?
- On which days and in which months are orders highest?
- Which pizza categories and sizes bring the most sales?
- Which pizzas are the best sellers and which are the worst sellers?

The work has two parts:

1. **SQL** — queries on the `pizza_sales` table to calculate KPIs and trends.
2. **Power BI** — an interactive dashboard with filters (slicers) and page navigation.

---

## 🛠 Tools Used

| Tool | Purpose |
|------|---------|
| SQL Server (T-SQL) | Writing queries for KPIs, trends and top/bottom analysis |
| Power BI Desktop | Dashboard design, measures, slicers and page navigation |
| DAX | Measures used in the dashboard (Total Revenue, Total orders, etc.) |

---

## 🗂 Dataset

- **Table:** `pizza_sales`
- **Period:** 01 Jan 2015 – 31 Dec 2015
- **Columns used in the queries:**

| Column | Meaning |
|--------|---------|
| `order_id` | Unique ID of each order |
| `order_date` | Date of the order |
| `pizza_name` | Name of the pizza |
| `pizza_category` | Classic, Supreme, Chicken or Veggie |
| `pizza_size` | S, M, L, XL or XXL |
| `quantity` | Number of pizzas in the line item |
| `total_price` | Price of the line item (quantity × unit price) |

---

## 📊 Dashboard Pages

### Page 1: HOME
- 5 KPI cards: Total Revenue, Avg Order Value, Total Pizzas Sold, Total Orders, Avg Pizzas per Order
- Daily Trend for Total Orders (column chart)
- Monthly Trend for Total Orders (area chart)
- % of Sales by Pizza Category (donut chart)
- % of Sales by Pizza Size (donut chart)
- Total Pizzas Sold by Pizza Category (funnel chart)
- Slicers: **Pizza Category** and **Order Date** range

### Page 2: Best / Worst Seller
- Top 5 and Bottom 5 pizzas by **Revenue**
- Top 5 and Bottom 5 pizzas by **Quantity**
- Top 5 and Bottom 5 pizzas by **Total Orders**
- Same KPI cards and slicers as the HOME page

Navigation between the two pages is done with **Page Navigator** buttons.

# screenshots

<img width="1258" height="762" alt="Screenshot 2026-11-25 111331" src="https://github.com/user-attachments/assets/660ced64-ebec-49a5-978d-47ca7edfaced" />
<img width="1245" height="758" alt="Screenshot 2026-11-25 111408" src="https://github.com/user-attachments/assets/b6360dd5-f75a-41e8-b8e3-a652ea2fcfcf" />
<img width="1235" height="762" alt="Screenshot 2026-11-25 111447" src="https://github.com/user-attachments/assets/62642b3e-a893-4683-9ad9-a8b3d68b0d13" />


---

## 🎯 KPIs

| KPI | Value |
|-----|-------|
| Total Revenue | **817.86K** |
| Average Order Value | **38.31** |
| Total Pizzas Sold | **49,574** |
| Total Orders | **21,350** |
| Average Pizzas per Order | **2.32** |

---

## 🧾 SQL Queries

All queries are written for **SQL Server (T-SQL)** and run on the `pizza_sales` table.

### A. KPIs

**1. Total Revenue**
```sql
SELECT SUM(total_price) AS Total_Revenue
FROM pizza_sales;
```

**2. Average Order Value**
```sql
SELECT (SUM(total_price) / COUNT(DISTINCT order_id)) AS Avg_order_Value
FROM pizza_sales;
```

**3. Total Pizzas Sold**
```sql
SELECT SUM(quantity) AS Total_pizza_sold
FROM pizza_sales;
```

**4. Total Orders**
```sql
SELECT COUNT(DISTINCT order_id) AS Total_Orders
FROM pizza_sales;
```

**5. Average Pizzas Per Order**
```sql
SELECT CAST(CAST(SUM(quantity) AS DECIMAL(10,2)) /
       CAST(COUNT(DISTINCT order_id) AS DECIMAL(10,2)) AS DECIMAL(10,2))
       AS Avg_Pizzas_per_order
FROM pizza_sales;
```

### B. Daily Trend for Total Orders
```sql
SELECT DATENAME(DW, order_date) AS order_day,
       COUNT(DISTINCT order_id) AS total_orders
FROM pizza_sales
GROUP BY DATENAME(DW, order_date);
```

### C. Monthly Trend for Total Orders
```sql
SELECT DATENAME(MONTH, order_date) AS Month_Name,
       COUNT(DISTINCT order_id) AS Total_Orders
FROM pizza_sales
GROUP BY DATENAME(MONTH, order_date);
```

### D. % of Sales by Pizza Category
```sql
SELECT pizza_category,
       CAST(SUM(total_price) AS DECIMAL(10,2)) AS total_revenue,
       CAST(SUM(total_price) * 100 / (SELECT SUM(total_price) FROM pizza_sales) AS DECIMAL(10,2)) AS PCT
FROM pizza_sales
GROUP BY pizza_category;
```

### E. % of Sales by Pizza Size
```sql
SELECT pizza_size,
       CAST(SUM(total_price) AS DECIMAL(10,2)) AS total_revenue,
       CAST(SUM(total_price) * 100 / (SELECT SUM(total_price) FROM pizza_sales) AS DECIMAL(10,2)) AS PCT
FROM pizza_sales
GROUP BY pizza_size
ORDER BY pizza_size;
```

### F. Total Pizzas Sold by Pizza Category
The `WHERE` line filters one month (2 = February). Remove it to get the full year.
```sql
SELECT pizza_category, SUM(quantity) AS Total_Quantity_Sold
FROM pizza_sales
WHERE MONTH(order_date) = 2
GROUP BY pizza_category
ORDER BY Total_Quantity_Sold DESC;
```

### G. Top 5 Pizzas by Revenue
```sql
SELECT TOP 5 pizza_name, SUM(total_price) AS Total_Revenue
FROM pizza_sales
GROUP BY pizza_name
ORDER BY Total_Revenue DESC;
```

### H. Bottom 5 Pizzas by Revenue
```sql
SELECT TOP 5 pizza_name, SUM(total_price) AS Total_Revenue
FROM pizza_sales
GROUP BY pizza_name
ORDER BY Total_Revenue ASC;
```

### I. Top 5 Pizzas by Quantity
```sql
SELECT TOP 5 pizza_name, SUM(quantity) AS Total_Pizza_Sold
FROM pizza_sales
GROUP BY pizza_name
ORDER BY Total_Pizza_Sold DESC;
```

### J. Bottom 5 Pizzas by Quantity
```sql
SELECT TOP 5 pizza_name, SUM(quantity) AS Total_Pizza_Sold
FROM pizza_sales
GROUP BY pizza_name
ORDER BY Total_Pizza_Sold ASC;
```

### K. Top 5 Pizzas by Total Orders
```sql
SELECT TOP 5 pizza_name, COUNT(DISTINCT order_id) AS Total_Orders
FROM pizza_sales
GROUP BY pizza_name
ORDER BY Total_Orders DESC;
```

### L. Bottom 5 Pizzas by Total Orders
```sql
SELECT TOP 5 pizza_name, COUNT(DISTINCT order_id) AS Total_Orders
FROM pizza_sales
GROUP BY pizza_name
ORDER BY Total_Orders ASC;
```

---

## 🔗 SQL Query to Dashboard Mapping

| SQL Query | Power BI Visual | Page |
|-----------|-----------------|------|
| A1 – A5 (KPIs) | 5 KPI cards | HOME and Best/Worst Seller |
| B. Daily Trend | Daily Trend for Total Orders (column chart) | HOME |
| C. Monthly Trend | Monthly Trend for Total Orders (area chart) | HOME |
| D. % Sales by Category | % of Sales by Pizza Category (donut) | HOME |
| E. % Sales by Size | % of Sales Category (donut, by size) | HOME |
| F. Pizzas Sold by Category | Total Pizza Sold by Pizza Category (funnel) | HOME |
| G / H. Top / Bottom 5 by Revenue | Top 5 / Bottom 5 by Revenue (bar charts) | Best/Worst Seller |
| I / J. Top / Bottom 5 by Quantity | Top 5 / Bottom 5 Pizzas by Quantity (bar charts) | Best/Worst Seller |
| K / L. Top / Bottom 5 by Orders | Top 5 / Bottom 5 Pizzas by Total Orders (bar charts) | Best/Worst Seller |

---

## 💡 Key Insights

- **Weekends are the busiest:** orders are highest on Friday and Saturday evenings.
- **Peak months:** July and January have the maximum orders.
- **Classic is the top category:** about 26.91% of revenue, and the most pizzas sold (14,888).
- **Large size leads:** Large pizzas bring about 45.89% of sales, followed by Medium (30.49%) and Regular (21.77%).
- **Best seller by revenue:** The Thai Chicken Pizza.
- **Best seller by quantity and orders:** The Classic Deluxe Pizza.
- **Worst seller:** The Brie Carre Pizza has the lowest revenue, quantity and orders.

**Business suggestions**
- Run weekend offers and keep extra staff on Friday and Saturday.
- Promote the low-selling pizzas with combos, or review the menu for the ones that never sell.
- Keep Large and Medium sizes well stocked, since they drive most of the sales.

---

## 📁 Repository Structure

```
pizza-sales-analysis/
│
├── README.md
├── pizza_power_bi_project.pbix      # Power BI dashboard file
├── PIZZA_SALES_SQL_QUERIES.docx     # SQL queries document
├── sql/
│   └── pizza_sales_queries.sql      # (optional) same queries as a .sql file
└── images/
    ├── home-page.png                # dashboard screenshot - HOME
    └── best-worst-seller-page.png   # dashboard screenshot - Best/Worst Seller
```

---

## ▶️ How to Use

1. **Clone** this repository.
   ```bash
   git clone https://github.com/<your-username>/<your-repo-name>.git
   ```
2. Open the `.pbix` file in **Power BI Desktop**.
3. In Power BI Desktop, use **Ctrl + Click** on the navigation buttons to switch pages.
   After publishing, a normal click works.
4. To practise the SQL, load the pizza sales data into a table named `pizza_sales`
   in **SQL Server** and run the queries from the [SQL Queries](#-sql-queries) section.

---

## 👤 Author

**Balaga Manohar Babu**
- GitHub: [your-github-link]
- LinkedIn: [your-linkedin-link]
- Email: [your-email]

⭐ If you like this project, please give it a star!
