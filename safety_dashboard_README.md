# Industrial Safety Observations Dashboard (Power BI)

> **Disclosure:** All data in this project is fully synthetic — generated to mirror the schema of industrial safety observation tracking systems used in oil & gas and petrochemical facilities. No real company data, personnel information, or operational records are included.

---

## Files

| File | Type | Description |
|---|---|---|
| `observations.csv` | Fact table | 500 synthetic safety observations, Jul 2024 – Jun 2025 |
| `units.csv` | Dimension | 8 industrial units |
| `zones.csv` | Dimension | 19 zones, each linked to a parent unit |
| `categories.csv` | Dimension | 10 safety observation categories |
| `status.csv` | Dimension | 4 resolution statuses |

---

## Data Model (Star Schema)

```
units ────────┐
              │ (1:many via UnitId)
zones ────────┤
              │ (1:many via ZoneId)
              ├──► observations (fact)
categories ───┤ (1:many via CategoryId)
              │
status ───────┘ (1:many via StatusId)
```

Set up these relationships in Power BI Desktop under **Model view** before building any visuals.

---

## DAX Measures to Write

Create these in a dedicated `_Measures` table (blank table, no data):

```dax
Total Observations = COUNT(observations[ObservationId])

Open Observations = 
    CALCULATE(
        COUNT(observations[ObservationId]),
        status[StatusName] = "Open"
    )

Closed Observations = 
    CALCULATE(
        COUNT(observations[ObservationId]),
        status[StatusName] = "Closed"
    )

Closure Rate % = 
    DIVIDE([Closed Observations], [Total Observations], 0)

High Risk Count = 
    CALCULATE(
        COUNT(observations[ObservationId]),
        observations[RiskRating] IN {"Critical", "High"}
    )

High Risk % = 
    DIVIDE([High Risk Count], [Total Observations], 0)

Highlight Count = 
    CALCULATE(
        COUNT(observations[ObservationId]),
        observations[is_highlight] = 1
    )

Avg Monthly Observations = 
    AVERAGEX(
        VALUES('Date'[Month]),
        [Total Observations]
    )
```

---

## Recommended Visuals

**Page 1 — Overview**
- 4 KPI cards: Total Observations | Open | High Risk % | Closure Rate %
- Bar chart: Observations by Category (sorted descending)
- Donut: Status breakdown
- Line chart: Monthly observation trend (Date hierarchy → Month)
- Slicer: Date range, Risk Rating

**Page 2 — Unit & Zone Breakdown**
- Matrix: Unit (rows) × Category (columns) → COUNT of observations
- Bar chart: Observations by Unit
- Treemap or filled map: Zone-level heatmap
- Slicer: Unit, Status

**Page 3 — Risk & Highlights**
- Bar chart: Risk Rating distribution
- Table: All is_highlight = 1 records (ObservationId, Unit, Category, Risk, Status, Date)
- Conditional formatting on Risk Rating column: Critical = red, High = orange, Medium = yellow, Low = green
- Slicer: Category, Date

---

## Key Patterns in the Data

- Medium and Low risk observations make up ~70% of volume — expected for a functioning safety program.
- Critical and High risk observations are more likely to remain Open or In Progress vs. Closed.
- PPE Compliance, Housekeeping, and Mechanical Integrity are consistently high-volume categories — worth calling out in a presentation as chronic vs. acute risk indicators.
- 101 flagged observations (`is_highlight = 1`) — driven almost entirely by Critical/High risk events.

---

## Tools
Power BI Desktop · Star schema data model · DAX measures · Conditional formatting
