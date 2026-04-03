# SQLi Global Reference & Cheat Sheet

**Purpose:** A centralized repository for cross-platform syntax, fingerprinting techniques, and common bypasses.

---

## Database Fingerprinting

Before choosing a payload, identify the backend database using these characteristic "tells":

| DB Type | String Concatenation | Comment Style | Version Query |
| --- | --- | --- | --- |
| **PostgreSQL** | `'a' || 'b'` | `--` | `SELECT version()` |
| **MySQL** | `'a' 'b'` or `CONCAT()` | `#` or `-- ` (space) | `SELECT @@version` |
| **Oracle** | `'a' || 'b'` | `--` | `SELECT banner FROM v$version` |
| **MS SQL** | `'a' + 'b'` | `--` | `SELECT @@version` |

---

## Technique-Specific Syntax

### 1. UNION Extraction (In-Band)

*Requirement: The number of columns must match the original query.*

* **Oracle:** Must append `FROM dual` to every `SELECT`.
* **MySQL/Postgres:** `FROM` is optional for static values.
* **Column Discovery:** Use `ORDER BY X--` or `UNION SELECT NULL, NULL--`.

### 2. Inferential (Blind) Functions

Functions used to ask "Yes/No" questions about the data.

| Feature | PostgreSQL / Oracle | MySQL | MS SQL |
| --- | --- | --- | --- |
| **Substring** | `SUBSTR(str, start, len)` | `SUBSTRING(str, start, len)` | `SUBSTRING(str, start, len)` |
| **Length** | `LENGTH(str)` | `LENGTH(str)` | `LEN(str)` |
| **ASCII Val** | `ASCII('a')` | `ASCII('a')` | `ASCII('a')` |

### 3. Time-Based Delays (Blind)

Used when the application suppresses both content changes and error messages.

* **PostgreSQL:** `pg_sleep(10)`
* **MySQL:** `SLEEP(10)`
* **Oracle:** `dbms_pipe.receive_message('a',10)`
* **MS SQL:** `WAITFOR DELAY '0:0:10'`

---

## Common Technical Caveats

* **Trailing Spaces:** MySQL comments (`-- `) **require** a space after the dashes. If the space is trimmed by the browser, use `--+` or `#`.
* **Oracle Metadata:** Always remember that table and column names in `all_tables` and `all_tab_columns` are **case-sensitive** and usually stored in `UPPERCASE`.
* **URL Encoding:** When injecting via the URL, always encode special characters:
* `#` → `%23`
* `&` → `%26`
* `+` → `%2b`

---

### **SQL String Comparison (Lexicographical Order)**

SQL compares strings **character-by-character** from left to right based on their **ASCII/numeric values**. It stops at the first character that differs.

| Rank | Category | Sample ASCII Values | Comparison Example |
| --- | --- | --- | --- |
| **1 (Lowest)** | **Special Chars** | Space (32), `!` (33) | `' ' < '1'` is **TRUE** |
| **2** | **Numbers** | `'0'` (48) ... `'9'` (57) | `'9' < 'A'` is **TRUE** |
| **3** | **Uppercase** | `'A'` (65) ... `'Z'` (90) | `'Z' < 'a'` is **TRUE** |
| **4 (Highest)** | **Lowercase** | `'a'` (97) ... `'z'` (122) | `'z'` is the highest |

---

## Useful Metadata Tables

* **Postgres/MySQL:** `information_schema.tables` / `information_schema.columns`
* **Oracle:** `all_tables` / `all_tab_columns` / `user_tables`
* **MS SQL:** `sys.tables` / `sys.columns`