# Lab 02: Stored XSS into HTML context with nothing encoded

**Difficulty:** Apprentice  
**Goal:** Submit a comment that executes `alert(1)` when the blog post is viewed (stored/persistent XSS)

## Vulnerability
The comment field is stored in the database and reflected directly into the HTML of the blog post **without any HTML escaping/encoding**.

# Lab Link : https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded

## Steps to solve
1. Go to any blog post.
2. In **Comment**: `<script>alert(1)</script>`
3. **Name**: anything
4. **Email**: anything@valid.com
5. **Website**: `https://example.com` (must include `https://` or validation fails with "Please match the requested format")
6. Click **Post comment**
7. Refresh / go back to the blog → alert pops when comments render.

## Key learning
- Stored XSS: Payload lives on the server → affects every visitor.
- Client-side validation (e.g. URL format) can be bypassed in real life with Burp, but here we just follow it.
- Only the comment field is vulnerable (others are encoded).

# What is "client-side validation"?
It's a check that happens inside your browser (using JavaScript), before the data even leaves your computer and goes to the server.In this Stored XSS lab:The "Website" field has a rule: "Must look like a real URL" (starts with http:// or https://, has a domain, etc.).
JavaScript in the page says: "Nope! You didn't put https:// → I won't even let you click Post comment → show error 'Please match the requested format'".

This is client-side = the browser is the "police officer" stopping you.


# Why is it easy to bypass in real hacking (with Burp Suite)?
Because client-side validation is 100% under your control — you can just turn off the police officer!
How to do it:
* Turn on Burp Proxy (intercept mode)
* Fill the form normally (or with invalid data)
* When you click "Post comment" → Burp catches the request before it reaches the server
* In Burp → you edit the "website" parameter to whatever you want (e.g. javascript:alert(1) or even your XSS payload)
* Forward the request → server receives the dirty data, no browser police stopped it!

The server might not care about the format (many don't re-check), so the bad data gets stored → vulnerability triggered.This is a very common mistake in real websites: developers trust the browser too much and don't re-validate on the server.


Bottom line:
In real attacks → always assume client-side rules are fake → use Burp to bypass them.
In this beginner lab → just play nice with the rule → focus on the real vulnerability (the comment field).



**Solved:** January 13/14, 2026 🚀