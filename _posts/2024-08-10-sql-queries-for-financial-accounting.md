---
layout: post
title: "Top SQL Queries Every Financial Accountant Should Know"
categories: finance sql
---

<img src="{{ site.baseurl }}/assets/images/sql-finance.png" alt="SQL for Financial Accounting" style="width:100%; height:auto;" />

In today's data-driven world, financial accounting professionals are increasingly expected to work closely with large datasets and enterprise systems. Whether you’re extracting data for financial reports, performing reconciliations, or analyzing cash flow patterns, SQL (Structured Query Language) is a powerful tool that helps accountants gain direct access to financial data. 

In this post, we'll explore essential SQL queries that every financial accountant should know. We’ll cover everything from basic SELECT statements to more advanced queries for financial reporting, aged receivables, and budgeting.

---

## Why Financial Accountants Should Learn SQL

While accounting software like SAP, Oracle, or QuickBooks offers built-in reporting tools, they often come with limitations. SQL enables accountants to bypass user interfaces and pull exactly the data they need, when they need it.

Here are a few key benefits:
- Access raw, real-time financial data
- Create custom reports beyond software limitations
- Reconcile differences across modules or systems
- Audit and trace transactional changes with clarity

---

## 1. Retrieving Basic General Ledger Entries

Most financial databases include a `general_ledger` table or its equivalent. Here's a query to fetch recent GL entries:

```sql
SELECT gl_date, account_code, description, debit, credit
FROM general_ledger
WHERE gl_date BETWEEN '2024-01-01' AND '2024-03-31'
ORDER BY gl_date;
```
This helps accountants retrieve journal entries over a specific period. Add filters like account_code or cost_center for more precision.

## 2. Trial Balance Query
To generate a trial balance, group transactions by account and calculate balances.

```sql
SELECT account_code, 
       SUM(debit) AS total_debit, 
       SUM(credit) AS total_credit,
       SUM(debit - credit) AS balance
FROM general_ledger
GROUP BY account_code;
```
This allows you to quickly review whether debits and credits are balanced, a critical step in monthly close.

## 3. Aged Receivables Report
An aged receivables report helps you identify overdue customer balances. Assume you have an `invoices` table with `invoice_date` and `due_date` fields.

```sql
SELECT customer_id,
       SUM(CASE WHEN due_date >= CURRENT_DATE THEN amount ELSE 0 END) AS current,
       SUM(CASE WHEN due_date < CURRENT_DATE AND due_date >= CURRENT_DATE - INTERVAL '30 days' THEN amount ELSE 0 END) AS overdue_30,
       SUM(CASE WHEN due_date < CURRENT_DATE - INTERVAL '30 days' AND due_date >= CURRENT_DATE - INTERVAL '60 days' THEN amount ELSE 0 END) AS overdue_60,
       SUM(CASE WHEN due_date < CURRENT_DATE - INTERVAL '60 days' THEN amount ELSE 0 END) AS overdue_90_plus
FROM invoices
WHERE payment_status = 'Unpaid'
GROUP BY customer_id;
```
This aged breakdown helps AR teams prioritize collections and flag high-risk customers.

## 4. Vendor Payment History
Understanding vendor payment trends is crucial for managing cash flow and building good supplier relationships.

```sql
SELECT vendor_name, invoice_number, payment_date, amount_paid
FROM vendor_payments
WHERE payment_date BETWEEN '2024-01-01' AND '2024-06-30'
ORDER BY vendor_name, payment_date;
```
You can also calculate average days to pay using `DATEDIFF(payment_date, invoice_date)`.

## 5. Matching Invoices with Payments
Reconciling invoice payments is a common month-end task. This query helps link invoices and payments:

```sql
SELECT i.invoice_number, i.customer_id, i.amount AS invoice_amount, 
       p.payment_id, p.payment_date, p.amount AS paid_amount
FROM invoices i
LEFT JOIN payments p ON i.invoice_number = p.invoice_number
WHERE i.invoice_date BETWEEN '2024-01-01' AND '2024-03-31';
```
This query shows unmatched (open) invoices and matched payments for reconciliation.

## 6. Cost Center Expenditure
For companies that track expenses by cost center, this query can be used to monitor spending.

```sql
SELECT cost_center, account_code, 
       SUM(debit) AS total_expense
FROM general_ledger
WHERE gl_date BETWEEN '2024-01-01' AND '2024-06-30'
  AND account_code LIKE '5%' -- Assuming 5-series is expense accounts
GROUP BY cost_center, account_code;
```
Cost center reporting helps identify areas where expenses are rising or over budget.

## 7. Budget vs. Actual Comparison
Comparing budgeted values against actual expenditures is key in performance analysis. Assuming a `budget` table and `actuals` table:

```sql
SELECT b.account_code, b.cost_center, b.budget_amount, 
       a.actual_amount,
       (a.actual_amount - b.budget_amount) AS variance
FROM budget b
JOIN (
  SELECT account_code, cost_center, SUM(debit) AS actual_amount
  FROM general_ledger
  WHERE gl_date BETWEEN '2024-01-01' AND '2024-06-30'
  GROUP BY account_code, cost_center
) a ON b.account_code = a.account_code AND b.cost_center = a.cost_center;
```

This variance analysis is valuable for departmental budgeting and internal control.

## 8. Cash Flow Summary by Month
You can extract a high-level view of monthly cash inflows and outflows with:

```sql
SELECT DATE_TRUNC('month', transaction_date) AS month,
       SUM(CASE WHEN transaction_type = 'Inflow' THEN amount ELSE 0 END) AS inflow,
       SUM(CASE WHEN transaction_type = 'Outflow' THEN amount ELSE 0 END) AS outflow
FROM cash_transactions
WHERE transaction_date BETWEEN '2024-01-01' AND '2024-06-30'
GROUP BY DATE_TRUNC('month', transaction_date)
ORDER BY month;
```

Cash flow reporting is vital for liquidity planning and financing decisions.

## 9. Accrual vs. Cash Basis Comparison
Many accountants need to reconcile accrual accounting results with actual cash movements. You can query both views:

# Accrual-based:
```sql
SELECT invoice_date AS date, SUM(amount) AS revenue
FROM sales_invoices
GROUP BY invoice_date;
```
# Cash-based:
```sql
SELECT payment_date AS date, SUM(amount) AS cash_collected
FROM customer_payments
GROUP BY payment_date;
```
Visualizing both over time can help identify collection delays and revenue recognition gaps.

## 10. Monthly Expense Report

```sql
SELECT DATE_FORMAT(gl_date, '%Y-%m') AS month, 
       account_number,
       SUM(debit - credit) AS net_expense
FROM gl_entries
WHERE account_number BETWEEN '600000' AND '699999' -- Expense accounts
GROUP BY month, account_number
ORDER BY month, account_number;
```

## 11. Daily Cash Balance
```sql
SELECT gl_date, 
       SUM(debit - credit) AS daily_cash_change,
       SUM(SUM(debit - credit)) OVER (ORDER BY gl_date) AS cumulative_cash
FROM gl_entries
WHERE account_number = '100000' -- Cash account
GROUP BY gl_date
ORDER BY gl_date;
```
This calculates both the daily and cumulative change in your cash account, ideal for treasury operations.




