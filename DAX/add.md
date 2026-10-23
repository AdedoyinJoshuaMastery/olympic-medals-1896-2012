# Explanation — Discipline Medal Monopolization (≥ 60%)

This calculation is best created as a **Calculated Table** in Power BI. It generates a new table containing only the historical instances where a nation won **at least 60% of all medals (Gold + Silver + Bronze)** across an entire **Discipline** within a single Olympic edition (**Year**).

---

## How it works (step-by-step)

### 1) Handling team events correctly (`MedalUnits`)
The raw dataset is typically **athlete-level** (one row per athlete).  
If a country wins a **team medal**, counting rows would overcount that medal (e.g., 12 athletes could look like 12 medals).

To prevent this, the solution first builds an intermediate table called **`MedalUnits`** at a “one medal awarded” grain:

- Year  
- Sport  
- Discipline  
- Gender  
- Event  
- Medal  
- Country  

Why this works:
- A **team medal** is counted **once**, not once per athlete.
- If an event awards **two bronzes** (two countries), both bronzes are counted correctly because **`Medal + Country`** creates separate medal-units.

---

### 2) Establishing the baseline (`Base`)
Next, we create a baseline list of all unique combinations of:

- Year  
- Sport  
- Discipline  
- Country  

Each row now represents the question:

> “For this year and discipline, how many medals did this country win?”

---

### 3) Contextual counting (`WithCounts`)
For every row in `Base`, we compute two values using `COUNTROWS()` + `FILTER()`:

- **Country_Medals**: the number of medal-units won by that specific country in the discipline-year.
- **Total_Discipline_Medals**: the total number of medal-units awarded in that discipline-year across all countries.

This is the key comparison: **country performance vs total medals available**.

---

### 4) Calculating dominance (`Monopolization_Pct`)
We compute the dominance ratio:

**Monopolization_Pct = Country_Medals / Total_Discipline_Medals**

`DIVIDE()` is used instead of normal division to safely handle edge cases (like divide-by-zero).

---

### 5) Filtering to meaningful dominance (final filter logic)
Finally, we keep only rows that match the rules:

- **Total_Discipline_Medals ≥ 6**  
  This removes discipline-years with very small medal pools (where dominance could happen too easily).

- **Monopolization_Pct ≥ 0.60**  
  This ensures the country won **at least 60%** of all medals awarded in that discipline-year.

---

## Suggested visual (how to display the results)

Because this is a **calculated table**, you don’t need a special “formula visual.”

Recommended options:

1. **Data view (fast check)**  
   Go to **Data view** → select `Discipline_Monopolization_60pct` → review the filtered rows.

2. **Report view (best for sharing)**  
   Create a **Table** visual and add:
   - Year, Sport, Discipline, Country  
   - Country_Medals, Total_Discipline_Medals  
   - Monopolization_Pct (format as Percentage)

Optional improvements:
- Sort by **Monopolization_Pct (descending)**.
- Use conditional formatting to highlight the strongest dominance.

---

**MasterPiece Data Centre**