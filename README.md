# parthbudhia_bitsom_ba_2511121_part1_data_cleaning


# E-Commerce Sales Data Quality & Analytics Pipeline

## 1. Problem Summary
In any data-driven retail organization, raw transactional logs from checkout and logistical systems are frequently riddled with human errors, system synchronization issues, and formatting discrepancies. This project addresses a messy, unvalidated e-commerce order export (`raw_orders.xlsx`) that suffered from missing attributes, negative temporal logic (ship dates preceding order dates), financial calculation mismatches, and broken discount values. 

The primary objective was to build an automated, rule-based data transformation pipeline that:
1. Audits and separates clean, warning-level, and completely invalid data.
2. Formally documents data quality errors in a structured registry.
3. Produces a production-ready, reconciled master file named `cleaned_orders.xlsx`.
4. Summarizes strategic performance metrics in a multi-tabbed Excel Pivot dashboard (`outputs/pivot_summary.xlsx`).

---

## 2. Dataset Description
The analysis was performed on a transactional sales dataset containing **696 raw records** spanning multi-regional order histories. Each transaction tracks the lifecycle of an order from purchase to shipment across the following core attributes:

*   **Order Identifiers:** `order_id`, `customer_id`, `customer_name`
*   **Temporal Markers:** `order_date`, `ship_date`
*   **Logistical Attributes:** `ship_mode`, `region`
*   **Product Categorization:** `category`, `sub_category`, `product_name`
*   **Financial Metrics:** `sales`, `quantity`, `discount`, `profit`, `unit_price`, `cost`
*   **Status Flags:** `order_status` (Completed, Cancelled, Returned), `payment_status` (Paid, Failed, Refunded), `segment` (Consumer, Corporate, Home Office)

---

## 3. Tools Used
*   **Markdown:** For generating the auditable `outputs/cleaning_log.md`.
*   **Microsoft Excel:** To engineer the final interactive pivot table logic, custom financial fields, and visual summaries inside `outputs/pivot_summary.xlsx`.

---

## 4. Cleaning Steps Performed
The data processing script evaluates the raw dump linearly using a multi-stage validation grid:

1.  **Deduplication:** Hashed entire rows to identify and permanently drop **20 exact row duplicates**, preserving the integrity of unique transactional entries.
2.  **Type & Date Standardization:** Parsed 75 non-standard textual date representations into unified ISO formats (`YYYY-MM-DD`) to enable chronological timeline tracking.
3.  **Financial Shadow-Testing:** Engineered shadow metrics (`calculated_sales` and `calculated_profit`) using standard retail formulas to isolate rows where manually entered totals deviated from mathematically expected yields.
4.  **Imputation:** Populated missing categorical text with systematic fallbacks to prevent data dropping during multi-variable pivot grouping.
5.  **Isolation & Export:** Split the parent data into three standalone data sheets within `cleaned_orders.xlsx`: `Clean` (fully valid records), `Warning` (recomputed/imputed entries), and `Invalid` (critically broken data).

---

## 5. Business Rules Applied
To ensure compliance with organizational accounting and logistics policies, the following business constraints were applied:

| Rule Area | Required Action / Constraint | Pipeline Execution Status |
| :--- | :--- | :--- |
| **Missing Region** | Fill as `"Unknown"` and flag in quality report. | Imputed; moved to Warning state. |
| **Missing Ship Mode** | Fill as `"Unknown"` and flag in quality report. | Imputed; moved to Warning state. |
| **Missing Discount** | Treat as `0` only if all other sales fields are valid. | Imputed to `0.00` if verified; otherwise flagged. |
| **Negative Discount** | Flag as invalid. | Moved to Invalid state; excluded from primary metrics. |
| **Discount Out of Range** | Flag as invalid if outside allowed operational thresholds. | Moved to Invalid state. |
| **Cancelled Orders** | Should not contribute to final completed sales summary. | Filtered and excluded from core revenue pivots. |
| **Failed Payments** | Should not contribute to final completed sales summary. | Filtered and excluded from core revenue pivots. |
| **Refunded Orders** | Must be separately summarized. | Isolated into a dedicated evaluation summary view. |
| **Logistical Anomaly** | Flag as invalid shipping record if `ship_date` is before `order_date`. | Moved to Invalid state (temporal paradox). |

All decision logic, rules applied, and records shifted are documented verbatim in `outputs/cleaning_log.md`.

---

## 6. Summary of Data Quality Issues Found
The automated data audit revealed significant quality risks in the upstream collection system. Full diagnostics are written to `data_quality_report.xlsx`. A high-level summary of issues includes:

*   **25 Missing Regions & 20 Missing Ship Modes:** Inhibited accurate local logistics modeling.
*   **21 Corrupted Discounts:** Included negative promotional percentages and values exceeding maximum limits.
*   **17 Chronological Errors:** Items marked as shipped before the customer placed the order.
*   **37 Status Mismatches:** High-risk discrepancies where transactions were tagged with a `Failed` payment status but marked as `Completed` or `Returned` in the logistics field.
*   **105 Financial Math Mismatches:** Independent sales and profit entries that failed algorithmic validation checks.

---

## 7. Summary of Final Pivot Reports
The workbook `outputs/pivot_summary.xlsx` references the validated records from `cleaned_orders.xlsx` and provides six interactive analytical layers:

1.  **Sales and Profit by Region:** High-level executive view mapping revenue performance across geographical sectors. *(Sorted descending by Sales to highlight top-performing regions).*
2.  **Sales and Profit by Category & Sub-Category:** A nested matrix breaking down operational category metrics.
3.  **Order Count by Ship Mode:** Logistical density tracker highlighting shipping vendor volume. *(Filtered to exclude "Unknown" values for carrier clarity).*
4.  **Profit Margin by Customer Segment:** Evaluates corporate vs. consumer value generation via custom Calculated Fields ($\sum \text{Profit} / \sum \text{Sales}$).
5.  **Refunded/Cancelled/Failed Orders by Region:** A dedicated operational loss matrix grouping non-completed sales value by territory.
6.  **Monthly Sales Trend:** A chronological chart detailing monthly velocity and seasonal purchase patterns across operational years.

---

## 8. Key Business Insights
*   **Regional Dominance:** Sorting the regional pivot demonstrates that a single primary territory drives a disproportionate percentage of net margin, identifying where marketing allocation is most efficient.
*   **Leakage Tracing:** Evaluating the non-completed transaction pivot reveals specific regions suffering from high rates of failed payments and customer refunds, highlighting localized payment-gateway instability or delivery fulfillment friction.
*   **Product Health:** Correlating sub-category revenue against profit margins isolates items generating high top-line revenue but generating negative bottom-line returns due to aggressive discounting or high underlying wholesale costs.

---

## 9. Assumptions and Limitations
*   **Formulaic Supremacy:** The pipeline assumes that if calculated values diverge from entered values, the base components (`Quantity`, `Unit_Price`, `Discount`) are correct, and the aggregated `Sales`/`Profit` cells are wrong.
*   **Operational Intent:** The system can flag operational anomalies (e.g., failed payments marked completed), but cannot distinguish whether these stem from systemic software bugs, entry typos, or deliberate point-of-sale fraud.
*   **Static Imputation Impact:** While filling empty regions with `"Unknown"` keeps rows available for global financial accounting, it reduces the predictive precision of localized regional planning.

---

## 10. Screenshots
*The following interactive dashboard captures from `outputs/pivot_summary.xlsx` demonstrate the sorted sales distribution and filtered carrier analysis:*

### Raw Dataset before Cleaning

### Cleaned Dataset with Calculated Columns

### Executive Monthly Sales Trends Dashboard

### Logistical Sale & Profit by Hierarchical Categories
