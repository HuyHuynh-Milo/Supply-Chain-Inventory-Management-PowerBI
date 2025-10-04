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
**🗃️ Raw Dataset**
- This is a dataset of an imaginary e-commerce company named *Adventure Works*, they sells bicycles and things related to sport.
- The dataset is in the Google cloud service, to get the data, use *Google BigQuery*. Check for it:
  - [BigQuery Plaform](https://cloud.google.com/bigquery/docs/sandbox)
- This is a very massive dataset with several tables contain information about sales, customers, employees, manufacutures, etc. Connect to each other using ***keys***.
- An exemple for the dataset conection:

  <img width="824" height="796" alt="image" src="https://github.com/user-attachments/assets/d1ed601d-72c5-4930-a1f5-e4e9c2312360" />

- But for this project i'll have some step to extract, transform and load the dataset to get the necessary information only. **Product, Inventory, Category, WorkOrder and SaleOrder** are the main tables that will be used for this report.
