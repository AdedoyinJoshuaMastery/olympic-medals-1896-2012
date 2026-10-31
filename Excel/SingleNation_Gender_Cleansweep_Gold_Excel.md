<!-- File: Excel/SingleNation_Gender_Cleansweep_Gold_Excel.md -->

# Single‑Nation Gender Cleansweep (Gold) — Excel Formula Replication

**MasterPiece Data Centre**  
Repo: https://github.com/AdedoyinJoshuaMastery/olympic-medals-1896-2012

This document shows how to replicate the **Single‑Nation Gender Cleansweep (Gold)** analysis in **Microsoft Excel** using formulas (Excel 365 / Excel 2021 recommended).

---

## 1) Problem Statement (Clear for Everyone)

In some Olympic editions, a country can dominate a sport. This analysis asks:

> Which Olympic editions had **one country** win **every Gold medal** in **both Men’s and Women’s events** of the **same Sport**?

In other words: within a given **Year + Sport**, the same country must own:
- **100% of Men’s Gold medals**, and
- **100% of Women’s Gold medals**

---

## 2) Dataset Used

**Source sheet/table:** `summer` (Excel Table)  
**Dataset columns:**
- Year
- City
- Sport
- Discipline
- Athlete
- Country
- Gender
- Event
- Medal

> Important: This dataset is often **athlete-level** (one row per athlete).  
> Team event medals can appear multiple times (one per athlete), so we must avoid double-counting.

---

## 3) Workbook Design (Sheets)

This approach mirrors the same logic used in Power BI (DAX), but implemented with Excel formulas.

Recommended sheets:

1. **`summer`** (raw dataset as an Excel Table named `summer`)
2. **`GoldEvents`** (event-level Gold winners to avoid double counting)
3. **`Base`** (Year–Sport–Country anchor table + counts + Cleansweep flag)
4. **`Result`** (final filtered output)

---

## 4) Step 0 — Prepare the `summer` Table

### 4.1 Convert the dataset to an Excel Table
- Select the dataset range → **Insert → Table**
- Ensure headers are recognized
- Rename the table to: **`summer`** (Table Design → Table Name)

### 4.2 Create an Event Key (prevents team overcounting)
Add a new column to `summer` called **`EventKey`**:

```excel
=[@Year]&"|"&[@Sport]&"|"&[@Discipline]&"|"&[@Gender]&"|"&[@Event]