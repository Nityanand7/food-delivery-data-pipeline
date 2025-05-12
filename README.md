# Food Delivery End-to-End Data Pipeline using Snowflake

## Prerequisites
- A Snowflake account (trial or paid) with appropriate roles (e.g., SYSADMIN).
- SnowSQL CLI installed or access to Snowsight web UI.
- Python 3.8+ (for the Streamlit dashboard).
- Dependencies: `snowflake-connector-python`, `snowflake-snowpark-python`, `streamlit`, `pandas`, `altair`.

## Flow
1. **Database & Schema Setup**: Create warehouse, database, schemas, file formats, stages, tags, and masking policies.  
2. **Data Ingestion**: Upload initial and delta CSVs (for each entity) into Snowflake internal stages and load into raw staging tables.  
3. **Clean Layer**: Apply type conversions, business logic, and upsert (MERGE) from staging streams into clean tables.  
4. **Consumption Layer**: Build dimension (SCD Type 2) and fact tables, then upsert into consumption-layer tables.  
5. **Visualization**: Launch a Streamlit dashboard to explore key metrics.

## High-Level Data Flow Architecture
![image](https://github.com/user-attachments/assets/5af814ca-9e51-4a41-af2c-7c249f84b7ad)

## Food Order Booking and Receiving Flow
![image](https://github.com/user-attachments/assets/14e5a652-7c7b-4ac3-aff8-5d565f9efdea)

## ER Model
![image](https://github.com/user-attachments/assets/1016538b-2831-41a4-86bf-f69dcdd176c3)


---

## 1. Create Database, Schema & Common Objects
All DDL scripts for foundation setup are in **scripts/setup_foundation.sql**.  
This script includes:
- Warehouse creation (`adhoc_wh`)
- Database & schemas (`sandbox`, `stage_sch`, `clean_sch`, `consumption_sch`, `common`)
- File format & internal stage
- Tags & masking policies

---

## 2. Data Loading Step
- CSV files are located in the **data/** directory, organized per entity:
  ```
  data/
    location/
      initial/location-5rows.csv
      delta/delta-day01-2rows-add.csv
      delta/delta-day02-2rows-update.csv
      ... (for error-handling demos)
    restaurant/
      ...
    customer/
      ...
    orders/
      ...
    order_item/
      ...
  ```
- **scripts/load_stage.sql** contains `PUT` and `COPY INTO` commands for all entities.
- Load both initial and delta files to populate your staging tables.

---

## 3. Location Dimension
You’ll transform raw location data into an SCD Type 2 dimension:
- **scripts/01_location_dimension.sql**  
  - Cleans and casts data types.
  - Generates surrogate keys, state codes, city tiers, union territory flags, etc.
  - Upserts (MERGE) into `clean_sch.restaurant_location`.
  - Builds `consumption_sch.restaurant_location_dim` with SCD2 logic.

---

## 4. Restaurant Dimension
- **scripts/02_restaurant_dimension.sql**  
  - Process `restaurant` staging table.
  - Apply business rules (cuisine types, pricing tiers).
  - Upsert into `clean_sch.restaurant`.
  - Build `consumption_sch.restaurant_dim`.

---

## 5. Customer & Address Dimensions
- **scripts/03_customer_dimension.sql**  
- **scripts/04_customer_address_dimension.sql**  
  - Clean and dedupe customer data.
  - Build `consumption_sch.customer_dim` & `customer_address_dim`.

---

## 6. Menu & Delivery Agent Dimensions
- **scripts/05_menu_dimension.sql**  
- **scripts/06_delivery_agent_dimension.sql**  
  - Structure menu items and delivery agent data.
  - Upsert into clean and consumption schemas.

---

## 7. Delivery Entity Fact Table
- **scripts/07_delivery_fact.sql**  
  Merge delivery events into `consumption_sch.delivery_fact`.

---

## 8. Order Entity Fact Table
- **scripts/08_order_entity_fact.sql**  
  Merge order header data into `consumption_sch.order_fact`.

---

## 9. Order Item Entity Fact Table
- **scripts/09_order_item_entity_fact.sql**  
  Merge individual order items into `consumption_sch.order_item_fact`.


---

## 10. Date Dimension
- **scripts/10_date_dimension.sql**  
  Populate a calendar table via a recursive CTE for date lookups.
  

---

## 11. Order Item Fact
- **scripts/11_order_item_fact.sql**  
  Merge individual order items into `consumption_sch.order_item_fact`.

---

## 12. Final Views
- **scripts/12_final_views.sql**  
  Create reporting views (daily, monthly, yearly KPIs).

---

## 13. Streamlit Dashboard
- **app.py**  
  Connects via Snowpark.
  Displays metrics and charts using Altair.
- To run:
  ```bash
  pip install -r requirements.txt
  streamlit run app.py
  ```


## Snowsight dashboard

![image](https://github.com/user-attachments/assets/afb0dc7c-2d97-496c-a4e1-863b63f9d20c)

![image](https://github.com/user-attachments/assets/446efd86-1b85-43a1-bc63-2cc0a28316fe)


---

> Scripts are in the `scripts/` directory, and raw data is in the `data/` directory.
> Credits: Data Engineering Simplified


