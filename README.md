# ElectroHub – Sales & Profit Analytics Dashboard (Power BI)

An interactive Power BI report that analyses four years of retail sales for **ElectroHub**, a multi-category store selling electronics, footwear, clothing, home appliances, accessories, kitchenware, bags and personal-care products across India.

The dashboard answers eight business questions on sales, profit, quantity, discounts, promotions, customers and geography – from a high-level overview down to order-level detail.

## 📌 Business Questions Answered

| # | Requirement | Where it is answered | Visual used |
|---|-------------|----------------------|-------------|
| 1 | Top / Bottom 5 products by Sales, Profit, Quantity Sold | **Top/Bottom 5** page | 6 bar charts (Top N / Bottom N filters on `Product Name`) |
| 2 | Sales trends over time | **Overview** page | Line chart on the date hierarchy (drill Year → Quarter → Month → Day) |
| 3 | Relationship between Sales and Profit | **Overview** page | Scatter chart – Profit (X) vs Net Sales (Y) |
| 4 | Compare Sales / Profit / Quantity between two user-selected periods | **Comparison Sales/Profit/Quantity** page, plus **Edit Interactions** page | Two independent date slicers + 3 column charts driven by `USERELATIONSHIP` measures |
| 5 | Average discount in each discount category | **Overview** page | Bar chart – average discount by `Promotion Name` (blank/no-promotion excluded) |
| 6 | Total number of orders | **Overview** page | Card – Number of Orders |
| 7 | Order-level details filterable by Product / Date / Customer / Promotion | **Table Visuals** page | Detail table + 4 slicers |
| 8 | Sales by city | **Overview** page | Azure Map, bubble size = Net Sales |

---

## 📄 Report Pages

1. **Overview** – sales trend, profit-vs-sales scatter, order count, discount by promotion, sales map.
2. **Top/Bottom 5** – best and worst five products on Net Sales, Profit and Units Sold.
3. **Comparison Sales/Profit/Quantity** – pick *Period 1* and *Period 2* with two date slicers and compare totals side by side.
4. **Edit Interactions** – demonstrates Power BI's *Edit interactions* feature: the left slicer filters only the left column of KPI charts and the right slicer only the right column, giving a second way to compare two periods.
5. **Table Visuals** – full transaction table with slicers for Date, Customer, Product and Promotion.

---

## 🗂️ Data Model

Star schema built from an Excel workbook (`Store+Data.xlsx`) with one fact table and three dimensions.

```
Dim Customers ──┐
Dim Product   ──┼──►  Fact Table  (3,510 orders)
Dim Promotion ──┘

Date Table 1 / Date Table 2  (CALENDARAUTO) – two disconnected-use calendars for period comparison
```

| Table | Rows | Key columns |
|-------|------|-------------|
| **Fact Table** | 3,510 | Date, CustomerID, Product ID, PromotionID, Units Sold, Price Per Unit, Total Sales, Discount %, Discount, Net Sales, Profit, order id |
| **Dim Customers** | 50 | Customer ID, Customer Name, City, State, Pincode |
| **Dim Product** | 30 | ProductID, Product Name, Product Line (8 categories), Price Per Unit (INR) |
| **Dim Promotion** | 5 | PromotionID, Promotion Name, Ad Type, Coupon Code, Price Reduction Type, Percentage |

Relationships are many-to-one, single direction, from the fact table to each dimension.

### Calculated fields (Power Query)

| Field | Logic |
|-------|-------|
| `Total Sales` | `Units Sold × Price Per Unit` |
| `Discount Percentage` | Looked up from the promotion (20 / 10 / 50 / 50 / 70 %; 0 when no promotion) |
| `Discount` | `Total Sales × Discount % / 100` |
| `Net Sales` | `Total Sales − Discount` |
| `Profit` | `0.1 × Net Sales` (a flat 10 % margin assumption) |
| `order id` | Index column (1…3,510) |

### DAX measures (period comparison)

```DAX
sum of Net Sales =
CALCULATE(
    SUM('Fact Table'[Net Sales]),
    ALL('Date Table 1'),
    USERELATIONSHIP('Date Table 2'[Date], 'Fact Table'[Date (dd/mm/yyyy)])
)

Total Profit =
CALCULATE(
    SUM('Fact Table'[Profit]),
    ALL('Date Table 1'),
    USERELATIONSHIP('Fact Table'[Date (dd/mm/yyyy)], 'Date Table 2'[Date])
)

Total Quantity =
CALCULATE(
    SUM('Fact Table'[Units Sold]),
    ALL('Date Table 1'),
    USERELATIONSHIP('Fact Table'[Date (dd/mm/yyyy)], 'Date Table 2'[Date])
)
```

Each chart on the comparison page shows two bars: one filtered by **Date Table 1** (Period 1) and one by **Date Table 2** (Period 2, via `USERELATIONSHIP`).

---

## 🔍 Key Insights (from the data)

| Metric | Value |
|--------|-------|
| Orders | **3,510** |
| Units sold | **7,125** |
| Gross sales | **₹12.95 Cr** |
| Total discount given | **₹71.9 L** (≈ 5.5 % of gross) |
| Net sales | **₹12.23 Cr** |
| Profit (10 % margin) | **₹1.22 Cr** |
| Period covered | Jan 2020 – Dec 2023 (+ 1 order on 1 Jan 2024) |

- **Electronics dominate revenue.** Five electronics products make up ≈ 74 % of net sales (₹9.0 Cr of ₹12.2 Cr), even though they sell a similar number of units to cheap items.
- **Top 5 by net sales:** Apple iPhone 14 (₹2.14 Cr), Apple MacBook Air (₹1.96 Cr), Sony Bravia 55" TV (₹1.94 Cr), Samsung Galaxy S21 (₹1.53 Cr), HP Pavilion Laptop (₹1.44 Cr).
- **Bottom 5 by net sales:** Colgate Toothpaste (₹0.21 L), Dove Soap Pack, Nivea Body Lotion, L'Oréal Shampoo, Tupperware Lunch Box.
- **Units sold tell a different story.** The unit-volume ranking is flat (≈ 200–280 units per product) – the iPhone 14 leads with 281 units, while Borosil Glass Set is last with 203. Sales rank is driven by *price*, not popularity.
- **Yearly net sales are stable:** 2020 ₹3.14 Cr → 2021 ₹3.00 Cr → 2022 ₹2.86 Cr → 2023 ₹3.23 Cr. 2023 is the strongest year, and Q3 2023 (₹98 L) is the best quarter.
- **Promotions touch only ~20 % of orders.** 720 of 3,510 orders used a promotion. *Summer Sale* accounts for 574 of them (80 %).
- **Deepest discounts are the least used.** Weekend Flash Sale (50 % off) has the highest average discount value (≈ ₹22.6 K per order, 83 orders) and Clearance Sale (70 % off) follows at ≈ ₹17.7 K on only 59 orders, versus ≈ ₹7.4 K for the much more common Summer Sale (20 % off).
- **Geography:** Bhopal (₹1.54 Cr), Kanpur (₹1.41 Cr) and Indore (₹1.34 Cr) are the top three cities, followed by Lucknow and Mumbai.

---

## 🛠️ Tools & Techniques

- **Power BI Desktop** – report, model and DAX
- **Power Query (M)** – data cleaning, merges, custom columns, type changes
- **DAX** – `CALCULATE`, `ALL`, `USERELATIONSHIP`
- **Star-schema modelling** with role-playing date tables
- **Top N / Bottom N** visual-level filters
- **Edit Interactions** to isolate slicer–visual relationships
- **Azure Maps** visual, **Advance Card** custom visual
- **Drill-down** on the date hierarchy

---

## ▶️ How to Run

1. Clone the repository
   ```bash
   git clone https://github.com/taha5847/Test.git
   ```
2. Open `Project_1.pbix` in **Power BI Desktop**.
3. The queries point to a local file path (`C:\Users\HP\Downloads\Store+Data.xlsx`). Update it via **Home → Transform data → Data source settings → Change Source**, then point to `data/Store+Data.xlsx`.
4. Click **Refresh**.

---

## ⚠️ Known Limitations & Planned Improvements

These are honest notes on the current version and what I plan to improve:

**Data / model**
- [ ] **Profit is a fixed 10 % of Net Sales**, so Profit vs Net Sales is a perfectly straight line and the Profit ranking is identical to the Sales ranking. Replace with real cost data (e.g. add a `Cost Price` column) to unlock true margin analysis.
- [ ] **2024 contains a single order (1 Jan 2024)**, which shows as a near-zero 2024 point on the yearly trend. Filter it out or label it as a partial period.
- [ ] Move hard-coded file path into a **Power Query parameter** so the report refreshes on any machine.
- [ ] Clean column names (`Net Sales  ` and `Total Sales  ` have trailing spaces) and remove unused objects (`Sum Dim` measure, empty `Measure` table).
- [ ] Replace the two `CALENDARAUTO` tables with a proper marked date table (with Year / Quarter / Month / Weekday) and a *disconnected* period-selector table.
- [ ] Customer table holds e-mail and phone numbers – keep these out of any published/public version of the report.

**Report**
- [ ] The trend line opens at **Year level only**; add a Year/Quarter/Month/Day toggle (field parameter) so all four granularities from the requirement are visible without drilling.
- [ ] Add the **Product Line** (8 categories) as a slicer / treemap – the category breakdown is not currently visible on any page.
- [ ] Add KPI cards (Net Sales, Profit, Units Sold, Avg Discount) with period-over-period % change to the Overview.
- [ ] Table page: slicer uses *Customer Name* rather than *Customer ID*, shows `Product ID` instead of `Product Name`, and *sums* `Price Per Unit` / `Discount Percentage` in totals – switch these to proper names and averages.
- [ ] Verify the aggregation on the **discount bar chart** (should be *Average*, not *Count*) and on the **orders card** (should be *Count of order id*).
- [ ] Use a consistent theme/font (the report currently uses Times New Roman) and add ElectroHub branding, titles and navigation buttons between pages.
- [ ] Add conditional formatting and data labels to the Top/Bottom 5 charts.

---







