## 1. What is SQLi?
**SQL Injection (SQLi)** is a web security vulnerability that allows an attacker to interfere with the queries an application makes to its database. 

By injecting malicious input, an attacker can manipulate the application's logic to bypass authentication, extract sensitive data (like passwords and credit cards), modify database records, or even escalate privileges to achieve Remote Code Execution (RCE) on the underlying server.

---

## 2. Types of SQL Injection
SQL Injection is generally categorized into three main distinct branches based on how the database returns data to the attacker.

### A. In-Band SQLi (Classic)
This is the most straightforward type of SQLi. It occurs when the attacker uses the **same communication channel** to both launch the attack and gather the results. The data is displayed directly on the web page.

* **Error-Based SQLi:** * **Description:** The attacker intentionally inputs characters that break the SQL syntax. The application then displays the raw database error directly in the HTTP response. Pentesters use these visible errors for quick enumeration to map out database structure.
    * **Example:** Injecting a single quote (`'`) into an `id` parameter results in an output like: `Syntax error in SQL statement near '''`.
* **UNION-Based SQLi:**
    * **Description:** The attacker leverages the SQL `UNION` operator to combine the results of the original, legitimate query with the results of their own injected, malicious query. It is the fastest way to dump large amounts of data.
    * **Example:** `' UNION SELECT username, password FROM users--`
    * **🚨 The Golden Rules of UNION:** Before you can successfully extract data using a UNION attack, two strict conditions must be met:
        1.  **Column Count Match:** The injected query must ask for the *exact same number of columns* as the original query. (Usually calculated using the `ORDER BY` technique).
        2.  **Data Type Match:** The data types of the corresponding columns in both queries must be compatible (e.g., you cannot inject a string into a column designed to hold an integer).

### B. Inferential SQLi (Blind)
Blind SQLi is just as dangerous as In-Band, but significantly stealthier. There is **no actual transfer of data** via the web application, and no errors are shown. The attacker must reconstruct the database information piece-by-piece by observing the application's behavior.

* **Boolean-Based (Content-Based) SQLi:**
    * **Description:** The attacker asks the database a series of True/False questions. The vulnerability is confirmed if the application's response (its content or formatting) changes depending on whether the injected condition was mathematically TRUE or FALSE.
    * **Example:** * True Query: `' AND 1=1--` (Page loads normally with all content).
        * False Query: `' AND 1=2--` (Page loads, but is missing text, images, or shows "0 results").
* **Time-Based SQLi:**
    * **Description:** When the application is completely silent and the content never changes, the attacker relies on response time. They force the database to pause (sleep) for a specified amount of time before responding. 
    * **Example:** `' AND SLEEP(5)--` (If the page takes exactly 5 seconds longer to load than usual, the vulnerability is confirmed).

### C. Out-of-Band (OOB) SQLi
* **Description:** The rarest and most advanced form. It is used when the application is completely blind (not even time delays work) because the query is executed asynchronously. The attacker forces the database server to make an external network connection (like a DNS lookup or HTTP request) to an attacker-controlled server to deliver the extracted data.

---

## 3. Basic Detection Payloads
Before launching a full extraction attack, a pentester must first identify the injection point. These are the primary payloads used to "poke" the application and observe how it handles malicious input.

| Payload | Purpose | Expected Vulnerable Result |
| :--- | :--- | :--- |
| `'` | **Syntax / Break Test** | Breaks the query. Should return a database error or a visually broken page. |
| `"` | **Escape Test** | Alternative to the single quote for breaking out of double-quote string enclosures. |
| `' OR '1'='1` | **Authentication Bypass** | Modifies the `WHERE` clause to always evaluate to True. Often returns all records or bypasses a login. |
| `1 AND 1=1` | **Boolean True Test** | The mathematical logic is true. The page should return normal results. |
| `1 AND 1=2` | **Boolean False Test** | The mathematical logic is false. The page should return empty/different results compared to the True test. |