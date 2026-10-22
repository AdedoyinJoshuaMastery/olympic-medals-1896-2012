# 🏅 Summer Olympics Medalists Dataset (1896 – 2012)

A comprehensive, clean, and analysis-ready dataset of every medal-winning athlete
across the Summer Olympic Games from Athens 1896 to London 2012.

This dataset is designed for sports analytics, historical exploration, and
Power BI dashboards. Use it to analyse medal counts, country performance,
athlete achievements, sport trends, and more.

---

## 📁 Repository Structure

```
olympic-medals-1896-2012/
├── README.md
└── data/
    └── summer.csv
```

---

## 📌 What's Inside

The file `data/summer.csv` contains **one row per medal awarded** — meaning an
athlete who wins two medals appears in two rows, and every medal in a team event
is listed for each team member.

- **Editions covered:** 1896 – 2012 (Summer Games only)
- **Cities:** 27 host cities
- **Sports:** Aquatics, Athletics, Fencing, Cycling, Gymnastics, Football,
  Hockey, Rowing, Sailing, Shooting, Tennis, Wrestling, and many more (including
  discontinued sports such as Basque Pelota, Cricket, Croquet, Jeu de Paume,
  Lacrosse, Polo, Rackets, Roque, Rugby, Tug of War, and Water Motorsports)
- **Grain:** One row per athlete per medal

---

## 🧾 Columns

| Column       | Type    | Description                                                       | Example          |
|--------------|---------|-------------------------------------------------------------------|------------------|
| `Year`       | Integer | Year the Games were held                                         | `1896`           |
| `City`       | Text    | Host city of the Games                                           | `Athens`         |
| `Sport`      | Text    | Broad sport category                                             | `Aquatics`       |
| `Discipline` | Text    | Sub-discipline within the sport                                  | `Swimming`       |
| `Athlete`    | Text    | Athlete name (surname first, uppercase surname by convention)    | `HAJOS Alfred`   |
| `Country`    | Text    | IOC 3-letter country code of the athlete/team                    | `HUN`            |
| `Gender`     | Text    | `Men` or `Women`                                                 | `Men`            |
| `Event`      | Text    | Specific event name                                              | `100M Freestyle` |
| `Medal`      | Text    | `Gold`, `Silver`, or `Bronze`                                    | `Gold`           |

### Notes on the data

- **Country codes** are IOC codes (e.g. `URS` for Soviet Union, `GDR` for East
  Germany, `ZZX` for mixed teams, `EUA` for the Unified Team of Germany,
  `EUN` for the Unified Team, `ANZ` for Australasia).
- **Team events** are exploded so that each member has their own row.
- **Mixed/combined teams** (e.g. early tennis doubles) are coded `ZZX`.
- **Name formatting** follows the historical source convention
  (`SURNAME Firstname`), which makes parsing into separate first/last name
  columns straightforward in Power BI.
- Some historical events are no longer contested (e.g. *Tug of War*,
  *100M Freestyle For Sailors*, *Plunge For Distance*).

---

## 📊 Suggested Analyses

- Total medal table by country and by Games edition
- Medal trends over time by sport or discipline
- Top athletes by number of medals
- Gender participation over the decades
- Host-nation performance ("home advantage")
- Discontinued sports and events
- Drill-through from Country → Sport → Discipline → Event → Athlete

---

## 🔌 Loading into Power BI

### Option 1 — Import the CSV directly

1. Open **Power BI Desktop** → `Home` → **Get Data** → **Text/CSV**.
2. Select `data/summer.csv`.
3. In the preview window, verify:
   - **Delimiter:** Comma
   - **File Origin:** 65001: Unicode (UTF-8)
4. Click **Transform Data** (recommended) or **Load**.

### Option 2 — Power Query (recommended cleanup steps)

In the Power Query Editor, apply these steps:

```m
let
    Source = Csv.Document(
        File.Contents("data/summer.csv"),
        [Delimiter = ",", Columns = 9, Encoding = 65001, QuoteStyle = QuoteStyle.Csv]
    ),
    PromotedHeaders = Table.PromoteHeaders(Source, [PromoteAllScalars = true]),
    ChangedTypes = Table.TransformColumnTypes(PromotedHeaders, {
        {"Year", Int64.Type},
        {"City", type text},
        {"Sport", type text},
        {"Discipline", type text},
        {"Athlete", type text},
        {"Country", type text},
        {"Gender", type text},
        {"Event", type text},
        {"Medal", type text}
    })
in
    ChangedTypes
```

### Optional recommended additions

- **Split Athlete name** into `Surname` and `Firstname` using
  *Split Column → By Delimiter → Space → Left-most delimiter*.
- Create a **Medal Color** measure for visuals:

```dax
Medal Color =
SWITCH(
    SELECTEDVALUE('summer'[Medal]),
    "Gold",   "#FFD700",
    "Silver", "#C0C0C0",
    "Bronze", "#CD7F32",
    BLANK()
)
```

- Create a **Medal Count** measure:

```dax
Medal Count = COUNTROWS('summer')
```

- Build a **star schema**: create separate dimension tables for `Country`,
  `Sport`, `Event`, and `Year` (or a `Date` table) and relate them back to the
  fact table for best performance.

### Modelling tips

- Because the data is already flat and clean, a **single-table model** works
  fine for quick exploration; a **star schema** is better for larger dashboards.
- Add a **Year → Date** table if you want time-intelligence functions.

---

## 🗂️ Data Quality & Methodology

- The dataset reflects **officially awarded medals at the time of the Games**
  and does not include later reallocations from doping disqualifications in all
  cases.
- **Early Games (1896–1908)** contain several anomalies typical of historical
  records: combined teams, athletes competing for multiple nations, and
  discontinued events.
- Country codes follow the **IOC standard**, not modern ISO codes.

---

## 📜 Source & Credit

- Compiled from publicly available **Olympic historical records** and the
  well-known **"Olympic History" / Summer Olympic medalists** open datasets
  widely used in the data community (originally popularised via Kaggle and
  sports data archives).
- Please credit this repository and the original historical sources when
  reusing the dataset in public posts, articles, or dashboards.

> **Suggested attribution:** "Summer Olympics Medalists Dataset (1896–2012),
> sourced from historical Olympic records."

---

## 📄 License

This dataset is shared for **educational and analytical purposes**. Historical
Olympic results are public information; the compiled CSV is provided as-is.
Please verify against official IOC records before using it for official
publications.

---

## 🤝 Contributing

Found an inconsistency or want to add editions beyond 2012? Open an issue or
submit a pull request. Contributions that improve accuracy against official
records are especially welcome.

---

## ⭐ If you find this useful

Star the repo, and tag me when you build something with it on LinkedIn — I love
seeing what people create!
