# # 04 - Prevention & Mitigation (Defending the Database)

---

As an offensive security engineer, you cannot effectively bypass a defense if you do not understand how it is constructed. This document covers the industry-standard methodologies for preventing SQL Injection (SQLi) and mitigating the potential damage if a breach occurs.
---
## 1. The Gold Standard: Parameterized Queries (Prepared Statements)

If a developer implements Parameterized Queries correctly, traditional SQL Injection is virtually impossible. 

**The Concept:** Instead of dynamically gluing user input directly into a SQL string, the developer writes the SQL query first, compiles it, and *then* inserts the user input as strict parameters. The database engine treats the input exclusively as literal data, never as executable code.

** Vulnerable to SQLi via String Concatenation :**
php
// PHP Example
$username = $_POST["username"];
// The input is glued directly into the query
$sql = "SELECT * FROM users WHERE username = '" . $username . "'";
$db->query($sql);

*If $username is ' OR 1=1--, the query evaluates to SELECT * FROM users WHERE username = '' OR 1=1--.*

** Secure via Prepared Statements:**

PHP

// PHP Data Objects (PDO) Example
$username = $_POST["username"];
// 1. Prepare the statement with a placeholder (?)
$stmt = $db->prepare("SELECT * FROM users WHERE username = ?");
// 2. Execute the statement and pass the variable safely
$stmt->execute([$username]);

*If $username is ' OR 1=1--, the database looks for a literal user named [' OR 1=1--]. The malicious logic is completely neutralized.*

---

## 2. Input Validation & Type Casting

While Prepared Statements are the primary defense, validating the data before it ever touches the database provides crucial Defense-in-Depth.

### A. Strict Type Casting

If a parameter is expecting a specific data type (like a numeric `id`), the backend code should force the input to conform to that type before executing the query.

- *Example:* If an attacker injects `?id=5' UNION...`, strict integer casting will drop the string portion and evaluate it simply as `5`, neutralizing the attack.

### B. Allow-Listing vs. Block-Listing

- **Block-Listing (BAD):** Trying to filter out "bad" characters like `'`, `"`, or keywords like `SELECT` and `UNION`. Attackers can easily bypass these using URL encoding, HEX encoding, or uppercase/lowercase tricks (e.g., `sElEcT`).
- **Allow-Listing (GOOD):** Defining exactly what *is* allowed. For example, if the input is a US Zip Code, the code should reject anything that isn't exactly 5 numeric digits.

---

## 3. Escaping Input (The Legacy Fix)

Before Prepared Statements became the universal standard, developers used escaping functions to neutralize special characters.

- *Example:* PHP's `mysqli_real_escape_string()`.
- *How it works:* If an attacker inputs a single quote (`'`), the function automatically adds a backslash (`\'`) in front of it. The database reads this as a literal quote character, not a code breakout.
- **Why it's no longer recommended:** It is highly prone to human error. If a developer forgets to escape even one input field on an entire website, the database is compromised.

---

## 4. Web Application Firewalls (WAF)

A WAF sits in front of the web application and analyzes incoming HTTP traffic.

- **How it helps:** It looks for known SQLi signatures (like `UNION SELECT` or `1=1`) in the URL, Headers, or POST body and blocks the malicious request before it ever reaches the application server.
- **The Attacker's Perspective:** WAFs are "band-aids." They do not fix the underlying vulnerability in the application's source code. A skilled attacker will spend time crafting payloads designed to bypass the WAF's regex filters (e.g., using SQL comments `/**/` to break up keywords like `U/**/NION S/**/ELECT`).

---

## 5. The Principle of Least Privilege

Mitigation is about limiting the blast radius if an attacker *does* successfully bypass the defenses and find a SQLi vulnerability.

- **The Mistake:** Connecting the web application to the database using the default `sa` (System Admin) or `root` account. If SQLi is found here, the attacker can drop the entire database, read/write local OS files, and potentially achieve Remote Code Execution (RCE).
- **The Fix:** Create a specific, low-privileged database user dedicated solely to the web application.
    - It should only have `SELECT`, `INSERT`, and `UPDATE` permissions.
    - It should have `DELETE` privileges explicitly revoked.
    - It should be strictly forbidden from accessing system directories or `information_schema` tables unless absolutely required by the application logic.

