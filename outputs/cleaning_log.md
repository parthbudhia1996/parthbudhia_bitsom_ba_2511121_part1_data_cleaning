# Data Cleaning Log & Audit Trail

**Date of Execution:** June 24, 2026  
**Source Dataset:** raw_orders.xlsx  
**Output Dataset:** cleaned_orders.xlsx  

---

## 1. Executive Summary
This document provides a transparent, end-to-end accounting of data cleaning actions, business rule applications, anomalies handled, and record flagging states executed on the e-commerce orders dataset. 

The original messy data was parsed into three logical validation states:
* **Clean:** Fully valid records contributing to financial performance KPIs.
* **Warning:** Records with minor structural computation mismatches, adjusted or isolated for specific breakdowns.
* **Invalid:** Critical structural or physical impossibilities (e.g. negative time, negative prices) flagged and omitted from mainline financial reporting.

---

## 2. Issues Found & Business Rules Applied

| Rule Area | Phenomenon / Metric | Business Rule Applied | Record Action | Count |
| :--- | :--- | :--- | :--- | :--- |
| **Missing Geography** | 25 records with null `region` entries. | Fill missing values as `"Unknown"` and log to quality report. | Retained as `Warning` / Imputed | 25 |
| **Missing Logistics** | 20 records with null `ship_mode`. | Fill missing values as `"Unknown"` and log to quality report. | Retained as `Warning` / Imputed | 20 |
| **Missing Discounts** | 10 records with null `discount` values. | If quantity, price, and sales align assuming 0% discount, treat as `0`. Otherwise flag. | Imputed to `0` / Flagged as `Invalid` | 10 |
| **Out-of-Bounds Discount** | 14 negative discounts; 7 discounts above allowable thresholds (>50/100%). | Flag as structurally corrupted data. | Flagged as `Invalid` | 21 |
| **Temporal Paradox** | 17 records where `ship_date` occurs before `order_date`. | Flag as physically impossible logistical entry. | Flagged as `Invalid` | 17 |
| **Logical Status Conflict** | 37 records where payment failed (`Failed`) but order status was marked `Completed` or `Returned`. | Flag as status mismatches ("If payment fails, order should be cancelled"). | Flagged as `Invalid` | 37 |
| **Financial Mismatches** | 51 Sales mismatches; 54 Profit mismatches against formulaic calculations. | Recompute fields using: `Sales = Quantity * Unit_Price * (1 - Discount)` and `Profit = Sales - Cost`. Flag anomalies. | Flagged as `Warning` | 105 |
| **Transaction Drops** | Cancelled orders, Failed payments, and Refunded transactions. | Isolate from core finalized sales metrics. Move Refunds to dedicated evaluation tracker. | Segregated / Omitted from Final Summaries | Varied |

---

## 3. Cleaning Actions Performed & Data Transformations
1.  **Deduplication:** Identified 20 exact row duplicates via whole-record multi-column hashing. Removed all 20 identical copies. Retained 31 conflicting duplicate `order_id`s for manual review to prevent silent data loss.
2.  **Date Standardization:** Converted 75 textual date strings into standardized ISO formats (`YYYY-MM-DD`). 
3.  **Field Engineering:** * Created `cleaned_discount` to normalize raw percentages into numeric decimals.
    * Created `calculated_sales` and `calculated_profit` to benchmark integrity.
    * Created `shipping_delay_days` ("Ship\_Date - Order\_Date").
    * Derived calendar groupings: `order_month` and `order_year`.

---

## 4. Final Record Accounting Summary
* **Total Records Scanned:** 696
* **Clean Records:** 621 (Used for primary sales metrics)
* **Warning Records:** 42 (Imputed/Calculated mismatches safely bracketed)
* **Invalid Records:** 33 (Permanently excluded from pipeline totals)
* **Records Completely Dropped:** 20 (Exact duplicate lines)

**Note:** Not included non-completed/failed/refunded records in completed-sales summaries.

---

## 5. Systemic Assumptions Made
* **Revenue Source of Truth:** Where calculating fields differed negligibly from raw recorded revenue fields ($0.01 variance), raw data was maintained. Where severe variance occurred, the engineered arithmetic formulas were treated as correct.
* **Financial Scope:** Any transaction possessing an `order_status` of "Cancelled" or a `payment_status` of "Failed" was treated as zero net realized revenue for the organization.

---

## 6. Pipeline Limitations
* **Root Cause Obscurity:** The pipeline flags operational data mismatches (e.g., failed payment items marked as "Completed"), but cannot retroactively distinguish if this indicates system software bugs or intentional customer fraud.
* **Static Imputation:** Flagging a regional field as "Unknown" allows formulas to compile but weakens local targeted analytical breakdowns.