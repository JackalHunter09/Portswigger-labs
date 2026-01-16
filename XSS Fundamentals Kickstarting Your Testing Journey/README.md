# Lab: DOM XSS in document.write sink using location.search

**Difficulty:** Apprentice  
**Type:** DOM-based XSS  
**Goal:** Make `alert()` pop up using only the URL (no server reflection!)

* Lab Link : https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink

Analogy (super simple)
Imagine the website is a chef who blindly copies whatever recipe note you hand him and immediately starts cooking with it.
The note says: "Make an image with this ingredient list: [your words]".
If you write "nothing] <b>I'm a hacker!</b>" the chef doesn't question it — he just pastes it raw into the kitchen → suddenly your bold text appears on the plate.
That's DOM XSS: client-side JS takes your URL input and pastes it raw into the page with document.write().


## What actually happened? (Super simple version)

The website has JavaScript code that looks something like this:

```js
document.write('<img src="/tracker.gif?searchTerms=' + location.search + '">');

* location.search = whatever comes after ? in the URL (you control this!)
* document.write() = super dangerous! It injects raw HTML straight into the page.
* Your input ends up inside an img src= attribute → that's the context.

How to solve it (step-by-step) 

1) Basic test:
?search=testmarker<>"'
→ See raw < > " ' in the injected <img src=...> in Elements tab (not escaped).

2) Breakout payload (closes src attribute + injects code):
?search="><svg onload=alert(1)>
→ Closes src="..." with " → closes <img> with > → adds <svg onload=alert(1)>
→ onload fires immediately → alert(1) pops.

* Proof / Impact
Arbitrary JavaScript execution in victim's browser → can steal cookies (alert(document.cookie)), redirect, keylog, etc.
Real-world: Attacker sends phishing link with this URL → victim clicks → JS runs in their session.

* Fix (developer side)
Never use document.write(), innerHTML, or direct string concatenation with user input.
Use safe methods: createElement(), textContent, or sanitize input. 

Key Lesson
DOM XSS is client-side only — the payload never touches the server.
location.search gives you raw input → sinks like document.write() are extremely dangerous.

