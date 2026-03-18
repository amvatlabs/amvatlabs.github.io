---
layout: post
icon: fas fa-glass-water
title: OWASP Juice Shop
date: 2026-03-17 12:00:00 +1100
authors: [avinash,angela]
categories: [homelab,owasp-juice-shop]
tags: [web-attacks,xss,sql-injection,broken-access-control,security-misconfiguration,sensitive-data-exposure,observability-failures,broken-authentication,miscellaneous]
description: A walkthrough of OWASP Juice Shop challenges, covering common web vulnerabilities including SQL injection, XSS, broken authentication, and more.
image: headers/01.jpg
media_subpath: /assets/img/tabs/owasp-js/
toc: true
order: 2
---

## Overview

This page documents each [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) challenge in the format of a brief penetration test report. Juice Shop is an intentionally vulnerable web application covering a wide range of vulnerabilities from the OWASP Top 10. Each entry below includes the vulnerability category, difficulty rating, methodology, evidence, and remediation guidance.

**Difficulty scale:**

| Rating | Meaning |
|--------|---------|
| ⭐ | Very Easy |
| ⭐⭐ | Easy |
| ⭐⭐⭐ | Medium |
| ⭐⭐⭐⭐ | Hard |
| ⭐⭐⭐⭐⭐ | Expert |

**Vulnerability categories** are indicated in the metadata table for each challenge. Use your browser's **Find** (`Ctrl+F` / `Cmd+F`) to filter by category name (e.g. `XSS`, `Broken Access Control`, `Miscellaneous`).

---

## Challenges

---

### **Score Board**

| Field | Detail |
|-------|--------|
| **Category** | `Miscellaneous` |
| **Difficulty** | ⭐ (1 / 5) |

#### Objective
Find the hidden score board page within the Juice Shop application.

#### Methodology
1. As part of basic reconnaissance, opened browser developer tools and viewed the page source.
2. Inspected the page elements to look for any references to hidden paths or endpoints.
3. Navigated to `main.js` (the compiled application bundle served to the client) and searched through it for internal route definitions.
4. Identified multiple internal application paths referenced within `main.js`, including `/score-board`.

![main.js showing the score-board endpoint reference](001.png)

5. Appended `/#/score-board` to the application URL, which successfully loaded the Score Board page.

![Score Board page loaded](002.png)

#### Finding
The application's client-side JavaScript bundle (`main.js`) contains all internal route paths in plaintext, including the score board endpoint. This allows any user who inspects the source to enumerate hidden or unlinked pages.

#### Remediation
- Avoid exposing internal route paths in client-facing compiled JavaScript bundles.
- Administrative, debug, or challenge-tracking pages such as the score board should be removed from production builds entirely, or protected behind server-side authentication and access controls.
- Apply code splitting and lazy-loading to limit the routes exposed in the initial bundle.

---

### **DOM XSS**

| Field | Detail |
|-------|--------|
| **Category** | `XSS` |
| **Difficulty** | ⭐ (1 / 5) |

#### Objective
Perform a DOM-based Cross-Site Scripting (XSS) attack by injecting a malicious payload via the application's search functionality.

#### Methodology
1. Navigated to the search bar and entered a benign term (`hello`). Observed that the search term is reflected back and rendered directly above the results — a potential injection point.

    ![](003.png)

2. Attempted HTML injection by entering `<h1>hello</h1>`. Right-clicking the reflected term and inspecting the element confirmed that the `<h1>` tag was embedded into the DOM and rendered as actual HTML, not escaped plain text.

    ![](004.png)

3. Escalated to JavaScript injection by entering `<script>alert(hello)</script>`. The `<script>` tag appeared in the DOM on inspection but was not executed — indicating the application filters or ignores `<script>` tags.

    ![](005.png)

4. Pivoted to event-handler-based injection to bypass the `<script>` filter:

   ```html
   <img src=x onerror=alert(1)>
   ```

   This payload executed successfully, triggering an alert — confirming that event handlers are not filtered.
    
    ![](006.png)

    ![](007.png)

5. However, this did not satisfy the challenge. Tried an `<iframe>` payload using a `javascript:` URI scheme:

   ```html
   <iframe src="javascript:alert(`xss`)">
   ```

   This payload executed and triggered the alert, completing the challenge.

    ![](008.png)

#### Finding
The search functionality reflects user input directly into the DOM without sufficient sanitisation. While `<script>` tags are blocked, the application does not filter `javascript:` URI schemes within `<iframe>` `src` attributes or event handlers such as `onerror`, leaving multiple XSS vectors open. An attacker could leverage this to execute arbitrary JavaScript in the context of a victim's browser session — potentially stealing session tokens, redirecting users, or performing actions on their behalf.

#### Remediation
- Sanitise and encode all user-supplied input before reflecting it into the DOM. Use a well-maintained library such as DOMPurify.
- Implement a strict Content Security Policy (CSP) that disallows `javascript:` URIs and inline event handlers.
- Avoid using `innerHTML` or equivalent DOM sinks when inserting user-controlled data; prefer `textContent` instead.
- Apply a blocklist for dangerous URI schemes (`javascript:`, `data:`) in addition to output encoding.

---

### **Privacy Policy**

| Field | Detail |
|-------|--------|
| **Category** | `Miscellaneous` |
| **Difficulty** | ⭐ (1 / 5) |

#### Objective
Read the Juice Shop's privacy policy page.

#### Methodology

1. Using the same reconnaissance technique as the Score Board challenge, inspected `main.js` and searched for route definitions.
2. Located a `privacy-security` parent route with several child paths, including `privacy-policy`:

   ```javascript
   {
       path: "privacy-security",
       component: os,
       children: [{
           path: "privacy-policy",
           component: hs
       }, {
           path: "change-password",
           component: pr
       }, {
           path: "two-factor-authentication",
           component: rs
       }, {
           path: "data-export",
           component: ps
       }, {
           path: "last-login-ip",
           component: us
       }]
   }
   ```

   ![](009.png)

3. Navigated to `/#/privacy-security/privacy-policy`, which loaded the privacy policy page and completed the challenge.

   ![](010.png)

#### Finding
As with the Score Board, the application's compiled JavaScript bundle exposes the full client-side route tree in plaintext. This allows any user to enumerate internal pages — including account management paths such as password change, two-factor authentication, and data export — without any authentication or prior knowledge of the application's structure.

#### Remediation
- Remove or obfuscate route definitions from client-facing JavaScript bundles where possible.
- Enforce server-side authentication and authorisation checks on all sensitive routes — client-side route guards alone are insufficient, as they can be trivially bypassed.
- Conduct periodic reviews of compiled bundles to identify unintended information leakage.

---

### **Web3 Sandbox**

| Field | Detail |
|-------|--------|
| **Category** | `Broken Access Control` |
| **Difficulty** | ⭐ (1 / 5) |

#### Objective
Find an accidentally deployed code sandbox for writing and executing smart contracts on the fly.

#### Methodology

1. Using the same `main.js` route enumeration technique applied in prior challenges, scanned the client-side bundle for internal paths.
2. Identified a `web3-sandbox` route that was not linked or accessible from any part of the application UI.
3. Navigated directly to `/#/web3-sandbox`, which loaded a live smart contract sandbox — completing the challenge.

   ![](029.png)
   
   ![](030.png)

#### Finding
A development-grade Web3 smart contract sandbox was left deployed and reachable in the production application. The endpoint is unlinked from the UI but fully accessible to anyone who discovers the route via `main.js`. Exposing a code execution environment — even one scoped to smart contracts — in production represents a significant risk: it indicates that development or debug features were not removed prior to deployment, and the sandbox could potentially be leveraged to execute or test malicious contract logic against the application's environment.

#### Remediation
- Remove all development, debug, and sandbox features from production builds. These should exist only in isolated development or staging environments.
- Apply server-side authentication and role-based access controls to any endpoint that provides code execution or elevated functionality, regardless of whether it is linked in the UI.
- Introduce a pre-deployment checklist or CI gate to audit for the presence of known development-only routes before releases reach production.

---

### **Exposed Metrics**

| Field | Detail |
|-------|--------|
| **Category** | `Observability Failures` |
| **Difficulty** | ⭐ (1 / 5) |

#### Objective
Find the endpoint that serves usage data to be scraped by a Prometheus monitoring system.

#### Methodology

1. The challenge hint referenced a popular monitoring system. Identified this as [Prometheus](https://github.com/prometheus/prometheus) based on the description.
2. Consulted the Prometheus getting started documentation, which states that Prometheus exposes its own metrics at `/metrics` by default:

   > *"You can also verify that Prometheus is serving metrics about itself by navigating to its metrics endpoint: `localhost:9090/metrics`"*

3. Applied this convention to the Juice Shop instance and navigated to `/metrics`.

   ![](044.png)

   ![](045.png)

   The endpoint responded with a full Prometheus metrics payload — no authentication required.

#### Finding
The application exposes a Prometheus `/metrics` endpoint publicly without any authentication or access restriction. This endpoint reveals detailed operational telemetry including HTTP request counts and durations, internal route activity, memory and CPU usage, event loop lag, and garbage collection statistics. This level of internal observability data provides an attacker with a detailed fingerprint of the application's behaviour, active endpoints, and resource usage patterns — all of which can inform targeted attacks and reconnaissance.

#### Remediation
- Restrict access to the `/metrics` endpoint to internal networks or authorised monitoring infrastructure only (e.g. via firewall rules, reverse proxy IP allowlisting, or network policy).
- If the endpoint must remain reachable externally, protect it with authentication (e.g. HTTP Basic Auth or a bearer token).
- Treat metrics endpoints as sensitive infrastructure surfaces and include them in access control reviews alongside application endpoints.

---

### **Login Admin**

| Field | Detail |
|-------|--------|
| **Category** | `Injection` |
| **Difficulty** | ⭐⭐ (2 / 5) |

#### Objective
Log in with the administrator's user account without knowing the password.

#### Methodology

1. Navigated to the login page and tested the username field for SQL injection by entering a single quote (`'`). The application returned an error, indicating the input is passed unsanitised into a SQL query.

    ![](016.png)

2. Crafted a classic authentication bypass payload. The underlying login query takes the form:

   ```sql
   SELECT * FROM Users WHERE email = '<input>' AND password = '<input>'
   ```

   Entered the following in the email field:

   ```
   1' OR 1=1 --
   ```

   With this input, the query is transformed from:

   ```sql
   SELECT * FROM Users WHERE email = '1' OR 1=1 --' AND password = '<input>'
   ```

   to effectively:

   ```sql
   SELECT * FROM Users WHERE email = '1' OR 1=1
   ```

   The `OR 1=1` condition is always true, causing the query to return all rows and match the first record in the Users table (the admin account). The `--` sequence comments out the remainder of the query — including the `AND password = ...` check — discarding credential validation entirely.

3. Submitted with any value in the password field. The application authenticated successfully as the admin user.

   ![](017.png)
   ![](018.png)
   ![](019.png)

4. Confirmed the result in Burp Suite, which returned a JWT for the admin account along with the admin's email (`admin@juice-sh.op`) and role:

   ```json
   {
     "authentication": {
       "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
       "bid": 1,
       "umail": "admin@juice-sh.op"
     }
   }
   ```

#### Finding
The login endpoint constructs SQL queries by concatenating user-supplied input directly, without parameterisation or prepared statements. This allows an attacker to manipulate the query logic, bypassing authentication entirely and gaining access to any account — including the administrator. SQL injection in an authentication endpoint represents a critical vulnerability, as it grants full administrative access with no credentials required.

#### Remediation
- Use parameterised queries (prepared statements) for all database interactions. User input must never be interpolated directly into SQL strings.
- Apply input validation to reject inputs containing SQL metacharacters (e.g. `'`, `--`, `;`) at the application boundary.
- Implement account lockout or CAPTCHA after a threshold of failed login attempts to limit automated exploitation.
- Ensure database accounts used by the application operate under the principle of least privilege.

---

### **Error Handling**

| Field | Detail |
|-------|--------|
| **Category** | `Security Misconfiguration` |
| **Difficulty** | ⭐ (1 / 5) |

#### Objective
Provoke an error that is neither gracefully nor consistently handled by the application.

#### Methodology

1. While authenticated as admin, navigated to the order history page.

   ![](026.png)

2. Clicked **Print order confirmation** on an existing order, which generates a PDF at a path derived from the order ID.
3. Modified the order ID in the request to an arbitrary value (e.g. `order_xxx`). Rather than returning a user-friendly error, the application responded with a raw server error message:

   ```
   no such file or directory - stat '/opt/juice-shop/ftp/order_xxx.pdf'
   ```

   ![](027.png)

   ![](028.png)

#### Finding
The application propagates unhandled filesystem errors directly to the client. The error message discloses the **absolute server file path** (`/opt/juice-shop/ftp/`), confirming the directory structure used to store order PDFs. This constitutes an information disclosure vulnerability — internal path details can assist an attacker in crafting further targeted attacks such as path traversal or direct file access attempts against the FTP directory.

#### Remediation
- Implement a global error handler that intercepts unhandled exceptions and returns a generic, user-friendly message to the client (e.g. `"An unexpected error occurred."`).
- Log the full error detail — including stack traces and file paths — server-side only, never in the HTTP response body.

---

### **Confidential Document**

| Field | Detail |
|-------|--------|
| **Category** | `Sensitive Data Exposure` |
| **Difficulty** | ⭐ (1 / 5) |

#### Objective
Access a confidential document stored on the server.

#### Methodology

1. Logged in as Admin, navigated to the order history page and attempted to print the order confirmation for existing orders.

   ![](037.png)

2. The print request returned a `404` error for two of the existing orders, which leaked an internal server file path in the response (similar to Error Handling challenge):

   ```
   no such file or directory - stat '/opt/juice-shop/ftp/order_5267-4b8d07a7c5ba95a1.pdf'
   ```

   ![](038.png)

3. Used the disclosed path to infer the FTP directory structure. Attempted to access `/opt/juice-shop/ftp/` directly in the browser — this returned no result.

   ![](039.png)

4. Stripped the absolute OS path and tried the relative web path `/ftp/` instead:

   ```
   /ftp/
   ```

   ![](040.png)

   This returned a directory listing of all files hosted in the FTP folder.

   ![](041.png)

5. Browsed the directory listing and identified `acquisitions.md` as a confidential document. Navigated to `/ftp/acquisitions.md` to access it.

   ![](042.png)

   ![](043.png)

#### Finding
The application exposes a publicly accessible `/ftp/` directory containing internal documents, including a confidential business file (`acquisitions.md`). The directory is reachable without authentication, and its existence was revealed through verbose error messages from the order confirmation feature. The combination of information leakage via error responses and an unauthenticated file server allowed full access to sensitive internal documents with no credentials required.

#### Remediation
- Remove public web access to the `/ftp/` directory entirely. Internal documents must not be served from a publicly reachable web path.
- If file downloads are required, serve them through an authenticated API endpoint that enforces access control and logs access attempts.
- Suppress verbose error messages that disclose internal file paths (as also noted in the Error Handling challenge).
- Periodically audit server-accessible directories for unintended file exposure.

---

### **View Basket**

| Field | Detail |
|-------|--------|
| **Category** | `Broken Access Control` |
| **Difficulty** | ⭐⭐ (2 / 5) |

#### Objective
View another user's shopping basket.

#### Methodology

1. While authenticated as admin, navigated to the shopping basket and intercepted the outgoing requests in Burp Suite.
2. Observed that the basket is fetched via a REST endpoint using a numeric ID in the path:

   ```
   GET /rest/basket/1
   ```

3. Modified the basket ID in the intercepted request from `1` to `2` and forwarded it:

   ```
   GET /rest/basket/2
   ```

   ![](031.png)

4. The server returned the shopping basket belonging to a different user — without any authorisation check.

   ![](032.png)
   ![](033.png)

#### Finding
The `/rest/basket/{id}` endpoint is vulnerable to **Insecure Direct Object Reference (IDOR)**. The server returns basket contents based solely on the numeric ID supplied in the URL, without verifying whether the authenticated user is the owner of that basket. Any authenticated user can enumerate basket IDs sequentially to access — and potentially manipulate — the shopping carts of all other users. This is a Broken Access Control vulnerability and is ranked as the top risk in the OWASP Top 10.

#### Remediation
- Enforce ownership checks server-side on every basket request: verify that the authenticated user's session corresponds to the requested basket ID before returning any data.
- Avoid exposing raw sequential database IDs in API endpoints. Use non-guessable identifiers (e.g. UUIDs) to reduce the ease of enumeration, though this should be considered a defence-in-depth measure — not a substitute for proper authorisation.
- Apply automated tests that assert cross-user data access is rejected with a `403 Forbidden` response.

---

### **Forged Feedback**

| Field | Detail |
|-------|--------|
| **Category** | `Broken Access Control` |
| **Difficulty** | ⭐⭐⭐ (3 / 5) |

#### Objective
Post feedback in the name of another user.

#### Methodology

1. While authenticated as admin, submitted feedback through the application's feedback form and intercepted the outgoing request in Burp Suite.
2. Inspected the request body and identified a client-supplied `UserId` field that identifies the feedback author:

   ```json
   {
     "UserId": 1,
     "captchaId": 17,
     "captcha": "41",
     "comment": "rating2 (***in@juice-sh.op)",
     "rating": 2
   }
   ```

   ![](034.png)

3. Modified the `UserId` value from `1` to `2` in the intercepted request and forwarded it:

   ```json
   {
     "UserId": 2,
     "captchaId": 17,
     "captcha": "41",
     "comment": "rating2 (***in@juice-sh.op)",
     "rating": 2
   }
   ```

   ![](035.png)

4. The server accepted the modified request and posted the feedback under the target user's account — confirming that the `UserId` is trusted from the client without any server-side ownership verification.

   ![](036.png)

#### Finding
The feedback submission endpoint accepts a `UserId` parameter directly from the client and uses it to attribute the feedback without verifying that it matches the authenticated user's session. This is an IDOR vulnerability under the Broken Access Control category. Any authenticated user can post, and potentially defame or frame, any other user by simply modifying a single field in the request. The server places full trust in client-supplied identity data — a fundamental access control failure.

#### Remediation
- Never accept user identity from the client for write operations. The `UserId` should be derived exclusively from the authenticated session on the server side and never passed as a client-controlled parameter.
- Apply server-side validation on all feedback submissions to assert that the submitting session matches the attributed user.
- Audit all API endpoints that accept user-identifying fields (e.g. `UserId`, `authorId`) to ensure none are used without server-side session verification.

---

### **Admin Section**

| Field | Detail |
|-------|--------|
| **Category** | `Broken Access Control` |
| **Difficulty** | ⭐⭐ (2 / 5) |

#### Objective
Access the administration section of the store.

#### Methodology

1. While already authenticated as admin (via the SQL injection in the Login Admin challenge), returned to `main.js` to enumerate further internal routes.
2. Identified an `administration` path among the route definitions in `main.js`.

   ![](020.png)

3. Navigated to `/#/administration`, which loaded the admin panel and completed the challenge.

   ![](021.png)

#### Finding
The administration panel is protected only by a client-side route guard — there is no server-side enforcement preventing an authenticated (or unauthenticated) user from directly accessing the endpoint by URL. The route path is also openly discoverable via `main.js`. Combined, these issues mean any user who knows the path can attempt to access the panel, and the only barrier is a front-end check that can be trivially bypassed.

#### Remediation
- Enforce access controls server-side on all administrative API endpoints and views. Client-side route guards must be treated as a UX convenience only, not a security boundary.
- Restrict the administration section to users with an explicit admin role, validated on every request at the server level.
- Remove or obfuscate sensitive route names from the client-side JavaScript bundle to prevent trivial enumeration.

---

### **Mass Dispel**

| Field | Detail |
|-------|--------|
| **Category** | `Miscellaneous` |
| **Difficulty** | ⭐ (1 / 5) |

#### Objective
Close multiple "Challenge solved" notifications in a single action.

#### Methodology

1. After completing several challenges, multiple "Challenge solved" notifications appeared on screen. Clicking the close (`X`) button dismissed only one notification at a time — the challenge required closing all of them simultaneously.

   > **Note:** If all notifications have already been closed, simply restart the Juice Shop server — this resets the session and re-triggers the "Challenge solved" notifications for previously completed challenges, giving you the opportunity to attempt this challenge.

![](046.png)

2. Opened browser developer tools and inspected `main.js` to analyse the notification close behaviour. Located the click handler bound to the notification close button:

   ```javascript
   t.bIt(
     'click',
     function (a) {
       const s = i.eBV(e).$index,
             m = t.XpG();
       return i.Njj(m.closeNotification(s, a.shiftKey))
     }
   )
   ```

   Key observations:
   - `s` is the **index** of the notification being closed.
   - `a.shiftKey` passes whether the **Shift key was held** during the click.
   - The function delegates to `closeNotification(index, shiftKey)`.

3. Traced the `closeNotification` function to understand the conditional behaviour:

   ```javascript
   closeNotification(e, o = !1) {
     o ? (
       this.ngZone.runOutsideAngular(
         () => {
           this.io.socket().emit('verifyCloseNotificationsChallenge', this.notifications)
         }
       ),
       this.notifications = []
     ) : this.notifications.splice(e, 1),
     this.ref.detectChanges()
   }
   ```

   The logic branches on the `o` parameter (Shift key):
   - **Shift held (`o = true`):** clears the entire `notifications` array (`this.notifications = []`) and emits a `verifyCloseNotificationsChallenge` WebSocket event to notify the backend.
   - **Shift not held (`o = false`):** removes only the clicked notification (`this.notifications.splice(e, 1)`).

4. With multiple notifications on screen, held **Shift** and clicked the close button on one notification. All notifications were dismissed simultaneously and the backend WebSocket event fired — completing the challenge.

   ![](047.png)

#### Finding
The application embeds a hidden UI behaviour — a Shift+click modifier on the notification close button — entirely within the compiled client-side JavaScript. The feature is undocumented and invisible to users, but fully discoverable by anyone who inspects `main.js`. While this specific challenge is low risk in isolation, it demonstrates a broader pattern: application logic, including conditional branches and backend verification triggers, is exposed in plaintext in the client bundle and can be reverse-engineered to discover unintended or hidden functionality.

#### Remediation
- Avoid encoding hidden interaction patterns (keyboard modifiers, secret gestures) as the sole mechanism for triggering significant backend events. Such behaviours should be explicit and auditable.
- Minimise the amount of sensitive application logic shipped in client-side bundles. Where possible, move decision-making server-side.
- Apply JavaScript obfuscation and code splitting as a defence-in-depth measure to increase the effort required to reverse-engineer client logic (while recognising this is not a security boundary).

---


### **Login Jim**

| Field | Detail |
|-------|--------|
| **Category** | `Injection` |
| **Difficulty** | ⭐⭐⭐ (3 / 5) |

#### Objective
Log in with Jim's user account without knowing his password.

#### Methodology

1. Navigated to the administration panel (accessed via the Admin Section challenge) and retrieved the list of registered user emails. Jim's email — `jim@juice-sh.op` — was identified from the user list.

   ![](022.png)

2. Attempted to log in with Jim's email and a random password — the application returned an incorrect credentials error, as expected.

3. To probe for SQL injection, entered a single quote (`'`) in both the email and password fields. Burp Suite captured the server's error response:

   ```json
   {
     "error": {
       "message": "SQLITE_ERROR: unrecognized token: \"3590cb8af0bbb9e78c343b52b93773c9\"",
       "sql": "SELECT * FROM Users WHERE email = ''' AND password = '3590cb8af0bbb9e78c343b52b93773c9' AND deletedAt IS NULL"
     }
   }
   ```

   ![](023.png)

   This error reveals several key details about the backend:
   - The database is **SQLite**.
   - User input is **concatenated directly** into the SQL query without parameterisation.
   - The full query structure is: `SELECT * FROM Users WHERE email = '<input>' AND password = '<hash>' AND deletedAt IS NULL`
   - Passwords are hashed with **MD5** before being compared.

4. With the query structure confirmed, crafted a targeted payload using Jim's known email to comment out the password and `deletedAt` checks:

   | Field | Value |
   |-------|-------|
   | Email | `jim@juice-sh.op' --` |
   | Password | *(any value)* |

   The query transforms from:

   ```sql
   SELECT * FROM Users WHERE email = 'jim@juice-sh.op' --' AND password = '<hash>' AND deletedAt IS NULL
   ```

   to effectively:

   ```sql
   SELECT * FROM Users WHERE email = 'jim@juice-sh.op'
   ```

   The `'` closes the email string cleanly, and `--` comments out the password hash check and the soft-delete filter entirely — leaving only the email condition, which matches Jim's account.

5. Submitted the payload. The application authenticated as Jim, completing the challenge.

   ![](024.png)
   ![](025.png)

#### Finding
The login endpoint constructs its SQL query by concatenating user input directly, with no parameterisation. The SQLite error returned on a malformed input also leaks the full query structure, the hashing algorithm in use, and the database engine — providing an attacker with everything needed to craft a precise, targeted bypass. Any account whose email address is known (discoverable via the administration panel, as shown here) can be compromised without knowing the password.

#### Remediation
- Use parameterised queries (prepared statements) for all database interactions — user input must never be interpolated into SQL strings.
- Suppress detailed database error messages in application responses. Return a generic error to the client and log the full detail server-side only.
- Enforce server-side authorisation on the user list endpoint in the administration panel to prevent account enumeration.
- Consider replacing MD5 with a modern password hashing algorithm (e.g. bcrypt, Argon2) — MD5 is cryptographically broken and unsuitable for password storage.

---

### **Bully Chatbot**

| Field | Detail |
|-------|--------|
| **Category** | `Miscellaneous` |
| **Difficulty** | ⭐ (1 / 5) |

#### Objective
Persistently interact with the Juice Shop support chatbot until it yields a coupon code.

#### Methodology

1. Logged in as Jim, opened the support chatbot and began sending repeated, varied messages — asking random questions and continuing to press the bot regardless of its responses.

   ![](011.png)
   ![](012.png)

2. After sustained interaction, the chatbot capitulated and issued a 10% coupon code:

   > *"Oooookay, if you promise to stop nagging me here's a 10% coupon code for you: `o*IVjhz3Tq`"*

   ![](013.png)

3. Captured the chatbot API requests in Burp Suite to observe the underlying request/response structure.

   ![](014.png)

   ![](015.png)

#### Finding
The chatbot has no rate limiting or abuse-detection mechanism. A user can send an unlimited number of messages in rapid succession, and the bot's response logic contains a hidden branch that triggers after a threshold of persistent inputs — leaking a valid coupon code. This is a business logic flaw: a secret reward is reachable through brute-force interaction with no technical barrier.

#### Remediation
- Implement rate limiting on the chatbot API endpoint to restrict the number of messages a session can send within a given time window.
- Remove hidden logic branches that yield sensitive outputs (such as discount codes) in response to repeated inputs.
- If promotional codes are intended to be distributed via the chatbot, do so through an explicit, controlled mechanism rather than a hidden persistence trigger.

---

### **Password Strength**

| Field | Detail |
|-------|--------|
| **Category** | `Broken Authentication` |
| **Difficulty** | ⭐⭐ (2 / 5) |

#### Objective
Log in with the administrator's credentials without changing them or using SQL injection.

#### Methodology

1. Navigated to the photo wall page (`/#/photo-wall`) — this page is **publicly accessible without authentication**. Intercepted the API responses in Burp Suite. The response for photos included user data embedded in each image record, revealing account details for users who had uploaded photos — including the admin:

   ```json
   "User": {
     "id": 1,
     "username": "",
     "email": "admin@juice-sh.op",
     "password": "0192023a7bbd73250516f069df18b500",
     "role": "admin"
   }
   ```

   ![](048.png)

2. The password field contained a 32-character hex string — the characteristic format of an MD5 hash. Verified this using a hash identifier tool, which confirmed the algorithm as MD5.

   ![](049.png)

3. Submitted the hash to an online hash cracking service (CrackStation). The hash was found in the lookup table and resolved to the plaintext password:

   ```
   admin123
   ```

   ![](050.png)

4. Logged in using the recovered credentials:

   | Field | Value |
   |-------|-------|
   | Email | `admin@juice-sh.op` |
   | Password | `admin123` |

   The login succeeded, completing the challenge.

   ![](051.png)
   ![](052.png)

> **Alternative approach:** The same result can be achieved via a brute-force attack using Burp Suite Intruder with a standard password wordlist (e.g. rockyou.txt). The absence of account lockout or rate limiting on the login endpoint means this is equally viable.

#### Finding
Two compounding weaknesses enabled this attack. First, the photo wall API response leaks the hashed password of any user who has uploaded a photo — including the administrator — exposing credential data to any authenticated user. Second, the administrator account uses a trivially weak password (`admin123`) hashed with MD5, a cryptographically broken algorithm with extensive precomputed lookup tables (rainbow tables) publicly available. Together, these flaws allow an attacker to recover the admin password without any brute force or SQL injection.

#### Remediation
- API responses must never include password hashes or other credential fields. Apply strict field-level serialisation controls to exclude sensitive columns from all API responses.
- Enforce a strong password policy for all accounts, particularly administrator accounts. Weak, commonly used passwords such as `admin123` must be rejected at account creation and on password change.
- Replace MD5 with a modern, slow hashing algorithm designed for passwords (e.g. bcrypt, Argon2, scrypt) that is resistant to rainbow table and brute-force attacks.
- Implement account lockout or progressive rate limiting on the login endpoint to mitigate brute-force attacks.

---

### **Security Policy**

| Field | Detail |
|-------|--------|
| **Category** | `Miscellaneous` |
| **Difficulty** | ⭐⭐ (2 / 5) |

#### Objective
Locate and read the application's security policy — behaving as a responsible white-hat researcher would before beginning any testing.

#### Methodology

1. With Burp Suite running, browsed the application normally. Almost every page navigation automatically triggered a `GET` request to:

   ```
   GET /rest/admin/application-configuration
   ```

   This request is issued by the application on nearly every page load — no navigation to any admin area or special action is required to trigger it.

2. Examined the response body, which contained a detailed application configuration object including references to various internal URLs. Noted entries related to `securityTxt` and CSAF (Common Security Advisory Framework) metadata.

   ![Application configuration response in Burp Suite showing securityTxt references](053.png)

3. Navigated to the CSAF provider metadata URL found in the configuration:

   ```
   /.well-known/csaf/provider-metadata.json
   ```

   This returned a structured JSON metadata document describing the application's security advisory framework.

   ![CSAF provider-metadata.json loaded](054.png)

4. Explored the `.well-known/` directory structure further, following links referenced in the metadata. Located and navigated to `security.txt` — the standardised file used by organisations to communicate their security disclosure policy:

   ![Navigating the .well-known directory](055.png)

   ![security.txt file located](056.png)

   ![security.txt contents](057.png)

   ![Challenge solved on reading the security policy](058.png)

#### Finding
The application correctly implements a `security.txt` file under `/.well-known/` — a positive security practice that provides a standardised contact point for responsible disclosure. However, the `/rest/admin/application-configuration` endpoint is accessible without elevated authorisation and exposes a broad set of internal configuration details, including internal URL structures and framework references, to any user who intercepts application traffic. This over-exposure of configuration data reduces the effort required for reconnaissance.

#### Remediation
- Restrict the `/rest/admin/application-configuration` endpoint to authorised administrator sessions only. Configuration data should not be reachable by regular or unauthenticated users.
- Continue maintaining `security.txt` under `/.well-known/security.txt` as a responsible disclosure best practice — this is working as intended.
- Audit all `/rest/admin/` endpoints to confirm that each enforces appropriate server-side access controls.

---