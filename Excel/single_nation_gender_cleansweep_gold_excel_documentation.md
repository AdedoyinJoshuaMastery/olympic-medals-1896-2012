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
```

This produces one identifier per Olympic event (Year + Sport + Discipline + Gender + Event).

---

## 5) Step 1 — Build GoldEvents (Event‑level Gold winners)

### Purpose (non‑technical)
The raw data has one row per athlete. A team Gold medal can appear many times.
So we build a clean table where each row = one Gold medal for one event.

### 5.1 GoldEvents layout
Create headers in `GoldEvents!A1:G1`:

Year | Sport | Discipline | Gender | Event | EventKey | Country

### 5.2 Spill formula for the event-level Gold list
In `GoldEvents!A2`, spill the Gold events:

Using CHOOSE (preferred in this workbook):

```excel
=UNIQUE(
  FILTER(
    CHOOSE(
      {1,2,3,4,5,6},
      summer[Year],
      summer[Sport],
      summer[Discipline],
      summer[Gender],
      summer[Event],
      summer[EventKey]
    ),
    summer[Medal]="Gold"
  )
)
```

### 5.3 Country column (XLOOKUP approach)
In `GoldEvents!G2`, return the Country for each EventKey (this uses the corrected spill reference):

```excel
=XLOOKUP(
  INDEX($A$2#,,6),
  FILTER(summer[EventKey], summer[Medal]="Gold"),
  FILTER(summer[Country],  summer[Medal]="Gold"),
  "Not found",
  0
)
```

**Note: UNIQUE alternative to XLOOKUP (optional)**  
Instead of XLOOKUP, you can include `summer[Country]` directly inside the UNIQUE spill (no lookup needed).  
This is a valid alternative because duplicate athlete rows share the same Country.

---

## 6) Step 2 — Build the Base sheet (Year–Sport–Country + Counts)

### 6.1 Base headers
Create headers in `Base!A1:I1`:

Year | Sport | Country | Male_Gold_Count | Female_Gold_Count | Total_Male_Gold | Total_Female_Gold | Total_Sport_Gold_Count | Cleansweep

### 6.2 Base anchor spill (Year–Sport–Country)
In `Base!A2`, spill the unique Year–Sport–Country combinations (CHOOSE version):

```excel
=LET(
  ev, GoldEvents!$A$2#,
  n,  ROWS(ev),
  UNIQUE(
    CHOOSE(
      {1,2,3},
      INDEX(ev,,1),
      INDEX(ev,,2),
      INDEX(GoldEvents!$G$2#, SEQUENCE(n))
    )
  )
)
```

---

## 7) Step 3 — Counts (Male/Female + Totals) in Base

These formulas use the GoldEvents spill ranges to compute correct counts.

### 7.1 Male_Gold_Count (`Base!D2`)
```excel
=IF($A2="","",
  LET(
    ev, GoldEvents!$A$2#,
    yr, INDEX(ev,,1),
    sp, INDEX(ev,,2),
    ge, INDEX(ev,,4),
    cn, INDEX(GoldEvents!$G$2#, SEQUENCE(ROWS(ev))),
    COUNTIFS(yr,$A2, sp,$B2, ge,"Men", cn,$C2)
  )
)
```

### 7.2 Female_Gold_Count (`Base!E2`)
```excel
=IF($A2="","",
  LET(
    ev, GoldEvents!$A$2#,
    yr, INDEX(ev,,1),
    sp, INDEX(ev,,2),
    ge, INDEX(ev,,4),
    cn, INDEX(GoldEvents!$G$2#, SEQUENCE(ROWS(ev))),
    COUNTIFS(yr,$A2, sp,$B2, ge,"Women", cn,$C2)
  )
)
```

### 7.3 Total_Male_Gold (`Base!F2`)
```excel
=IF($A2="","",
  LET(
    ev, GoldEvents!$A$2#,
    yr, INDEX(ev,,1),
    sp, INDEX(ev,,2),
    ge, INDEX(ev,,4),
    COUNTIFS(yr,$A2, sp,$B2, ge,"Men")
  )
)
```

### 7.4 Total_Female_Gold (`Base!G2`)
```excel
=IF($A2="","",
  LET(
    ev, GoldEvents!$A$2#,
    yr, INDEX(ev,,1),
    sp, INDEX(ev,,2),
    ge, INDEX(ev,,4),
    COUNTIFS(yr,$A2, sp,$B2, ge,"Women")
  )
)
```

### 7.5 Total_Sport_Gold_Count (`Base!H2`)
```excel
=IF($A2="","",
  LET(
    ev, GoldEvents!$A$2#,
    yr, INDEX(ev,,1),
    sp, INDEX(ev,,2),
    COUNTIFS(yr,$A2, sp,$B2)
  )
)
```

---

## 8) Step 4 — Cleansweep Flag (`Base!I2`)

A cleansweep is TRUE only when:
- the sport-year has at least 2 Men events and 2 Women events, and
- the country won all Men Golds and all Women Golds

Use this corrected flag formula (forces numeric comparison):

```excel
=IF($A2="","",
  LET(
    m, --$D2,
    f, --$E2,
    tm, --$F2,
    tf, --$G2,
    AND(tm>=2, tf>=2, m=tm, f=tf)
  )
)
```

---

## 9) Step 5 — Final Output in Result

### 9.1 Result headers
In `Result!A1:F1`:

Year | Sport | Country | Male_Gold_Count | Female_Gold_Count | Total_Sport_Gold_Count

### 9.2 Result spill formula (filter cleansweeps only)
In `Result!A2`:

```excel
=LET(
  y, Base!$A$2:$A$5000,
  s, Base!$B$2:$B$5000,
  c, Base!$C$2:$C$5000,
  m, Base!$D$2:$D$5000,
  f, Base!$E$2:$E$5000,
  t, Base!$H$2:$H$5000,
  k, Base!$I$2:$I$5000,
  out, FILTER(CHOOSE({1,2,3,4,5,6}, y,s,c,m,f,t), k=TRUE),
  SORT(out, {1,2}, {1,1})
)
```

This produces the final Year–Sport–Country cleansweep output (same logic as the Power BI DAX solution).

---

## 10) Notes / Tips

- If you get `#SPILL!`, ensure the spill output area is empty.
- If Gender values don’t match exactly (e.g., trailing spaces), use `TRIM()` on Gender in the raw table or use wildcards like `"Women*"` in `COUNTIFS`.
- This workbook intentionally mirrors the Power BI approach:
  - event-level deduplication
  - anchor base table
  - gender counts vs totals
  - final filter for 100% dominance across both genders

---

**MasterPiece Data Centre**
