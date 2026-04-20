# 💉 Offensive SQL Injection Playbook

> A comprehensive, methodology-first guide to understanding, fingerprinting, and exploiting SQL Injection vulnerabilities across database architectures.

## 📖 Overview
This repository serves as a documented methodology of my hands-on training in manual SQL Injection. Rather than relying on automated tools, the focus here is on understanding the underlying database mechanics, crafting manual payloads, and developing a systematic approach to data extraction.

This playbook is designed to shift the focus from "throwing payloads at a wall" to structured, inferential database interrogation.

## 🗂️ Repository Structure

### 📁 1-SQLi-Basics
Foundational knowledge covering the core concepts of relational databases, SQL syntax, and how backend logic processes user input.
* **`01_What-is-SQL.md`**: Core definitions and the mechanics of query manipulation.
* **`02_What-of-SQLi.md`**: Breakdown of In-band, Inferential (Blind), and Out-of-band SQLi.
* **`03_Identification-Techniques.md`**: Methods for locating injection points in web applications.
* **`04_Prevention-and-Mitigation.md`**: Defensive strategies (Parameterized queries, WAFs, input sanitization).

### 📁 2-Cheat-Sheets
Custom-built reference guides mapping out syntax differences across major database engines (MySQL, MSSQL, PostgreSQL, Oracle, SQLite).
* **`1-Error-Based.md`**: Payloads for triggering verbose database errors.
* **`2-Union-Based.md`**: Syntax for column counting and string concatenation.
* **`3-Blind-SQLi.md`**: Boolean payloads for True/False inferential testing.
* **`4-Time-Based.md`**: Stopwatch/Sleep commands across all major databases.
* **`5-Miscellaneous.md`**: Additional edge cases, WAF bypasses, and encoding tricks.

### 📁 3-Attack-Workflows
Step-by-step diagnostic flowcharts for identifying and exploiting specific SQLi categories.
* **`1-Error-Based-Workflow.md`**: Extracting data directly through forced syntax errors.
* **`2-Union-Attack-Workflow.md`**: The complete chain for in-band extraction (from `ORDER BY` to `information_schema`).
* **`3-Blind-SQLi-Workflow.md`**: The diagnostic tree for Boolean-based extraction.
* **`4-Time-Based-Workflow.md`**: Execution flow for time-delay data extraction.
* **`5-Automation-Scripts.md`**: Custom scripts and notes for automating tedious extraction processes.

### 📁 4-Lab-Writeups
Highlight reports detailing the exploitation process for specific training scenarios. *(Note: Exact flags and target URLs are redacted to respect the integrity of the training platforms.)*

### 📁 5-References
A curated collection of external tools, documentation, and resources utilized during the research and testing phases.

## 🧠 Core Philosophy
A successful penetration test requires understanding the system better than the people who built it. The methodologies documented here emphasize:
1. **Patience over Noise:** Mapping the target surface before attempting extraction.
2. **Precision Execution:** Tailoring payloads specifically to the fingerprinted database engine.
3. **Manual Validation:** Confirming vulnerabilities manually to eliminate false positives before considering automation.

## ⚠️ Disclaimer
All techniques, payloads, and methodologies documented in this repository are strictly for **educational purposes** and authorized penetration testing. Do not execute these payloads against systems you do not own or have explicit, written permission to test.

## 🔗 Connect With Me
* **Author:** Baddula Pavan Kumar
* **LinkedIn:** https://www.linkedin.com/in/baddula-pavan-kumar/