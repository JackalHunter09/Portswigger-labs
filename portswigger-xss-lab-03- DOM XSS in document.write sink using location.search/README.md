* Lab Writeup: DOM XSS in document.write sink using location.searchLab: DOM XSS in document.write sink using location.search
* Difficulty: Apprentice
* Type: DOM-based XSS
* Goal: Trigger alert() using only the URL parameter (no server-side reflection)


Lab Link : https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink


Analogy (super simple)
Imagine the website is a chef who blindly copies whatever recipe note you hand him and immediately starts cooking with it.
The note says: "Make an image with this ingredient list: [your words]".
If you write "nothing] <b>I'm a hacker!</b>" the chef doesn't question it — he just pastes it raw into the kitchen → suddenly your bold text appears on the plate.
That's DOM XSS: client-side JS takes your URL input and pastes it raw into the page with document.write().

What actually happened?
The page has JavaScript that does this (simplified):js

document.write('<img src="/resources/images/tracker.gif?searchTerms=' + location.search.substring(1) + '">');

* location.search = everything after ? in the URL (you fully control it).
* document.write() injects the string directly as HTML — no escaping, no sanitization.
* Our input ends up inside the src attribute → perfect place to break out of the quotes and inject our own tags/events.

How to solve it (step-by-step)  
1) Basic test:
?search=testmarker<>"'
→ See raw < > " ' in the injected <img src=...> in Elements tab (not escaped).

2) Breakout payload (closes src attribute + injects code):
?search="><svg onload=alert(1)>
→ Closes src="..." with " → closes <img> with > → adds <svg onload=alert(1)>
→ onload fires immediately → alert(1) pops.

Proof / Impact
Arbitrary JavaScript execution in victim's browser → can steal cookies (alert(document.cookie)), redirect, keylog, etc.
Real-world: Attacker sends phishing link with this URL → victim clicks → JS runs in their session.

Fix (developer side)
Never use document.write(), innerHTML, or direct string concatenation with user input.
Use safe methods: createElement(), textContent, or sanitize input.

Key Lesson
DOM XSS is client-side only — the payload never touches the server.
location.search gives you raw input → sinks like document.write() are extremely dangerous.

