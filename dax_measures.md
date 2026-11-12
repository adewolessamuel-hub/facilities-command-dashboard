# DAX Measures

Every formula used in `FacilitiesDashboard.pbix`, in the order it was built.

## Setup

Load `work_orders.csv` and rename the table to **WorkOrders**. In Power Query set `date_opened` and `date_closed` to Date/Time, `cost_cad` to Fixed decimal number, and the two hours columns to Decimal number.

### Date table (Modeling > New table)
```dax
Calendar = ADDCOLUMNS(CALENDAR(DATE(2025,11,1), DATE(2026,10,31)), "MonthStart", DATE(YEAR([Date]), MONTH([Date]), 1), "Month", FORMAT([Date], "MMM yyyy"))
```
Sort the `Month` column by `MonthStart` (Column tools > Sort by column) so months appear in calendar order, not alphabetical order.

### Date column on WorkOrders (Modeling > New column)
```dax
OpenedDate = DATE(YEAR(WorkOrders[date_opened]), MONTH(WorkOrders[date_opened]), DAY(WorkOrders[date_opened]))
```

### Relationship
`Calendar[Date]` (one) to `WorkOrders[OpenedDate]` (many), single cross-filter direction.

## Measures

```dax
Total Work Orders = COUNTROWS(WorkOrders)
```
```dax
Total Cost = SUM(WorkOrders[cost_cad])
```
```dax
Avg Response Hours = AVERAGE(WorkOrders[response_hours])
```
```dax
Open Work Orders = CALCULATE(COUNTROWS(WorkOrders), KEEPFILTERS(WorkOrders[status] <> "Closed"))
```
`KEEPFILTERS` matters here. Without it, splitting the backlog chart by status would show the site total in every segment.

```dax
Preventive WOs = CALCULATE(COUNTROWS(WorkOrders), WorkOrders[work_type] = "Preventive")
```
```dax
Preventive % = DIVIDE([Preventive WOs], [Total Work Orders])
```

## Visuals

| Visual | Type | Fields |
|---|---|---|
| Five KPI cards | Card | Total Cost, Avg Response Hours, Preventive %, Open Work Orders, Total Work Orders |
| Response time by priority (hours) | Clustered bar | Y-axis: priority. X-axis: Avg Response Hours |
| Open backlog by site | Stacked column | X-axis: site. Y-axis: Open Work Orders. Legend: status |
| Preventive share by month, 40% target | Line | X-axis: Calendar[Month]. Y-axis: Preventive %. Constant line at 0.4 |
| Cost per site (CAD) | Clustered column | X-axis: site. Y-axis: Total Cost |
| Cost by trade (CAD) | Clustered bar | Y-axis: trade. X-axis: Total Cost |
| Sites | Button slicer | site |

## Verification

With no site selected, the cards must read:

| Measure | Expected |
|---|---|
| Total Work Orders | 1,200 |
| Total Cost | $886,407.74 |
| Avg Response Hours | 10.65 (shown as 10.7) |
| Preventive % | 39.08% |
| Open Work Orders | 49 |

With Heartlake Medical Plaza selected: 407 work orders and $328,177.25 total cost.
