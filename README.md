# 📊 Manpower Budgeting in Excel

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

# Active Months Calculation

This formula computes the number of months an employee is active in the budget year, handling:

- Hiring before/after the budget year  
- Retirement mid-year  
- Diwali eligibility adjustments  
- Pro-rata calculations
```excel
=LET(
  HiringDate;      F5;
  RetireDate;      G5;
  prodate;         $E$3;
  PrevYear;        YEAR(prodate) - 1;
  CurrYear;        YEAR(prodate);
  BudgetYear;      YEAR(prodate) + 1;
  YearH;           YEAR(HiringDate);
  YearR;           YEAR(RetireDate);
  MonthThresh;     10;
  Divisor;         30,5;
  StartCurrYear;   DATE(CurrYear; 1; 1);
  EndCurYear;      DATE(CurrYear; 12; 31);
  diwally;         DATE(CurrYear; 10; 1);

  BaseMonths;      (EndCurYear - prodate) / Divisor;
  RetireMonths;    (RetireDate - prodate) / Divisor;
  HireMonths;      (EndCurYear - HiringDate) / Divisor;
  RetireHiring;    (RetireDate - HiringDate) / Divisor;
  DiwallyAdj;      ((diwally - HiringDate) / Divisor) / 12;
  DiwaliCheck;     (diwally - HiringDate) / Divisor > 11,9;

  NoRetire;
    IF(
      YearH = BudgetYear;
      0;
    IF(
      YearH < PrevYear;
      BaseMonths + 1;
    IF(
      YearH = PrevYear;
      IF(DiwaliCheck; BaseMonths + 1; BaseMonths + DiwallyAdj);
    IF(
      YearH = CurrYear;
      IF(HiringDate > prodate;
        IF(MONTH(HiringDate) > MonthThresh;
           HireMonths;
           HireMonths + DiwallyAdj
      );IF(MONTH(HiringDate) > MonthThresh;
          BaseMonths;
          BaseMonths + DiwallyAdj)
      );0
    ))));

  RetireCalc;
  IFS(
    AND(YearH <= CurrYear; RetireDate <= prodate); 0;

    AND(YearH < PrevYear; YearR = CurrYear);
      IF(MONTH(RetireDate) > MonthThresh; RetireMonths + 1; RetireMonths);

    AND(YearH = PrevYear; YearR = CurrYear);
      IF(MONTH(RetireDate) > MonthThresh;
         IF(DiwaliCheck; RetireMonths + 1; RetireMonths + DiwallyAdj);
         RetireMonths
      );

    AND(YearH = CurrYear; YearR = CurrYear);
      IF(
        HiringDate <= prodate;
        IF(AND(MONTH(RetireDate) > MonthThresh; MONTH(HiringDate) < MonthThresh);
           RetireMonths + DiwallyAdj;
           RetireMonths
        );
        IF(AND(MONTH(RetireDate) > MonthThresh; MONTH(HiringDate) < MonthThresh);
           RetireHiring + DiwallyAdj;
           RetireHiring
        )
      );

    AND(YearH = CurrYear; YearR = BudgetYear);
      IF(
        HiringDate <= prodate;
        IF(MONTH(HiringDate) < MonthThresh;
           BaseMonths + DiwallyAdj;
           BaseMonths
        );
        IF(MONTH(HiringDate) < MonthThresh;
           HireMonths + DiwallyAdj;
           HireMonths
        )
      );

    AND(YearH <= PrevYear; YearR = BudgetYear);
      IF(DiwaliCheck;
           BaseMonths + 1;
           BaseMonths + DiwallyAdj);

    TRUE; NA()
  );

  Result;
    IF(
      YearH = BudgetYear;
      0;
      IF(NOT(ISBLANK(RetireDate)); RetireCalc; NoRetire)
    );

  Result
)

---


# Conditional Salary Increase

This formula applies a mid-year salary increase only for the portion of active months **after** July 2025, given eligibility:

```excel
=ROUND(
  IF(
    AND(
      F5 <= EDATE($E$3; -12);
      OR(ISBLANK(G5); G5 > DATE(YEAR($E$3)+1; MONTH($E$3); DAY($E$3)))
    );
    (MIN(M5; 6) * O5) + (MAX(M5 - 6; 0) * O5 * (1 + $E$2));
    M5 * O5
  );
2)
