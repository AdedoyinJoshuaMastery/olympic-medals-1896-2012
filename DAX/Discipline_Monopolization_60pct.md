# Discipline Medal Monopolization (≥ 60%) — Power BI DAX Solution

**MasterPiece Data Centre**

This markdown documents the **Power BI (DAX)** solution for identifying Olympic editions where a **single country won ≥ 60% of all medals** (Gold + Silver + Bronze combined) across an entire **Discipline** within the same Olympic edition (Year).

- **Dataset:** Summer Olympics medals (1896–2012)
- **Power BI source table:** `summer`
- **GitHub repo:** https://github.com/AdedoyinJoshuaMastery/olympic-medals-1896-2012.git

---

## 1) Problem Statement (Non‑Technical Explanation)

In every Olympics, each **discipline** (for example: *Diving*, *Fencing*, *Table Tennis*) contains multiple events, and each event awards medals.

This analysis asks:

> In a given Olympic year, did any country dominate a whole discipline by winning **at least 60% of all medals available** in that discipline?

This captures dominance across **many events**, not just winning one event.

---

## 2) Technical Definition (For Technical Readers)

For each combination of:

- **Year (Y)**
- **Sport**
- **Discipline (D)**
- **Country (C)**

Compute:

- `Country_Medals` = medals won by country C within discipline D in year Y  
- `Total_Discipline_Medals` = all medals awarded within discipline D in year Y  
- `Monopolization_Pct` = `Country_Medals / Total_Discipline_Medals`

### Filters / Edge Cases
Keep only rows where:

- `Total_Discipline_Medals >= 6`  
- `Monopolization_Pct >= 0.60`

### Output Columns
Return:

- `Year`
- `Sport`
- `Discipline`
- `Country`
- `Country_Medals`
- `Total_Discipline_Medals`
- `Monopolization_Pct`

---

## 3) Important Data Note (Why We Deduplicate Medals)

The `summer` dataset is commonly **athlete-level**.

That means a single medal (especially in team events) can appear multiple times—one row per athlete—so counting rows can **overcount medals**.

### What we do instead: count “medal-units”
We define 1 medal awarded using this unique grain:

**Year + Sport + Discipline + Gender + Event + Medal + Country**

This approach:
- counts team medals **once**
- correctly counts double-bronze situations as **two medal-units** (two countries)

---

## 4) DAX Implementation (Calculated Table)

Create this in **Power BI Desktop → Modeling → New table**.

**Calculated table name:** `Discipline_Monopolization_60pct`

```DAX
Discipline_Monopolization_60pct =
VAR MedalUnits =
    -- Deduplicate to "one medal awarded" (prevents team-event athlete duplication)
    SUMMARIZE (
        FILTER (
            summer,
            summer[Medal] IN { "Gold", "Silver", "Bronze" }
                && NOT ( ISBLANK ( summer[Country] ) )
                && NOT ( ISBLANK ( summer[Discipline] ) )
        ),
        summer[Year],
        summer[Sport],
        summer[Discipline],
        summer[Gender],
        summer[Event],
        summer[Medal],
        summer[Country]
    )
VAR Base =
    -- One row per (Year, Sport, Discipline, Country)
    SUMMARIZE ( MedalUnits, [Year], [Sport], [Discipline], [Country] )
VAR WithCounts =
    ADDCOLUMNS (
        Base,
        "Country_Medals",
            VAR y = [Year]
            VAR s = [Sport]
            VAR d = [Discipline]
            VAR c = [Country]
            RETURN
                COUNTROWS (
                    FILTER (
                        MedalUnits,
                        [Year] = y
                            && [Sport] = s
                            && [Discipline] = d
                            && [Country] = c
                    )
                ),
        "Total_Discipline_Medals",
            VAR y = [Year]
            VAR s = [Sport]
            VAR d = [Discipline]
            RETURN
                COUNTROWS (
                    FILTER (
                        MedalUnits,
                        [Year] = y
                            && [Sport] = s
                            && [Discipline] = d
                    )
                )
    )
VAR WithPct =
    ADDCOLUMNS (
        WithCounts,
        "Monopolization_Pct",
            DIVIDE ( [Country_Medals], [Total_Discipline_Medals] )
    )
RETURN
    SELECTCOLUMNS (
        FILTER (
            WithPct,
            [Total_Discipline_Medals] >= 6
                && [Monopolization_Pct] >= 0.60
        ),
        "Year", [Year],
        "Sport", [Sport],
        "Discipline", [Discipline],
        "Country", [Country],
        "Country_Medals", [Country_Medals],
        "Total_Discipline_Medals", [Total_Discipline_Medals],
        "Monopolization_Pct", [Monopolization_Pct]
    )

