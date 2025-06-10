# 📊 Manpower Budgeting in Excel (India Division)

## Overview

This Excel-based tool was built to automate and streamline the **Manpower Budgeting Process** for HR and Finance teams. It supports dynamic calculations of active employment periods, pro-rata salary adjustments, and conditional salary increases based on business rules.

Developed with Excel `LET()`, `IFS()`, `ROUND()`, and date functions for **precision and scalability**, this solution reduces manual work while ensuring accurate cost forecasting per employee.

---

## 🔧 Key Features

- **Automatic Month Calculations**  
  Calculates the number of active months in the budget year, accounting for:
  - Hiring date
  - Retirement date
  - Pro-rata adjustments
  - Diwali-based eligibility bonuses

- **Salary Increase Logic**  
  Incorporates logic to:
  - Apply salary increases only if the employee has completed 12 months of service by 1st July 2025.
  - Apply increase only to months **after** July 2025.

- **Two-Tier Monthly Calculation**  
  - Pre-increase months use original monthly salary.
  - Post-increase months apply uplifted rate.

- **Contribution Calculation**  
  Based on monthly contribution (cell `O5`) with same logic as salary.

---

## 📘 Formula Snippet: Conditional Salary Increase

```excel
=ROUND(
  IF(
    AND(F5 <= $E$3; OR(ISBLANK(G5); G5 > DATE(YEAR($E$3)+1; MONTH($E$3); DAY($E$3))));
    (MIN(M5; 6) * O5) + (MAX(M5 - 6; 0) * O5 * (1 + $E$2));
    M5 * O5
  );
2)
