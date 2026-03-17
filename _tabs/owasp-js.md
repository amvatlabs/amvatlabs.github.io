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

### Score Board

| Field | Detail |
|-------|--------|
| **Category** | `Miscellaneous` |
| **Difficulty** | ⭐ (1 / 5) |

**Objective**
Find the hidden score board page within the Juice Shop application.

**Methodology**

1. As part of basic reconnaissance, opened browser developer tools and viewed the page source.
2. Inspected the page elements to look for any references to hidden paths or endpoints.
3. Navigated to `main.js` (the compiled application bundle served to the client) and searched through it for internal route definitions.
4. Identified multiple internal application paths referenced within `main.js`, including `/score-board`.

![main.js showing the score-board endpoint reference](001.png)

5. Appended `/#/score-board` to the application URL, which successfully loaded the Score Board page.

![Score Board page loaded](002.png)

**Finding**
The application's client-side JavaScript bundle (`main.js`) contains all internal route paths in plaintext, including the score board endpoint. This allows any user who inspects the source to enumerate hidden or unlinked pages.

**Remediation**
- Avoid exposing internal route paths in client-facing compiled JavaScript bundles.
- Administrative, debug, or challenge-tracking pages such as the score board should be removed from production builds entirely, or protected behind server-side authentication and access controls.
- Apply code splitting and lazy-loading to limit the routes exposed in the initial bundle.

---
