---
layout: post
icon: fas fa-glass-water
title: OWASP Juice Shop
date: 2026-03-17 12:00:00 +1100
authors: [avinash, angela]
categories: [homelab, owasp-juice-shop]
tags: [web-attacks]
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

### **Bully Chatbot**

| Field | Detail |
|-------|--------|
| **Category** | `Miscellaneous` |
| **Difficulty** | ⭐ (1 / 5) |

#### Objective
Persistently interact with the Juice Shop support chatbot until it yields a coupon code.

#### Methodology

1. Opened the support chatbot and began sending repeated, varied messages — asking random questions and continuing to press the bot regardless of its responses.

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
