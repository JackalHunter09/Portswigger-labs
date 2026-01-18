# PortSwigger Web Security Academy - Solved Lab

## Lab: DOM XSS in jQuery anchor href attribute sink using location.search source

**Difficulty:** Apprentice  
**Type:** DOM-based XSS  
**Sink:** jQuery `.attr("href", ...)`  
**Source:** `location.search` (via `returnPath` query parameter)  
**Goal:** Make the "Back" link execute `alert(document.cookie)` on click

Lab Link: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink

### Super Simple Analogy
The website has a "Back" button like a magic door.  
Instead of a fixed destination, the JavaScript takes the note you left on the door (`?returnPath=your-text`) and says:  
"Set the door's destination (`href`) to whatever is on the note."  


If you write `javascript:alert(1)`, clicking the door runs your code instead of going to a normal page — that's DOM XSS via `javascript:` URI in `href`.

### What Actually Happened?
The page uses jQuery to set the href of the back link:

```js
$("a.back").attr("href", new URLSearchParams(location.search).get('returnPath') || '/');


* returnPath from the URL is taken raw.
* .attr("href", ...) sets it directly — no escaping.
* If returnPath starts with javascript:, clicking runs JS in the victim's context.



Exploit PayloadURL:

?returnPath=javascript:alert(document.cookie)

Or with leading / (sometimes needed for browser quirks):

?returnPath=/javascript:alert(document.cookie)

* Loads page normally.
* Back link's href becomes javascript:alert(document.cookie).
* Click "Back" → executes JS → alert pops with cookie → solved!

Proof of Concept
* Input: ?returnPath=javascript:alert(document.cookie)
* Result: Back link executes alert(document.cookie) on click.
* Impact: Steal cookies, keylog, redirect, or deface in victim's session.

Key Lessons
* javascript: URIs in href = code execution on click — huge risk in attribute sinks.
* jQuery .attr() is dangerous with user input — no automatic escaping.
* DOM XSS = client-side only — payload in URL, no server reflection needed.
* Browser quirks (leading /, encoding) often required in labs/real exploits.

