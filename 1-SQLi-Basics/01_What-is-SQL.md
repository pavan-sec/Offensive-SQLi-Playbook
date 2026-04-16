# 01 - What is SQL? (The Attacker's Perspective)

Before you can break a system, you have to understand how it communicates. This document covers the bare minimum SQL foundations required to understand how SQL Injection (SQLi) manipulates backend architecture.

## 🏗️ The Database Ecosystem
Data doesn't just float in the void. It lives in a structured ecosystem hosted on a server:
* **Database (DB):** The actual container where data is stored.
* **DBMS (Database Management System):** The software that manages and handles requests to the database (e.g., MySQL, Microsoft SQL Server, Oracle).
* **SQL (Structured Query Language):** The language we use to speak to the DBMS. 

For SQL injection, our primary targets are **Relational Databases** (SQL-based), where data is organized cleanly into Tables, Columns, and Rows.

## 🗣️ The Vocabulary of SQL
SQL instructions are broken down into categories. As attackers, we primarily abuse **DQL** and **DML**.
1. **DQL (Data Query Language):** Used to retrieve data (`SELECT`). *This is our main tool for data exfiltration.*
2. **DML (Data Manipulation Language):** Used to modify data (`INSERT`, `UPDATE`, `DELETE`).
3. **DDL (Data Definition Language):** Used to alter the structure (`CREATE`, `ALTER`, `DROP`).

## 🎯 Key Concepts for Pentesters
When reviewing SQL foundations, these specific mechanics are the ones most frequently abused during an injection attack:

### 1. The `SELECT` and `WHERE` Clauses
The `SELECT` statement extracts data. The `WHERE` clause filters it based on conditions. 
> **Why it matters:** 90% of SQL injection occurs inside a `WHERE` clause. If a developer writes `SELECT * FROM users WHERE username = '$input'`, we can inject malicious logic into that input to force the database to return data it shouldn't (like bypassing a login screen).

### 2. Strings and Quotation Marks
Databases use variable characters (`VARCHAR`, `CHAR`) to store text like usernames and passwords. In SQL syntax, strings must be wrapped in single quotes (`'string'`).
> **Why it matters:** Throwing a single quote (`'`) into an input field is the universal first step of SQLi. It is designed to "break out" of the developer's intended string and start writing our own malicious commands.

### 3. The Semicolon (`;`)
In SQL, the semicolon is used to terminate a command. 
> **Why it matters:** In some databases (like Microsoft SQL Server and PostgreSQL), attackers can use a semicolon to end the original query and immediately start a brand new, malicious one. This is known as a **Stacked Query** (e.g., `SELECT * FROM products; DROP TABLE users--`).

### 4. `ORDER BY`
Normally used to sort a result-set in ascending (ASC) or descending (DESC) order (e.g., `ORDER BY age DESC`).
> **Why it matters:** Pentesters hijack the `ORDER BY` command to mathematically calculate the exact number of columns a table has. (e.g., `ORDER BY 1`, `ORDER BY 2` until the query breaks). 

### 5. Operators and Wildcards
SQL uses operators (`=`, `>`, `<`, `<>`) to compare data. It also uses wildcards to search for partial matches via the `LIKE` command:
* `%` (Percent): Represents zero, one, or multiple characters (e.g., `LIKE 'admin%'`).
* `_` (Underscore): Represents exactly one single character.
> **Why it matters:** When extracting data blindly, attackers use wildcards and operators to guess passwords character by character. 

---

## 📚 Further Learning
As an offensive security engineer, you do not need to be a database administrator, but a solid foundation of SQL syntax is highly recommended. To deep-dive into standard SQL queries, check out:
* [NetworkChuck's SQL](https://youtu.be/xiUTqnI6xk8?si=HVnORPFl7b1locX6) (Excellent visual breakdowns).
* [W3Schools SQL Tutorial](https://www.w3schools.com/sql/) (The ultimate quick-reference cheat sheet).
* [SQL in Telugu](https://youtu.be/XEqTRwT9cW4?si=2hrZzCa102zEvieB),[SQL in Hindi](https://youtu.be/yE6tIle64tU?si=pUj6DPjqcKITQs1J)(want to learn SQL from basics use these tutorial's).