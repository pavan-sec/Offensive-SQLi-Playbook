# Identification Techniques & Detection 

To successfully execute an SQL Injection attack, you must first map the attack surface (Identification) and then analyze how the backend reacts to your probes (Detection). 

Think of a web application like a heavily fortified building. **Identification** is the process of finding every single door, window, and ventilation shaft. **Detection** is the process of sliding a lockpick into those openings and listening closely to hear how the internal tumblers react.

---

## 🔍 Part 1: Identification Techniques (Mapping the Attack Surface)

Every single piece of data that travels from a user's web browser to the backend server is a potential injection point. Beginners only look at the URL bar; professionals look at the entire HTTP request.

### 1. The "Visible" Surface: GET & POST Parameters
These are the front doors. They are the most obvious and frequently tested injection points.
* **GET Parameters:** Data passed directly in the URL. 
  * *Visual:* You see `http://target.com/products?category=shoes&id=15`. The `shoes` and `15` are direct lines to the database.
* **POST Parameters:** Data sent in the body of an HTTP request, typically through HTML forms.
  * *Visual:* When you type a username and password into a login box and click "Submit", that data is packaged into the request body before hitting the database query.

### 2. The "Invisible" Surface: HTTP Headers
These are the side windows. Browsers automatically send background metadata to the server with every request. If the developer logs this metadata into a database without sanitizing it, you have an injection point.
* **User-Agent:** Tells the server what browser you are using (e.g., Mozilla/5.0). 
* **Referer:** Tells the server what website you clicked a link from to get here.
* **X-Forwarded-For:** Used to identify the originating IP address.
* **Cookies:** Session trackers stored in your browser. 
  * *The Attack:* You intercept the request in Burp Suite and change your User-Agent to `Mozilla/5.0' OR SLEEP(10)--+`. If the server is logging User-Agents to a database, the sleep command executes.

### 3. The "Machine" Surface: API Endpoints (JSON / XML)
Modern web apps (especially Single Page Applications like React or Angular) communicate with the backend using hidden API calls formatted as JSON or XML.
* *Visual:* Instead of a standard web form, the browser quietly sends a block of data like `{"username": "admin", "role": "user"}`. You can inject directly into these JSON values: `{"username": "admin' OR 1=1--", "role": "user"}`.

### 4. Second-Order Injection (The Delayed Trap)
Sometimes, the injection point isn't where the payload executes. 
* *Visual:* You register a new account and set your username to `admin'--`. The application safely stores this in the database. But tomorrow, when the administrator logs in and searches for your profile, the application pulls `admin'--` out of the database and accidentally executes it in *their* search query. 

---

## 🎧 Part 2: Detection (Listening to the Database)

Once you have identified an opening, you "poke" it with a special character (like a single quote `'`) to break out of the developer's intended string. The way the database reacts tells you exactly what kind of vulnerability you are dealing with.

### 1. Verbose Errors (The Database Screams)
When you inject a `'`, you disrupt the grammatical structure of the SQL query. If the application is poorly configured, it will print the database's internal error directly to your screen.
* **What it looks like:** `Microsoft OLE DB Provider for SQL Server error '80040e14' Unclosed quotation mark after the character string '''.`
* **What it means:** You have struck gold. The database is actively talking to you. You can use this feedback loop to craft Error-Based or UNION-Based extraction payloads.

### 2. Behavioral Anomalies (The Database Stumbles)
Sometimes the application hides the ugly database errors, but the page still reacts strangely when the query breaks behind the scenes.
* **The 500 Internal Server Error:** You input a `'` and the webpage entirely crashes, giving you a generic "Something went wrong" page. But when you remove the `'`, it works perfectly. This proves your input is violently breaking the backend logic.
* **Missing Content:** You input a `'` and the page loads smoothly, but a specific paragraph of text is missing, or the "Recommended Products" carousel is suddenly blank. 
* **What it means:** The application is suppressing errors, but the logic is still vulnerable. You are dealing with a **Boolean-Based Blind** vulnerability. You must now use True/False statements to manipulate that missing content.

### 3. Timing Anomalies (The Database Freezes)
This is the ultimate test for a completely silent application. The webpage suppresses errors, and the content looks 100% identical no matter what you type.
* **The Test:** You inject a payload designed to halt the database engine, such as `'; WAITFOR DELAY '0:0:10'--`.
* **What it looks like:** You hit enter, and instead of loading instantly, your browser's loading spinner spins for exactly 10 seconds before the page appears.
* **What it means:** The application is completely blind, but you have successfully hijacked the execution thread. You are dealing with a **Time-Based Blind** vulnerability and must extract data by timing the server's responses.
