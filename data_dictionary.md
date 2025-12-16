# DATA_DICTIONARY.txt

Project: Billing & Revenue Analytics – Accounts Receivable Case Study  
Period: Jul 2023 – Jun 2025  
Currency: USD  
Data: Synthetic (modeled to reflect realistic billing, payment delays, and disputes)

---

## Table: customers.csv

**Primary Key:** Customer_ID

| Column Name | Type | Example | Description |
|---|---:|---|---|
| Customer_ID | Text | CUST-0012 | Unique customer identifier. |
| Customer_Name | Text | Company 12 | Customer name used in reporting. |
| Segment | Text | Enterprise | Customer segment (e.g., SMB, Mid-Market, Enterprise). |
| Country | Text | United Kingdom | Customer country. |
| Industry | Text | SaaS | Customer industry category. |
| Signup_Date | Date | 2023-08-15 | Date customer relationship started (used for context). |
| Is_Active | True/False | TRUE | Whether customer is active during the period. |

---

## Table: invoices.csv

**Primary Key:** Invoice_ID  
**Foreign Key:** Customer_ID → customers.csv

| Column Name | Type | Example | Description |
|---|---:|---|---|
| Invoice_ID | Text | INV-000245 | Unique invoice identifier. |
| Customer_ID | Text | CUST-0012 | Customer identifier (joins to customers). |
| Invoice_Date | Date | 2024-11-03 | Date the invoice was issued. |
| Due_Date | Date | 2024-12-03 | Invoice due date (used for aging). |
| Invoice_Amount_USD | Decimal | 1250.00 | Total invoice amount in USD. |
| Currency | Text | USD | Invoice currency (USD for this project). |
| Invoice_Status | Text | Open | Invoice status (e.g., Open, Paid, Partial, Cancelled). |
| Payment_Terms_Days | Integer | 30 | Contracted payment terms in days (e.g., Net 30). |
| Product_Plan | Text | Pro | Product or service plan/category billed. |
| Region | Text | EMEA | Customer region label (optional grouping field). |

**Derived fields created in Power BI (Invoices table):**
- DaysPastDue (calculated column)  
- Aging Bucket (calculated column)  
- Aging Bucket Sort (calculated column)

---

## Table: payments.csv

**Primary Key:** Payment_ID  
**Foreign Key:** Invoice_ID → invoices.csv

| Column Name | Type | Example | Description |
|---|---:|---|---|
| Payment_ID | Text | PAY-000901 | Unique payment identifier. |
| Invoice_ID | Text | INV-000245 | Invoice identifier (joins to invoices). |
| Payment_Date | Date | 2025-01-12 | Date cash was received (used for cash collection trend). |
| Payment_Amount_USD | Decimal | 500.00 | Payment amount in USD. |
| Payment_Method | Text | Bank Transfer | Payment method (e.g., Bank Transfer, Card). |
| Payment_Status | Text | Completed | Payment status (e.g., Completed, Failed, Refunded). |
| Reference | Text | REMIT-34921 | Optional remittance/reference field. |

---

## Table: disputes.csv

**Primary Key:** Dispute_ID  
**Foreign Keys:** Invoice_ID → invoices.csv, Customer_ID → customers.csv

| Column Name | Type | Example | Description |
|---|---:|---|---|
| Dispute_ID | Text | DSP-00077 | Unique dispute identifier. |
| Invoice_ID | Text | INV-000245 | Invoice linked to dispute. |
| Customer_ID | Text | CUST-0012 | Customer linked to dispute. |
| Dispute_Open_Date | Date | 2025-02-03 | Date dispute was opened. |
| Dispute_Close_Date | Date | 2025-03-05 | Date dispute was closed (blank if open). |
| Dispute_Status | Text | Open | Dispute status (Open or Closed). |
| Dispute_Reason | Text | Pricing | Reason category (e.g., Pricing, Contract, Service). |
| Dispute_Notes | Text | Price mismatch | Optional short notes describing the issue. |
| Disputed_Amount_USD | Decimal | 250.00 | Amount under dispute (USD). |

**Derived fields created in Power BI (Disputes table):**
- Resolution Days (calculated column)

---

## Table: Date (created in Power BI)

**Primary Key:** Date[Date]  
Used as a shared calendar table for time-series analysis across invoices and payments.

| Column Name | Type | Example | Description |
|---|---:|---|---|
| Date | Date | 2024-09-01 | Daily date grain. |
| Year | Integer | 2024 | Calendar year. |
| MonthNumber | Integer | 9 | Month number (1–12). |
| MonthName | Text | Sep | Month name (MMM). |
| YearMonth | Text | 2024-09 | Year-month label (YYYY-MM). |
| YearMonthSort | Integer | 202409 | Sort key for YearMonth. |
| Quarter | Text | Q3 | Calendar quarter label. |

Note: The Date table range was expanded to include both invoice and payment dates to prevent missing months in trends.

---

## Relationships (Model)

- customers[Customer_ID] (1) → invoices[Customer_ID] (many)
- invoices[Invoice_ID] (1) → payments[Invoice_ID] (many)
- invoices[Invoice_ID] (1) → disputes[Invoice_ID] (many)
- customers[Customer_ID] (1) → disputes[Customer_ID] (many)
- Date[Date] (1) → invoices[Invoice_Date] (many) . active
- Date[Date] (1) → payments[Payment_Date] (many) . inactive (activated in measures via USERELATIONSHIP)

---

## Key Measures and Logic (High Level)

- Total Invoiced = sum of invoice amounts  
- Total Collected = sum of payment amounts  
- Collection Rate = Total Collected / Total Invoiced  
- Invoice Balance (Non-Negative) = max(0, invoice amount - payments)  
- Total AR = sum of non-negative invoice balances  
- Aging logic based on DaysPastDue = Due_Date vs fixed as-of date (2025-06-30)  
- Dispute metrics: Open Disputes, Avg Resolution Days

