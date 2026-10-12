# Apache NiFi Customer Data Transformation Pipeline

An automated, scalable data processing pipeline built with **Apache NiFi (v2.11.0)**. The flow ingests raw customer data, validates record structures, routes malformed records to a dedicated error directory, enriches valid data through custom scripting, and exports the finalized dataset.

---

## Architecture & Dataflow Diagram

The pipeline processes records sequentially through the following architecture:

![NiFi Dataflow Architecture](./Nifi%20dataflow.png)

### Data Processing Steps

1. **`GetFile`**
   * Listens to incoming directory and ingests raw source datasets (`.csv`).
2. **`QueryRecord` (Validation Stage)**
   * Validates records against structural and field-level rules (e.g., non-null fields, age criteria).
   * **Invalid Records:** Transferred to `Save Rejected Records` (`PutFile`), creating `rejected data.csv`.
   * **Valid Records:** Routed downstream for transformation.
3. **`QueryRecord2` (Filtering Stage)**
   * Filters and isolates validated customer records for downstream enrichment.
4. **`ConvertRecord`**
   * Standardizes data formats across schema definitions.
5. **`ExecuteScript` (Enrichment Stage)**
   * Runs custom scripting logic to derive dynamic metadata, such as computing the `age_group` column based on raw `age`.
6. **`Saved Cleaned Records` (`PutFile`)**
   * Persists transformed and enriched customer records into `cleaned data.csv`.

---

## Repository Structure

```text
├── Nifi dataflow.png       # Apache NiFi pipeline topology screenshot
├── cleaned data.csv        # Final enriched & transformed dataset
├── rejected data.csv       # Logged invalid / rejected records
└── README.md               # Project documentation
```

---

## Schemas & Column Definitions

### 1. Cleaned Output (`cleaned data.csv`)

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| `customer_id` | Integer | Unique identifier for the customer |
| `name` | String | Full name of the customer |
| `registration_date` | Date (`YYYY-MM-DD`) | Customer registration date |
| `age` | Integer | Validated customer age |
| `age_group` | String | Categorical classification derived via `ExecuteScript` |

### 2. Rejected Records Log (`rejected data.csv`)

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| `customer_id` | Integer | Unique identifier |
| `customer_name` | String | Raw customer name |
| `age` | Mixed / Null | Invalid or missing age value |
| `registration_date` | Mixed / Null | Missing or malformed date value |
| `email` | String | Contact email address |

---

## How to Set Up & Run in Apache NiFi

1. **Clone the Repository**
   ```bash
   git clone https://github.com/salmahamdy246/NiFi-ETL-pipeline
   cd nifi-customer-etl-pipeline
   ```

2. **Import Flow into NiFi**
   * Open Apache NiFi Canvas (`http://localhost:8080/nifi`).
   * Import the flow definition JSON/XML template into  Process Group.

3. **Update Directory Paths**
   * **`GetFile`**: Update `Input Directory` to target incoming directory.
   * **`Save Rejected Records`**: Set output path for `rejected data.csv`.
   * **`Saved Cleaned Records`**: Set output path for `cleaned data.csv`.

4. **Start Flow**
   * Enable Controller Services (Record Readers/Writers).
   * Right-click the primary Process Group and select **Start**.