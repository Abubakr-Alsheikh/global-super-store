# Global Super Store - Data Warehouse and Analytics Project

## Project Overview

This project was undertaken as part of the **Advanced Data Modeling module for the Meta Database Engineer Professional Certificate**. The primary objective was to address the challenges of a business operating with a large, denormalized dataset in a single Excel file.

The goal was to restructure this data into a robust and scalable system using a proper relational database management system (MySQL). This involved designing a normalized transactional database, creating a star schema for analytics, and building an interactive dashboard to analyze business performance in the USA market.

### A Note on Technology: Using Metabase

While the original project brief suggested using Tableau, this implementation utilizes **Metabase**, a powerful, free, and open-source business intelligence tool. Metabase provides much of the same core functionality as Tableau, allowing for the creation of rich, interactive charts and dashboards, making it an excellent alternative for developers and analysts.

---

## Technology Stack

* **Database:** MySQL
* **Database IDE:** MySQL Workbench
* **BI & Visualization:** Metabase
* **Data Source:** Excel / CSV file

---

## 1. Database Design and Implementation

The first phase of the project focused on converting the flat Excel file into a structured, relational database.

### a. Logical Design (ERD)

Using MySQL Workbench, an Entity-Relationship Diagram (ERD) was designed to map out the entities and their relationships. The design adheres to the **Third Normal Form (3NF)** to reduce data redundancy and improve data integrity.

The normalized schema consists of the following tables:

* `tbl_customers`: Stores unique customer information.
* `tbl_locations`: Stores unique geographic data (City, State, Postal Code).
* `tbl_orders`: Contains high-level order information like dates and shipping details.
* `tbl_products`: Stores unique product information.
* `tbl_categories` & `tbl_subcategories`: Normalizes product categorization.
* `tbl_order_details`: A junction table linking orders and products, resolving the many-to-many relationship.

### b. Analytical Design (Star Schema)

For efficient analysis and reporting, a **Star Schema** was designed and implemented. This schema is optimized for the marketing department's needs, focusing on sales performance across products, locations, and time.

* **Fact Table:**
  * `Fact_Sales`: A central table containing quantitative measures like `SalesAmount`, `Profit`, `QuantitySold`, and `ShippingCost`.
* **Dimension Tables:**
  * `Dim_Product`: Describes product attributes (Name, Category, Sub-Category).
  * `Dim_Location`: Describes geographic attributes (City, State, Country).
  * `Dim_Time`: Describes date attributes (Year, Quarter, Month).

---

## 2. ETL Process: From Excel to a Data Warehouse

A multi-step ETL (Extract, Transform, Load) process was executed to migrate and clean the data.

### a. Extract

The data was first extracted from the source Excel file by saving it as a `CSV` file.

### b. Transform and Load

This phase was the most critical, involving several data cleaning and loading steps.

1. **Staging:** The raw CSV data was first loaded into a single `staging_superstore` table in MySQL. This isolated the raw, messy data from the clean, structured production tables.

2. **Normalization and Cleaning:** SQL scripts were written to populate the normalized `tbl_` tables from the staging table. This process uncovered and resolved several data quality issues:
    * **Duplicate Records:** The source data contained multiple entries for the same Customer ID, Product ID, and Order ID. The `GROUP BY` clause combined with aggregate functions like `MIN()` and `SUM()` was used to create single, unique master records.
    * **Inconsistent Date Formats:** The `OrderDate` column contained mixed formats (`MM/DD/YYYY` and `DD-MM-YYYY`). A `CASE` statement within the `STR_TO_DATE` function was used to handle this inconsistency.
    * **Primary/Unique Key Violations:** During the `INSERT` process, several duplicate key errors were encountered. These were systematically resolved by aggregating the data correctly (e.g., summing quantities for the same product in the same order) and using `TRUNCATE TABLE` to ensure a clean slate before re-running a load script.

3. **Populating the Data Warehouse:** Once the normalized tables were clean and populated, a final set of SQL scripts was run to populate the `Fact_Sales` and Dimension tables. These scripts joined the clean transactional tables and loaded the data into the star schema, making it ready for analysis.

---

## 3. Data Analysis and Visualization with Metabase

Using Metabase connected to the MySQL data warehouse, the following visualizations and dashboard were created.

### a. Sales in USA Map

A map chart was created to visualize total sales in different states across the USA.

* **Process:** The `Fact_Sales` table was used as the source. Sales (`SalesAmount`) were summed up and grouped by State (`Dim_Location.State`). Metabase's map visualization automatically rendered the data, with tooltips showing the state name and sales figures on hover.

![Sales in USA Map Chart](https://raw.githubusercontent.com/Abubakr-Alsheikh/global-super-store/main/images/sales-map.jpg)

### b. Profits in USA Bubble Chart

A bubble chart (implemented as a Scatter Plot in Metabase) was created to analyze profitability.

* **Process:** Data was grouped by State. The chart was configured with `Profit` on the x-axis, `ShippingCost` on the y-axis, and `QuantitySold` controlling the size of the bubble. This provides a multi-dimensional view of performance in each state.

![Profits in USA Bubble Chart](https://raw.githubusercontent.com/Abubakr-Alsheikh/global-super-store/main/images/profits-bubble-chart.jpg)

### c. Sales Trend in USA Line Chart

A line chart was created to show sales trends over the last four years for states with total sales exceeding $40,000.

* **Process:** Due to the complexity of the aggregate filter (`HAVING SUM(Sales) > 40000`), a **Native SQL Query** was written in Metabase. This query first identified the qualifying states and then pulled their monthly sales data to visualize the trend over time.

![Profits in USA Bubble Chart](https://raw.githubusercontent.com/Abubakr-Alsheikh/global-super-store/main/images/line-chart.jpg)

### d. Interactive Dashboard: Sales and Profits in the USA

Finally, all three charts were combined into a single, interactive dashboard named "Sales and Profits in the USA".

* **Interactivity:** A **State filter** was added to the dashboard and linked to all three charts. This allows a user to select a specific state (or multiple states) from a dropdown list, and all charts on the dashboard instantly update to display data only for the selected region.

Finally, all three charts were combined into a single, interactive dashboard named "Sales and Profits in the USA".

![Interactive Dashboard Screenshot](https://raw.githubusercontent.com/Abubakr-Alsheikh/global-super-store/main/images/interactive-dashboard.jpg)
