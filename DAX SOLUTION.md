# 🧮 DAX Solutions — Summer Olympics 1896–2012

A collection of worked DAX answers to complex analytical questions on the Summer Olympics medalists dataset (`data/summer.csv`).

> 💡 **Assumption:** The table is loaded as `summer` (the default name when you import `data/summer.csv` into Power BI). If you renamed it, replace `summer` with your table name throughout.

---

## 📚 Table of Contents

1. [Single-Nation Multi-Event Gender Cleansweep](#1-single-nation-multi-event-gender-cleansweep)

---

## 1. Single-Nation Multi-Event Gender Cleansweep

### ❓ Question
Identify all Olympic editions where a single country won **every Gold medal** in both the Male and Female divisions of the same `Sport`.

### 📋 Technical Specifications & Edge Cases
- The sport must have had **at least 2 Male events** and **at least 2 Female events** in that edition.
- Requires Nation `C` to hold **100%** of Male Gold medals AND **100%** of Female Gold medals in that sport for year `Y`.
- **Output Standard:** Display `Year`, `Sport`, `Country`, `Male_Gold_Count`, `Female_Gold_Count`, and `Total_Sport_Gold_Count`.

### 💻 DAX Solution

Create a new **Calculated Table** using the following DAX:

```dax
Gender_Cleansweep_Gold = 
VAR GoldByEvent =
    SUMMARIZE (
        FILTER (
            summer,
            summer[Medal] = "Gold"
                && summer[Gender] IN { "Men", "Women" }
        ),
        summer[Year],
        summer[Sport],
        summer[Discipline],
        summer[Gender],
        summer[Event],
        "GoldCountry", MAX ( summer[Country] )
    )
VAR Base =
    SUMMARIZE (
        GoldByEvent,
        [Year],
        [Sport],
        [GoldCountry]
    )
VAR WithCounts =
    ADDCOLUMNS (
        Base,
        "Male_Gold_Count",
            VAR y = [Year]
            VAR s = [Sport]
            VAR c = [GoldCountry]
            RETURN
                COUNTROWS (
                    FILTER (
                        GoldByEvent,
                        [Year] = y
                            && [Sport] = s
                            && [Gender] = "Men"
                            && [GoldCountry] = c
                    )
                ),
        "Female_Gold_Count",
            VAR y = [Year]
            VAR s = [Sport]
            VAR c = [GoldCountry]
            RETURN
                COUNTROWS (
                    FILTER (
                        GoldByEvent,
                        [Year] = y
                            && [Sport] = s
                            && [Gender] = "Women"
                            && [GoldCountry] = c
                    )
                ),
        "Total_Male_Gold_Count",
            VAR y = [Year]
            VAR s = [Sport]
            RETURN
                COUNTROWS (
                    FILTER (
                        GoldByEvent,
                        [Year] = y
                            && [Sport] = s
                            && [Gender] = "Men"
                    )
                ),
        "Total_Female_Gold_Count",
            VAR y = [Year]
            VAR s = [Sport]
            RETURN
                COUNTROWS (
                    FILTER (
                        GoldByEvent,
                        [Year] = y
                            && [Sport] = s
                            && [Gender] = "Women"
                    )
                ),
        "Total_Sport_Gold_Count",
            VAR y = [Year]
            VAR s = [Sport]
            RETURN
                COUNTROWS (
                    FILTER (
                        GoldByEvent,
                        [Year] = y
                            && [Sport] = s
                    )
                )
    )
RETURN
    SELECTCOLUMNS (
        FILTER (
            WithCounts,
            [Total_Male_Gold_Count] >= 2
                && [Total_Female_Gold_Count] >= 2
                && [Male_Gold_Count] = [Total_Male_Gold_Count]
                && [Female_Gold_Count] = [Total_Female_Gold_Count]
        ),
        "Year", [Year],
        "Sport", [Sport],
        "Country", [GoldCountry],
        "Male_Gold_Count", [Male_Gold_Count],
        "Female_Gold_Count", [Female_Gold_Count],
        "Total_Sport_Gold_Count", [Total_Sport_Gold_Count]
    )