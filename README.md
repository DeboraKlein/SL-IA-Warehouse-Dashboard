## Warehouse KPI Dashboard
This project presents an interactive dashboard developed in Power BI to monitor key logistics indicators in warehouse operations. The solution is based on simulated data and aims to demonstrate the ability to consolidate, model, and visualize critical information for operational decision-making.

### Objective
Provide a clear and reliable view of warehouse performance, focusing on:

Productivity by operation and shift

Inventory accuracy by SKU and unit

Operational costs by warehouse and period

Service level and SLA compliance (OTIF)

### Technologies Used
Power BI: Data modeling, DAX, Power Query, and interactive visuals

Excel / CSV: Simulated and structured operational data

SQL (conceptual): Foundation for data integration and extraction

Python (conceptual): Future applications for automation and forecasting

### Data Structure
The dataset is organized into four main files:

File	Description
warehouse_operations.csv	Item volume, operation time, and errors by shift
inventory_accuracy.csv	Comparison between physical and system stock, with accuracy calculation
operational_costs.csv	Fixed and variable costs by warehouse and month
service_level.csv	Real-time vs. SLA per order, with OTIF indicator

FactServiceLevel — Grain: one row per order per date (identified by Pedido/SKU code), contains SLA (min), Actual Time (min) and OTIF flag.

FactOperations — Grain: one row per operation occurrence per shift per date (operation × shift × date), contains processed time and error counts.

FactOperational_Costs — Grain: one row per warehouse per month (warehouse × month), aggregates cost and processed items.

FactInventory_Accuracy — Grain: one row per warehouse per SKU per inventory count date (warehouse × SKU × inventory_count_date), contains system vs physical counts and accuracy.

### Calculated Indicators
Average Productivity = Items Processed ÷ Total Time

Error Rate (%) = (Errors ÷ Items Processed) × 100

Total Volume Processed = Sum of Items Processed

Inventory Accuracy (%) = (Physical Qty ÷ System Qty) × 100

Cost per Item = Total Cost ÷ Items Processed

OTIF (%) = % of orders delivered within SLA

## DAX Measures Used
These measures were developed to calculate warehouse KPIs with precision and flexibility, using advanced functions like CALCULATE, DIVIDE, AVERAGEX, and context control via ALLEXCEPT.

### Average Inventory Accuracy by Warehouse
````
Average Inventory Accuracy by Warehouse = 
VAR TotalSystem = CALCULATE(
    SUM(FactInventory_Accuracy[System Quantity]), 
    ALLEXCEPT(FactInventory_Accuracy, FactInventory_Accuracy[Warehouse])
)
VAR TotalPhysical = CALCULATE(
    SUM(FactInventory_Accuracy[Physical Quantity]), 
    ALLEXCEPT(FactInventory_Accuracy, FactInventory_Accuracy[Warehouse])
)
RETURN DIVIDE(TotalPhysical, TotalSystem, 0)
````
### Cost per Item
````
Cost per Item = 
DIVIDE(
    SUM(FactOperational_Costs[Total Cost]), 
    SUM(FactOperational_Costs[Processed Items])
)
````
### OTIF (%) by Month
````
OTIF (%) by Month = 
VAR TotalOrders = COUNTROWS(FactService_Level)
VAR OnTimeDeliveries = CALCULATE(
    COUNTROWS(FactService_Level),
    FactService_Level[OTIF] = "Yes"
)
RETURN DIVIDE(OnTimeDeliveries, TotalOrders)
````
### Average Productivity
````
Average Productivity = 
AVERAGEX(
    'FactOperations', 
    'FactOperations'[Processed Items] / FactOperations[Total Time (min)]
)
````
### Error Rate (%)
````
Error Rate (%) = 
DIVIDE(
    SUM('FactOperations'[Errors]), 
    SUM(FactOperations[Processed Items])
)
````
### Total Volume Processed
````
Total Volume Processed = 
SUM('FactOperations'[Processed Items])
````
### Total System Quantity
````
Total System Quantity = 
SUM(FactInventory_Accuracy[System Quantity])

````
## Duplicate Records Documentation — FactOperations
### Objective
Identify and resolve duplicate records in the fact table FactOperations, ensuring correct granularity by operation, shift, date, and warehouse.

### Duplicate keys found
The following composite keys had duplicate entries (2 records each):

Picking|Morning|2025-06-09

Picking|Night|2025-07-11

Receiving|Afternoon|2025-02-12

Packing|Morning|2025-04-29

Picking|Night|2025-01-03

### Investigation steps
Created column FactOperationsKey Composed of Operation|Shift|Date, later expanded to include Warehouse.

Duplicate detection via DAX Measure created to list keys with more than one occurrence:
````
DuplicateKeysList = 
VAR DuplicateKeys =
    FILTER(
        GROUPBY(
            FactOperations,
            FactOperations[FactOperationsKey],
            "Cnt", COUNTX(CURRENTGROUP(), 1)
        ),
        [Cnt] > 1
    )
RETURN
    CONCATENATEX(DuplicateKeys, FactOperations[FactOperationsKey] & " (" & [Cnt] & " times)", ", ")
````
Record inspection Confirmed that duplicates represented the same event and could be safely aggregated.

### Correction applied
Aggregation by FactOperationsKey in Power Query:

Aggregated: Processed Items, Total Time (min), Errors

Preserved: Operation, Shift, Date, Warehouse (first non-null occurrence)

Added OriginalCount column for audit purposes

Removed temporary columns and test tables Diagnostic elements were removed after validation.

### Post-correction validation
Comparison measures:

Rows Count = COUNTROWS(FactOperations)

DistinctKey Count = DISTINCTCOUNT(FactOperations[FactOperationsKey])

Inventory Accuracy adjusted from 96.85% → 96.80%

A multi-card visual was used to compare all key metrics before and after correction.

### Final notes
The 0.05 p.p. change in accuracy was deemed acceptable.

The model now ensures one row per operational event, respecting the defined grain.

Aggregation logic was documented and validated through measures and visuals

## Visualizations
The dashboard includes:

Line and bar charts by operation and warehouse

KPI cards with key metrics

Dynamic filters by date, unit, operation, and SKU

### Next Steps
Integration with real data via SQL Server or API

Automated updates using Power Automate

Predictive modeling with Python (demand forecasting, stockout risk)

Expansion to transportation and advanced warehousing KPIs

### How to Use
1. Clone this repository:
bash
git clone https://github.com/DeboraKlein/Warehouse-KPI-Dashboard.git
2. Open the .pbix file in Power BI Desktop
3. Explore the dashboard pages and KPIs
The simulated data is available in the /data folder.

## Dashboard Link

### View the published dashboard here

### [Click here to access the Dashboard](https://app.powerbi.com/view?r=eyJrIjoiNjA5NDRhYmMtMmZmNC00MmI5LTk1MGYtMWNiZWNlMTQ5NjZjIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9)

### Visualization Samples
![Dashboard - Visão Geral](https://github.com/user-attachments/assets/029dcdf7-fb70-4744-a408-f5692a86ccf5)
![Dashboard - Inventário e SLA](https://github.com/user-attachments/assets/679a8d4f-8a06-425f-aafe-75c9e989f19c)

## Related Projects
Logistics Dashboard: Fleet and delivery performance analysis

## About Me
This project was developed by Debora Klein, Data Analyst with experience in BI, financial planning, and operational intelligence. Passionate about transforming data into decisions.
