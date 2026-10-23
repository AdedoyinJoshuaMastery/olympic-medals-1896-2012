# 🧮 DAX Solutions — Summer Olympics 1896–2012

A collection of worked DAX answers to analytical questions based on the
Summer Olympics medalists dataset (`data/summer.csv`).

Each entry follows a standard structure:
- **Analytical Task** — the business question and technical constraints.
- **DAX Solution** — the exact code required.
- **Explanation** — a breakdown of how the DAX handles the logic and edge cases.
- **Suggested Visual** — how to display the result in Power BI.

> 💡 **Assumption:** The dataset is loaded into a table named `summer`. 
> If you renamed it during import, replace `summer` with your table name throughout.

---

## 📚 Table of Contents

1. [Single-Nation Multi-Event Gender Cleansweep](#1-single-nation-multi-event-gender-cleansweep)

---

## 1. Single-Nation Multi-Event Gender Cleansweep

### Analytical Task
Identify all Olympic editions where a single country won **every Gold medal** in *both* the Male and Female divisions of the same `Sport`.

**Technical Specifications & Edge Cases:**
- The sport must have had at least **2 Male events** and at least **2 Female events** in that edition.
- Requires a nation to hold **100% of Male Gold medals** AND **100% of Female Gold medals** in that sport for that year.
- **Output Standard:** Display `Year`, `Sport`, `Country`, `Male_Gold_Count`, `Female_Gold_Count`, and `Total_Sport_Gold_Count`.

### DAX Solution

```dax
Gender_Cleansweep_Gold = 
// Step 1: Isolate only Gold medals for Men and Women, and group by Event
// to ensure we handle team events correctly (1 gold per event, not per athlete).
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

// Step 2: Create a base table of the unique Year, Sport, and Country combinations
VAR Base =
    SUMMARIZE (
        GoldByEvent,
        [Year],
        [Sport],
        [GoldCountry]
    )

// Step 3: Add counts for the specific country and the overall sport totals
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

// Step 4: Filter for the conditions and output the final columns
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
```

### Explanation

This calculation is best created as a **Calculated Table** in Power BI, which will generate a new table showing only the specific historical instances where a nation completely dominated both genders in a single sport.

Here is how it works step-by-step:

1. **Handling Team Events (`GoldByEvent`):** The raw dataset contains one row per athlete. If a country wins a team event (e.g., Basketball), counting rows would yield 12 gold medals instead of 1. The `SUMMARIZE` function groups by `Event` and takes the `MAX` country to ensure we count exactly **one Gold per Event**.
2. **Establishing the Baseline (`Base`):** We create a list of all unique combinations of `Year`, `Sport`, and `GoldCountry` to serve as our anchor points.
3. **Contextual Counting (`WithCounts`):** For every row in our `Base`, we calculate two things using `COUNTROWS` and `FILTER`:
   - How many golds **this specific country** won (Male and Female).
   - How many golds were available **in total** for that sport in that year (Male and Female).
4. **Filtering the Cleansweep (`FILTER`):** 
   - `[Total_Male_Gold_Count] >= 2` and `[Total_Female_Gold_Count] >= 2`: Ensures the sport actually had multiple events for both genders that year, satisfying the technical spec.
   - `[Male_Gold_Count] = [Total_Male_Gold_Count]`: Ensures the country won 100% of the men's golds.
   - `[Female_Gold_Count] = [Total_Female_Gold_Count]`: Ensures the country won 100% of the women's golds.

**Suggested visual:** 
Because this is a calculated table, you don't need a visual formula. Simply go to the Data view, select the new `Gender_Cleansweep_Gold` table, and it will display the exact rows that meet all conditions. You can then drag the columns into a Power BI **Table visual** to present the findings.
