# 🚀 Incremental Data Ingestion Using Watermark Method — Fabric Data Pipeline

A complete end-to-end guide demonstrating **Incremental Ingestion**, **Watermark Logic**, **Variable Library**, **Lookups**, **Copy Activity**, and **Stored Procedure Updates** in **Microsoft Fabric**.

---

## 📘 Overview

This project showcases **Incremental Data Ingestion** using the **Watermark Method** inside a **Microsoft Fabric Data Pipeline**.

The goal is simple:

- Only ingest **new or changed records**
- Use a **watermark table** to track the last load
- Parameterize values using **Variable Library**
- Ensure efficient, reliable, repeatable ingestion

This pattern applies to:

- **Fabric Data Pipelines**
- **Azure Data Factory**
- **Synapse Pipelines**

---

## 🧱 Architecture

| Component | Description |
|----------|-------------|
| **Source** | Azure SQL Database |
| **Ingestion** | Fabric Data Pipeline |
| **Method** | Watermark-based incremental load |
| **Control Table** | Stores Last Load Date ID |
| **Pipeline Activities** | Variable Library, Lookup, Copy, Stored Procedure |

### 📷 Architecture Diagram  

<img width="911" height="413" alt="image002 (1)" src="https://github.com/user-attachments/assets/c4d616ce-4da5-46e1-9ed0-9cac9b76803b" />

---

## 🏗️ Step-by-Step Implementation Guide

---

## ✅ **1. Configure Variable Library**

Navigate to: Data Pipeline → Settings → Variable Library → Add New Variable
<img width="608" height="382" alt="image003" src="https://github.com/user-attachments/assets/44c4b0e3-22db-4d5a-b520-abcb286ab89d" />

## ✅ 2. Create Watermark Table

This table stores the most recent successful load `DateID`.

```sql
CREATE TABLE dbo.WatermarkTable (
    last_load VARCHAR(100) 
);

-- Insert earliest date for initial backfill
INSERT INTO dbo.WatermarkTable 
VALUES ('DT00000');
```
💡 Why Date ID?
Most enterprise data models use a Date Dimension with a surrogate DateID for clean incremental logic.

## ✅ 3. Create Stored Procedure to Update Watermark

```sql
CREATE PROCEDURE UpdateWatermarkTable
    @lastload VARCHAR(200)
AS
BEGIN
    -- Start the transaction
    BEGIN TRANSACTION;

    -- Update the incremental column in the watermark table
    UPDATE water_table
    SET last_load = @lastload;

    -- Commit the transaction
    COMMIT TRANSACTION;
END;
```
 ## ✅ 4. Build the Fabric Data Pipeline 
 **🔹 Activity 1 — Lookup (Last Load)**
 Fetch previous watermark: 
 
 <img width="641" height="707" alt="image007" src="https://github.com/user-attachments/assets/b839ca06-86fe-4937-8b4d-0d7e1e2a0a9f" />

**🔹 Activity 2 — Lookup (Current Max Date)** 
Query the source to find the latest available DateID:

<img width="1745" height="689" alt="image006" src="https://github.com/user-attachments/assets/8aa2531d-7b96-4d5e-b236-4e429030eeb3" />

## 🤖 5. Copy Activity —
Incremental Load Logic Use the watermark values to pull only new records

Source: Azure SQL

Sink: Lakehouse / Warehouse / SQL DB 

Behavior: Load only delta rows

<img width="1830" height="681" alt="image005" src="https://github.com/user-attachments/assets/e71e551f-6e96-4742-bde5-7959de676fb5" />

## 🔁 6. Stored Procedure Activity —
Update Watermark Pass the new watermark value:

<img width="1701" height="761" alt="image004" src="https://github.com/user-attachments/assets/cdd70dbf-31e3-4055-a438-6c026ddfca27" />

# 📊 End-to-End Flow Diagram 
```sql
 ┌──────────────────────────┐
 │     Watermark Table      │
 │     (LastLoadDateId)     │
 └──────────────┬───────────┘
                │
        Lookup Activity
         (Last Load)
                │
                ▼
      Lookup Activity
    (Current Max Date)
                │
                ▼
         Copy Activity
         (Incremental)
                │
                ▼
     Stored Procedure Call
       (Update Watermark)


