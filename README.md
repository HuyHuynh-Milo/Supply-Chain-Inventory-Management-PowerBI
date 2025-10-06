# Supply-Chain-Inventory-Management-PowerBI
🏭This project was created to fulfill the needs of inventory management: keep track of the storage quantity, value, as well as fully control inventory status such as stockout, excess stock. The Dashboard performs in Power BI application with many usage of DAX and Power Query, BigQuery, and SQL are also used to query the dataset.
![Cover image](https://github.com/user-attachments/assets/9d4ddbd8-1880-4913-b3aa-a68cc20fea1e)
- Author: Huy Huynh
- Date: September 2025
- Tools: PowerBI, BigQuery, SQL
## 📝 1. Background & Overview
📌 **a. Problem Statement**
Businesses often struggle to maintain optimal inventory levels due to a lack of visibility into stock movements, valuation, and future demand. Inefficient inventory tracking can lead to overstock, stockouts, and inaccurate financial reporting. This project aims to develop an Inventory Tracking System that provides real-time insights into:
- Stock quantity, inventory value, and total items remaining
- Fully control of Stock status like stockout, atstock, excessstock, and below-safety stock
- Forecast future stock and day to stockout each item
- Understand the excess and missing-stock value.
- Helping businesses make data-driven decisions to improve supply chain efficiency and reduce costs.

🏹 **b. Who is this project for**
- Supply chain managers, inventory planners, and business analysts who need to monitor stock performance.
- Businesses seeking a data-driven approach to track inventory value, identify slow-moving items.

## 📂 2. Dataset Description & Data Structure
### 🗃️ Raw Dataset
- This is a dataset of an imaginary e-commerce company named *Adventure Works*, they sells bicycles and things related to sport.
- The dataset is in the Google Cloud Service. To get the data, use *Google BigQuery*. Check for it:
  - [BigQuery Plaform](https://cloud.google.com/bigquery/docs/sandbox)
- This is a very massive dataset with several tables containing information about sales, customers, employees, manufacutures, etc. Connect to each other using ***keys***.
- An exemple for the dataset conection:

 <img width="1667" height="1616" alt="image" src="https://github.com/user-attachments/assets/6395e28c-77da-4314-91f5-51a5a41a8840" />


- But for this project, I'll have some steps to extract, transform, and load the dataset to get the necessary information only.
- 5 tables: **Product, Inventory, Category, WorkOrder, and SaleOrder** are the main tables that will be used for this report.

### 📩 Get Data
- To synthesize 5 tables i mentioned before, some SQL functions  will be used on the Google BigQuery
```SQL
# Query Product Table
SELECT ProductID, Name, 
       SafetyStockLevel,
       ReorderPoint, StandardCost, 
       DaysToManufacture, FinishedGoodsFlag,
       ProductSubcategoryID
FROM `adventureworks2019.Production.Product`
;

# Query Category Table 
SELECT 
       ps.ProductSubcategoryID, 
       ps.Name AS Subcategory,
       ps.ProductCategoryID,
       pc.Name AS Category
From `adventureworks2019.Production.ProductSubcategory` AS ps
JOIN `adventureworks2019.Production.ProductCategory` AS pc 
ON ps.ProductCategoryID = pc.ProductCategoryID
;

# Query WorkOrder Table
SELECT ProductID, WorkOrderID, 
       OrderQty, StartDate, 
       EndDate, DueDate
FROM `adventureworks2019.Production.WorkOrder`
;

# Query Inventory Table
SELECT pi.ProductID,
       pi.LocationID,
       pi.Quantity,
       pl.Name AS Location,
       pi.ModifiedDate 
FROM `adventureworks2019.Production.ProductInventory` AS pi 
JOIN `adventureworks2019.Production.Location` AS pl 
ON pi.LocationID = pl.LocationID
;

# Query SaleOrder Table
SELECT sd.SalesOrderDetailID,
       sh.SalesOrderID,
       sh.OrderDate,
       sh.SubTotal AS SubTotal_Order,
       sd.ProductID,
       sd.OrderQty,
       sd.UnitPrice,
       sd.LineTotal AS LineTotal_Product
FROM `adventureworks2019.Sales.SalesOrderHeader` AS sh
JOIN `adventureworks2019.Sales.SalesOrderDetail` AS sd
ON sh.SalesOrderID = sd.SalesOrderID
ORDER BY sd.SalesOrderDetailID
;
```

### 📬 Using Dataset

A quick overview of these tables:
- **Product** table:

| Column Name              | Data Type          | Description                                                                                                                          | Example Value            |
| ------------------------ | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------ |
| **ProductID**            | `int`              | Unique identifier for each product in the database. Serves as the primary key for the table.                                         | 707                      |
| **Name**                 | `nvarchar(50)`     | The name of the product, typically used in listings and reports.                                                                     | *Mountain-200 Black, 42* |
| **SafetyStockLevel**     | `int`              | The minimum quantity of the product that should be kept in inventory to prevent stockouts.                                           | 500                      |
| **ReorderPoint**         | `int`              | The inventory level that triggers a new purchase or production order. When stock drops below this point, replenishment should begin. | 375                      |
| **StandardCost**         | `money`            | The manufacturing or purchasing cost of one unit of the product (not the selling price). Used for cost accounting and valuation.     | 594.83                   |
| **DaysToManufacture**    | `int`              | The average number of days required to manufacture the product. Used for production planning and lead time analysis.                 | 4                        |
| **FinishedGoodsFlag**    | `int`              | Indicates whether the product is a finished good (`1`) or a component/sub-assembly (`0`).                                            | 1                        |
| **ProductSubcategoryID** | `int` *(nullable)* | References the product’s subcategory in the `Production.ProductSubcategory` table. Helps group products by type.                     | 14                       |

- **Category** table:

| Column Name                           | Data Type      | Description                                                                                                                                                              | Example Value    |
| ------------------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------- |
| **ProductSubcategoryID**              | `int`          | Unique identifier for each product subcategory. Serves as the primary key in the `ProductSubcategory` table and a foreign key in `Product` (via `ProductSubcategoryID`). | 14               |
| **Subcategory** *(alias for ps.Name)* | `nvarchar(50)` | The name of the product subcategory (e.g., “Mountain Bikes”, “Helmets”).                                                                                                 | *Mountain Bikes* |
| **ProductCategoryID**                 | `int`          | Foreign key linking the subcategory to its parent category in the `ProductCategory` table.                                                                               | 1                |
| **Category** *(alias for pc.Name)*    | `nvarchar(50)` | The name of the higher-level product category (e.g., “Bikes”, “Accessories”).                                                                                            | *Bikes*          |

- **WorkOrder** Table:

| Column Name     | Data Type               | Description                                                                                       | Example Value       |
| --------------- | ----------------------- | ------------------------------------------------------------------------------------------------- | ------------------- |
| **ProductID**   | `int`                   | Foreign key referencing the product being manufactured (links to `Production.Product.ProductID`). | 707                 |
| **WorkOrderID** | `int`                   | Unique identifier for each work order — the primary key of the table.                             | 51234               |
| **OrderQty**    | `int`                   | The total quantity of the product that is scheduled to be manufactured in this work order.        | 100                 |
| **StartDate**   | `datetime`              | The date when production on the work order actually started.                                      | 2020-07-12 00:00:00 |
| **EndDate**     | `datetime` *(nullable)* | The date when production was completed. May be `NULL` if the work order is still in progress.     | 2020-07-20 00:00:00 |
| **DueDate**     | `datetime`              | The planned completion date for the work order, used to track delays or early completions.        | 2020-07-22 00:00:00 |

- **Inventory** Table:

| Column Name                        | Data Type      | Description                                                                                                             | Example Value            |
| ---------------------------------- | -------------- | ----------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| **ProductID**                      | `int`          | Foreign key referencing the product in `Production.Product`. Identifies which product this inventory record belongs to. | 707                      |
| **LocationID**                     | `smallint`     | Foreign key referencing the warehouse or storage location in `Production.Location`.                                     | 6                        |
| **Quantity**                       | `smallint`     | Number of units of the product currently in stock at that specific location.                                            | 254                      |
| **Location** *(alias for pl.Name)* | `nvarchar(50)` | Descriptive name of the inventory location, such as a warehouse or sub-assembly area.                                   | *Finished Goods Storage* |
| **ModifiedDate**                   | `datetime`     | Date and time when this inventory record was last updated — used for tracking stock changes.                            | 2020-09-12 00:00:00      |

- **SaleOrder** Table:

| Column Name                                      | Data Type       | Description                                                                                                              | Example Value       |
| ------------------------------------------------ | --------------- | ------------------------------------------------------------------------------------------------------------------------ | ------------------- |
| **SalesOrderDetailID**                           | `int`           | Unique identifier for each line item within a sales order. Serves as the primary key in `SalesOrderDetail`.              | 110563              |
| **SalesOrderID**                                 | `int`           | Foreign key linking the detail line to the sales order header. Used to group multiple products under one customer order. | 43659               |
| **OrderDate**                                    | `datetime`      | Date when the sales order was created. Useful for analyzing time-based trends and seasonality.                           | 2020-08-12 00:00:00 |
| **SubTotal_Order** *(alias for sh.SubTotal)*     | `money`         | The total amount for the entire sales order before tax, shipping, and discounts.                                         | 1,890.50            |
| **ProductID**                                    | `int`           | Foreign key referencing the product sold, linking to `Production.Product`.                                               | 707                 |
| **OrderQty**                                     | `smallint`      | Quantity of the product ordered in this line item.                                                                       | 3                   |
| **UnitPrice**                                    | `money`         | The price per unit of the product at the time of sale.                                                                   | 594.83              |
| **LineTotal_Product** *(alias for sd.LineTotal)* | `numeric(38,6)` | The total price for this line item (calculated as `OrderQty × UnitPrice`, adjusted for any discounts).                   | 1,784.49            |

- The Product table is the main table that connects to 3 other tables by the key ***ProductID***. With an exception for the category table, they are connected by the **ProductSubcategoryID**
## 3. Main process in Power BI
### 1. ⚒️ Preprocessing Data
**a. Dataset & Preprocessing explanation**
- This dataset doesn't have the stock quantity during the time period, so i'll hypothesize that the dataset ends its record on 31-May and the calculations like Stock Quantity, Value, Status,... are at that time.
- Every calculation after the end of May (like work order) is my Forecast for future demand.

**b. Inventory Turnover**
- The Inventory Turnover is gonna be calculated by approximation (due to the lack of stock quantity for the time period):

  <img width="427" height="77" alt="image" src="https://github.com/user-attachments/assets/e6ce4999-a075-4fe5-82fd-fd66ddd039a0" />
  
```DAX
InventoryTurnOver = 
VAR COGS = CALCULATE(
    SUMX(SaleOrder, SaleOrder[OrderQty]*RELATED('Product'[StandardCost])),
    FILTER(SaleOrder,SaleOrder[OrderDate] >= DATE(2013,6,1) && 
           SaleOrder[OrderDate]  < DATE(2014,6,1))
    )
RETURN DIVIDE(COGS,[InventValue])
```

**c. Stock Status**
- At Stock is the range of item quantity from Safety Stock -> SafetyStock * 1.3
- Excess-Stock is everything above the Maximum At-Stock
- Below safety are everything below the safety point.
- DAX code for stock status:
```DAX
StockStatus = 
    SWITCH(
        TRUE(),
        Inventory[Quantity] = 0, "Out Stock",
        Inventory[Quantity] < RELATED('Product'[SafetyStockLevel]), "Below Safety",
        Inventory[Quantity] <= RELATED('Product'[SafetyStockLevel])*1.3, "At Stock",
        "Over Stock"
        )
```
**d. Table Connection**
- I create a calendar table for datetime calculation and a Forecast Calendar for forecasting future inventory.

<img width="1663" height="1321" alt="image" src="https://github.com/user-attachments/assets/8ca51742-7638-4345-820b-16e560b50a6a" />


### 2. 📈 Dashboard
**a. Overview**

<img width="2556" height="1424" alt="image" src="https://github.com/user-attachments/assets/24f8dd34-57f0-4667-a14e-b9ba82afeb4d" />


🗝️**Key Observation**
- Total inventory value: ~$7.24M, Total Items: 151, Total Quantity: 17,319.
- SKU health: Out of 151 SKUs, 73 SKUs (~48.3%) are below safety stock, 50 SKUs (~33.1%) are overstock, 24 SKUs (~15.9%) are at target, and 4 SKUs (~2.65%) are out of stock.
- Value concentration: Bikes account for ~$7.06M (≈97.5% of total value)
- Quantity distribution: Bikes ~44.9% of units, Clothing ~34.9%, Accessories ~22.0%.
- Working capital needed to refill: Refill value ≈ $2.3M (~31.8% of total inventory value).
- Excess inventory value: ≈ $177.6K (~2.45% of total) — relatively small versus total value.

🧠**Implication**

**1.Stock-out risk vs. cash constraint tension.**
- The total inventory asset base comprises 151 individual items totaling a Quantity of 17K units, collectively valued at $7.24M.
- A minitory of items are maintained at satisfactory stock levels: **24 tiems (15.9%)** At Stock. However, more than **50.9%** of the portfolio is either Below Safety **(73 items)** or Out-stock **(4 items)**.
 
-> This critical imbalance results in a ratio of instability (Below Safety + Out Stock) to stability (At Stock) of 3.04:1 
- Inventory requirements indicate a necessary capital injection; the "Refill Value" of **$2.3M** represents **31.9%** of the total current inventory valuation.

**2. High financial risk concentrated in bikes**
- The $7.24M inventory portfolio demonstrates extreme financial concentration, the Bike category constitute majority of asset value: **$7.06M,** or **97.5%**, of the total inventory value.
- Clothing accounts for **$0.14M (1.9%)** and Accessories constitute the **negligible** remainder
- The company's entire asset base is exposed to bicycle market risk: unexpected shift in demand for model, competitive pricing pressure, and change in sourcing cost.

**3. Interpreting Inventory Turnover**
- The Overall Inventory turnover stands at **6.04**, indicating that the average capital holding cycle of approximately **61 days** (365/6.04).
- Accessories achieved the highest turnover at **10.96**, suggesting very strong sales velocity and lean inventory management. Clothing followed at **7.34**, showing stable movement aligned with seasonal demand. Bikes recorded a turnover of **5.26** — slightly slower but still within an acceptable range for high-value.
- Overall, a turnover **above 6** indicates that inventory management practices are **effective**, with stock levels well matched to demand. The focus going forward should be on **maintaining** this turnover while preventing stockouts in fast-moving categories like Accessories.

**4. Opportunity in overstocked categories.**
- Clothing and Accessories show high overstock percentages (per-SKU), more than **91%** items are overstock. Yet they represent a tiny portion of total value — good candidates for working-capital recovery through promotions/redistribution.
