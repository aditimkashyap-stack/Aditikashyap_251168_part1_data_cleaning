[README.md](https://github.com/user-attachments/files/29364838/README.md)
# Part 1: Business Data Cleaning, Validation & Excel Reporting

**Repository:** `studentname_studentid_part1_data_cleaning`

---

## Problem Summary

A retail company exports order-level sales data from multiple internal systems. The raw dataset (`raw_orders.xlsx`) contains 932 records with numerous data quality issues including inconsistent text formatting, mixed date formats, duplicate records, missing values, invalid discounts, calculation mismatches, and order status inconsistencies.

The goal is to produce a clean, validated, analysis-ready dataset along with Excel summary reports for business review.

---

## Dataset Description

| Attribute | Value |
|---|---|
| File | `data/raw_orders.xlsx` |
| Raw Records | 932 |
| Columns | 21 |
| Date Range | 2024–2025 |
| Domain | Indian retail — orders, customers, products, shipping |

### Columns
`order_id`, `order_date`, `ship_date`, `customer_id`, `customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `product_name`, `ship_mode`, `quantity`, `unit_price`, `discount`, `sales`, `cost`, `profit`, `payment_status`, `order_status`

---

## Tools Used

- **Python 3.12** — data cleaning and transformation
- **pandas** — data loading, manipulation, and aggregation
- **openpyxl** — Excel file creation with formatting, formulas, charts
- **python-dateutil** — parsing multiple mixed date formats
- **matplotlib** — generating screenshot previews
- **Microsoft Excel / LibreOffice** — manual review and validation

---

## Cleaning Steps Performed

### Step 1: Preserve Raw Data
Original file kept unchanged at `data/raw_orders.xlsx`. All cleaning performed in `data/cleaned_orders.xlsx`.

### Step 2: Text Field Standardization
- Stripped leading/trailing whitespace from all 10 text columns
- Collapsed internal multiple spaces (e.g. `Standard  Class` → `Standard Class`)
- Applied controlled vocabulary mapping for: `segment`, `region`, `category`, `order_status`, `payment_status`, `ship_mode`
- Applied `.title()` case normalization for: `customer_name`, `state`, `city`, `sub_category`

### Step 3: Date Cleaning
- Parsed 5+ different date formats using `dateutil.parser` with `dayfirst=False`
- Standardized to `DD-MMM-YYYY` format
- Computed `shipping_delay_days`, `order_month`, `order_year`
- Flagged 94 records where ship date precedes order date

### Step 4: Duplicate Handling
- Identified and removed 20 exact duplicate rows
- Identified 12 conflicting duplicate order_ids — flagged as `Invalid`, retained for audit

### Step 5: Missing Value Treatment
- `region` (26 missing): Filled with `Unknown` per business rule
- `ship_mode` (22 missing): Filled with `Unknown` per business rule
- `discount` (18 missing): Treated as `0.0` where all other sales fields are valid

### Step 6: Discount Validation
- 15 negative discount values flagged as `Invalid`
- `calculated_sales` uses `cleaned_discount.clip(lower=0)` to prevent inflation

### Step 7: Sales & Profit Recalculation
- `calculated_sales = quantity × unit_price × (1 - cleaned_discount)`
- `calculated_profit = calculated_sales - cost`
- `profit_margin = calculated_profit / calculated_sales`
- 56 records had sales mismatches > ₹0.50 — recalculated

### Step 8: Data Quality Flagging
Each record receives a `data_quality_flag`:
- **Clean** — passes all rules (714 records)
- **Warning** — minor issues (70 records)
- **Invalid** — serious data problems (128 records)

---

## Business Rules Applied

| Rule | Action |
|---|---|
| Missing region | Fill `Unknown`, flag in report |
| Missing ship_mode | Fill `Unknown`, flag in report |
| Missing discount | Treat as `0` if other sales fields valid |
| Negative discount | Flag as `Invalid` |
| Cancelled orders | Excluded from sales pivots |
| Failed payments | Excluded from sales pivots |
| Refunded/Returned orders | Summarized separately |
| Ship date before order date | Flag as `Invalid` |
| Exact duplicate rows | Remove entirely |
| Conflicting duplicate order_ids | Retain + flag `Invalid` |

---

## Summary of Data Quality Issues Found

| Issue | Count | Action |
|---|---|---|
| Exact duplicate rows | 20 | Removed |
| Conflicting duplicate order_ids | 12 unique IDs | Flagged |
| Missing region | 26 | Filled Unknown |
| Missing ship_mode | 22 | Filled Unknown |
| Missing discount | 18 | Treated as 0 |
| Negative discount | 15 | Flagged Invalid |
| Ship date before order date | 94 | Flagged Invalid |
| Sales calculation mismatch | 56 | Recalculated |
| Text/case inconsistencies | All text columns | Standardized |
| Mixed date formats | All date columns | Parsed & standardized |

---

## Summary of Final Pivot Reports

### `outputs/pivot_summary.xlsx`

| Sheet | Description |
|---|---|
| **Sales by Region** | Sales & profit by region, sorted descending by sales. South is the top region. Includes bar chart. |
| **Sales by Category** | Sales & profit by category & sub-category, sorted by sales. Technology leads. Includes horizontal bar chart. |
| **Orders by Ship Mode** | Order count & average shipping delay by ship mode. Standard Class is most used. |
| **Margin by Segment** | Profit margin by customer segment, sorted descending. |
| **Exceptions by Region** | Cancelled, returned, failed orders by region. |
| **Monthly Sales Trend** | Monthly sales and profit trend with line chart. |

---

## Key Business Insights

1. **Total validated sales (Completed + Paid): ₹59,59,649** across 912 cleaned records
2. **Overall profit margin: 28.9%** — healthy but varies significantly by category
3. **South region** generates the highest sales volume
4. **Technology category** leads in revenue; Office Supplies leads in order count
5. **94 shipping records** show ship date before order date — logistics data integrity issue requiring source-system investigation
6. **163 returned orders** represent significant potential revenue leakage — recommend return reason analysis
7. **145 cancelled orders** — cancellation rate analysis by region/segment recommended

---

## Assumptions and Limitations

- Maximum valid discount assumed to be 65% (based on data maximum)
- Ambiguous date formats parsed with `dayfirst=False` (US convention)
- For conflicting duplicate order_ids, first occurrence is retained as canonical
- Cost field not independently validated — taken as-is
- No master product/customer reference table available for cross-validation
- All monetary values assumed in Indian Rupees (₹)

---

## Screenshots Included

| File | Description |
|---|---|
| `screenshots/raw_data_preview.png` | First 12 records of raw dataset before cleaning |
| `screenshots/cleaned_data_preview.png` | First 12 records of cleaned dataset with calculated columns and quality flags |
| `screenshots/pivot_summary_1.png` | Sales & Profit by Region — bar chart and summary table |
| `screenshots/pivot_summary_2.png` | Monthly Sales Trend & Sales by Category |

---

## Repository Structure

```
part1_data_cleaning/
├── data/
│   ├── raw_orders.xlsx          ← Original raw dataset (unchanged)
│   └── cleaned_orders.xlsx      ← Cleaned, validated dataset with calculated columns
├── outputs/
│   ├── data_quality_report.xlsx ← 7-sheet quality report
│   ├── pivot_summary.xlsx       ← 6-sheet pivot analysis with charts
│   └── cleaning_log.md          ← Detailed log of all cleaning decisions
├── screenshots/
│   ├── raw_data_preview.png
│   ├── cleaned_data_preview.png
│   ├── pivot_summary_1.png
│   └── pivot_summary_2.png
└── README.md
```
