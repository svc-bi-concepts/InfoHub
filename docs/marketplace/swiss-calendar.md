# Swiss Holiday & Calendar Dataset Documentation

Welcome to the **Swiss Holiday & Calendar Dataset** documentation! This guide provides an in-depth look at how to leverage the dataset for efficient scheduling, regulatory compliance, operational planning, and more. Below, you'll find detailed descriptions, sample queries (complete with line numbers and syntax highlighting), and real-world use cases to help you get started.

---

## **Table of Contents**
1. [Overview](#overview)
2. [Dataset Structure](#dataset-structure)
3. [Sample Queries](#sample-queries)
   - [Query 1: All Public Holidays for a Specific Year](#query-1-all-public-holidays-for-a-specific-year)
   - [Query 2: Count Holidays per Year](#query-2-count-holidays-per-year)
   - [Query 3: Identify Weekdays with the Most Holidays](#query-3-identify-weekdays-with-the-most-holidays)
   - [Query 4: Find Consecutive Business Days](#query-4-find-consecutive-business-days)
   - [Query 5: Upcoming Holiday from Today](#query-5-upcoming-holiday-from-today)
   - [Query 6: Compare Weekend vs. Weekday Holidays](#query-6-compare-weekend-vs-weekday-holidays)
4. [Real-Life Use Cases](#real-life-use-cases)
5. [Conclusion](#conclusion)

---

## **Overview**

The **Swiss Holiday & Calendar Dataset** is a comprehensive collection of calendar data specifically tailored to Switzerland. It includes:

- **Public Holidays** (e.g., Swiss National Day, Berchtold's Day)  
- **Weekend Indicators** (Saturdays, Sundays)  
- **Business Day Flags** (identifying normal working days)  

By integrating this dataset into your workflows, you can:

- **Optimize Scheduling**: Plan staffing, deliveries, or maintenance around public holidays and weekends.  
- **Enhance Financial Processes**: Reconcile bank transactions and accounting data (e.g., with Bexio and UBS) by aligning with actual working days.  
- **Improve Compliance**: Align reporting and filings with official Swiss holidays to meet regulatory deadlines.  

---

## **Dataset Structure**

This dataset is exposed through a **secure view** named:

MARKETPLACE.CALENDAR_SWITZERLAND.CALENDAR_SWITZERLAND_W_NATIONAL_HOLIDAYS

It contains the following columns:

| **Column Name**    | **Data Type** | **Description**                                      |
|--------------------|---------------|------------------------------------------------------|
| `DATE`             | `DATE`        | The calendar date (YYYY-MM-DD)                      |
| `YEAR`             | `NUMBER`      | Year (e.g., 2023, 2024)                             |
| `MONTH`            | `NUMBER`      | Month of the year (1–12)                            |
| `DAY`              | `NUMBER`      | Day of the month (1–31)                             |
| `DAYOFWEEK`        | `VARCHAR`     | Name of the weekday (e.g., Monday, Tuesday)         |
| `ISSWISSHOLIDAY`   | `BOOLEAN`     | `TRUE` if the date is a recognized Swiss holiday     |
| `HOLIDAYNAME`      | `VARCHAR`     | Name of the holiday (if `ISSWISSHOLIDAY` is `TRUE`) |
| `ISWEEKEND`        | `BOOLEAN`     | `TRUE` if the date is a weekend (Saturday/Sunday)    |
| `ISWORKINGDAY`     | `BOOLEAN`     | `TRUE` if the date is a normal business day          |

---

## **Sample Queries**

Below are several SQL queries that showcase how to utilize the dataset. Each code block is annotated with line numbers and a descriptive title for clarity.

### **Query 1: All Public Holidays for a Specific Year**

```sql title="01_all_holidays_for_year.sql" linenums="1"
-- 1. Retrieve all Swiss public holidays for the year 2025
-- 2. Sort them in chronological order

SELECT
    DATE,
    DAYOFWEEK,
    HOLIDAYNAME
FROM MARKETPLACE.CALENDAR_SWITZERLAND.CALENDAR_SWITZERLAND_W_NATIONAL_HOLIDAYS
WHERE YEAR = 2025
  AND ISSWISSHOLIDAY = TRUE
ORDER BY DATE;
```

**Why It's Useful**
- Ideal for planning events or aligning workforce availability with public holidays in a given year.

---

### **Query 2: Count Holidays per Year**

```sql title="02_count_holidays_per_year.sql" linenums="1"
-- 1. Count the number of Swiss public holidays each year
-- 2. Group results by YEAR

SELECT
    YEAR,
    COUNT(*) AS HOLIDAY_COUNT
FROM MARKETPLACE.CALENDAR_SWITZERLAND.CALENDAR_SWITZERLAND_W_NATIONAL_HOLIDAYS
WHERE ISSWISSHOLIDAY = TRUE
GROUP BY YEAR
ORDER BY YEAR;
```

**Why It's Useful**
- Provides a quick reference for how many holidays occur in each year, useful for long-term planning and budgeting.

---

### **Query 3: Identify Weekdays with the Most Holidays**

```sql title="03_weekdays_with_most_holidays.sql" linenums="1"
-- 1. Determine which weekdays have the highest occurrence of Swiss public holidays
-- 2. Sort the results by frequency in descending order

SELECT
    DAYOFWEEK,
    COUNT(*) AS OCCURRENCES
FROM MARKETPLACE.CALENDAR_SWITZERLAND.CALENDAR_SWITZERLAND_W_NATIONAL_HOLIDAYS
WHERE ISSWISSHOLIDAY = TRUE
GROUP BY DAYOFWEEK
ORDER BY OCCURRENCES DESC;
```

**Why It's Useful**
- Helps identify trends (e.g., do most holidays fall on Mondays?), which can be relevant for scheduling and staffing policies.

---

### **Query 4: Find Consecutive Business Days**

```sql title="04_consecutive_business_days.sql" linenums="1"
-- 1. Use a CTE to detect "breaks" between consecutive dates
-- 2. Only consider days that are marked as working days
-- 3. Summation of streak breaks gives a "streak ID" for consecutive workdays

WITH consecutive_days AS (
    SELECT
        DATE,
        LAG(DATE) OVER (ORDER BY DATE) AS PREV_DATE,
        CASE
            WHEN DATEDIFF('day', LAG(DATE) OVER (ORDER BY DATE), DATE) = 1
                 AND ISWORKINGDAY
                 AND LAG(ISWORKINGDAY) OVER (ORDER BY DATE) = TRUE
            THEN 0
            ELSE 1
        END AS STREAK_BREAK
    FROM MARKETPLACE.CALENDAR_SWITZERLAND.CALENDAR_SWITZERLAND_W_NATIONAL_HOLIDAYS
    WHERE ISWORKINGDAY = TRUE
)
SELECT
    DATE,
    SUM(STREAK_BREAK) OVER (ORDER BY DATE ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS STREAK_ID
FROM consecutive_days
ORDER BY DATE;
```

**Why It's Useful**
- Identifies stretches of uninterrupted workdays, which is beneficial for planning project milestones or maintenance schedules.

---

### **Query 5: Upcoming Holiday from Today**

```sql title="05_upcoming_holiday.sql" linenums="1"
-- 1. Retrieve the next public holiday after today's date
-- 2. Limit to one result for a simple "next holiday" lookup

SELECT
    DATE,
    DAYOFWEEK,
    HOLIDAYNAME
FROM MARKETPLACE.CALENDAR_SWITZERLAND.CALENDAR_SWITZERLAND_W_NATIONAL_HOLIDAYS
WHERE ISSWISSHOLIDAY = TRUE
  AND DATE > CURRENT_DATE
ORDER BY DATE
LIMIT 1;
```

**Why It's Useful**
- Great for dashboards or applications needing to highlight the next holiday at a glance.

---

### **Query 6: Compare Weekend vs. Weekday Holidays**

```sql title="06_weekend_vs_weekday_holidays.sql" linenums="1"
-- 1. Determine how often holidays fall on weekends vs. weekdays
-- 2. Group by a CASE expression to label the day type

SELECT
    CASE WHEN ISWEEKEND THEN 'Weekend' ELSE 'Weekday' END AS HOLIDAY_TYPE,
    COUNT(*) AS HOLIDAY_COUNT
FROM MARKETPLACE.CALENDAR_SWITZERLAND.CALENDAR_SWITZERLAND_W_NATIONAL_HOLIDAYS
WHERE ISSWISSHOLIDAY = TRUE
GROUP BY HOLIDAY_TYPE
ORDER BY HOLIDAY_COUNT DESC;
```

**Why It's Useful**
- Helps understand how frequently holidays overlap with weekends, which can influence overtime policies and staffing needs.

---

## **Real-Life Use Cases**

Below are some real-world scenarios where the Swiss Holiday & Calendar Dataset proves invaluable:

1. **Financial Reconciliation**
    
    - Example: Reconciling Bexio accounting entries with UBS bank transactions, ensuring that transaction dates align with official business days.

2. **Demand Forecasting**

    - Example: A retail chain adjusts inventory before Swiss National Day, predicting a surge in sales due to holiday-related shopping.

3. **Compliance & Reporting**

    - Example: A payroll department schedules salary transfers around public holidays, ensuring employees are paid on time and regulatory filings are not delayed.


4. **Marketing Campaign Optimization**

    - Example: An e-commerce platform launches holiday-specific promotions to boost sales, targeting consumers before and during key Swiss holidays.

5. **Supply Chain Coordination**

       - Example: A logistics firm aligns deliveries to avoid holiday closures, reducing potential backlogs and ensuring smooth operations.

6. **Resource & Workforce Planning**

      - Example: A manufacturing plant uses holiday data to schedule maintenance on days when most workers are off, minimizing production downtime.

---

## **Conclusion**

By leveraging the Swiss Holiday & Calendar Dataset, businesses can:
- Enhance operational efficiency by accounting for non-working days.
- Ensure regulatory compliance through accurate holiday alignment.
- Optimize strategic decisions across forecasting, staffing, marketing, and more.

Whether you're planning large-scale projects or simply need to keep an eye on upcoming holidays, this dataset provides the timely, precise information you need to succeed in the Swiss market.

If you need more help in data modelling or data warehousing, contact us through our [website](https://www.bi-concepts.ch/contact){:target="_blank" rel="noopener"}.