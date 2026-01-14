What is XSS? (Super basic version)XSS = Cross-Site Scripting
Imagine a website is like a nice friendly shop. But because of a bug, a bad guy can sneak his own tiny evil JavaScript program inside the shop's webpage.  When you (the normal customer) visit that page, your browser runs the bad guy's code — thinking it's normal code from the shop.  
Now the bad guy can:
1) Steal your login cookie (like stealing your shop membership card)
2) Pretend to be you and change your email/password
3) Read private messages
4) Show you fake login forms to steal credentials
5) Basically do anything you can do on that site

It's called "cross-site" because the bad code comes from "outside" (the attacker), but runs on the trusted site.How does XSS actually work?The website takes something you type (or someone else types) → and puts it directly into the page without cleaning it. 
Bad guy writes:
<script>alert("Haha got you!")</script>If the website just copies it as-is into the HTML → your browser sees it as real JavaScript and runs it → popup appears → vulnerability confirmed!Real attacks usually try to steal document.cookie (your session) instead of just showing a popup.


The 3 main types of XSS (very simple) :

Type - REFLECTED
Nickname - "One-shot" / Mirror
When does the bad code appear? - Immediately in the same page load (reflected back)
How does the bad guy deliver it? - Trick you to click a malicious link (phishing)
Example situation - Search box: ?search=<script>evil()</script>

Type - STORED
Nickname - "Permanent" / Memory
When does the bad code appear? - Stored in database, shows to everyone later
How does the bad guy deliver it? - Post a comment, review, message that contains the attack
Example situation - Evil comment on blog that everyone sees

Type - DOM-BASED
Nickname - "Browser-side only"
When does the bad code appear? - Never goes to server — JavaScript on page misuses data
How does the bad guy deliver it? - Usually via URL, but code runs fully in your browser
Example situation - Page takes ?name=evil and does innerHTML = name

Quick analogy:
Reflected  →  Bad guy shouts something rude  →  shop loudspeaker repeats it immediately to you.
Stored  →  Bad guy writes rude graffiti on shop wall  →  everyone who comes later sees it.
DOM-based  →  Shop has a broken mirror inside  →  it reflects your own face in a scary way, but only you see it (no server involved).

Proof of Concept (How to check "yes, it's vulnerable!")
Most common & easiest way in labs:
Make a popup appear with alert(1) or alert(document.cookie)
→ If popup shows → XSS works!

Important note from PortSwigger (2021+ change):New Chrome versions block alert() in some special cases (cross-origin iframes)
So in labs nowadays they often tell you to use print() instead → same idea, just prints the page

How to find / test for XSS? (Beginner style) :
Easy cases (reflected & stored):
1) Put a weird unique word everywhere you can type (search, comment, name…)
*Example: xyz123test
2) Look everywhere on the site → do you see your weird word appear?
3) If yes → try to break out with <script>print()</script> or similar
4) If popup → jackpot!

DOM-based is trickier:
* Put your weird word in the URL ?param=xyz123test
* Open browser developer tools (F12) → search the whole page source for your word
* See where JavaScript takes your word and writes it dangerously (innerHTML, document.write, eval…)
* These usually need looking at the JavaScript code (hard manually)

PortSwigger says:
Burp Suite Professional scanner finds almost all of them automatically (including hard DOM ones).
Community Edition → you do it manually (Repeater + trial & error) → possible, but slower.



Dangling markup injection → Sneaky cousin of XSS. When full <script> is blocked, you can still "break" the HTML so it steals data (like CSRF tokens) by leaving tags open. Less powerful than real XSS but still dangerous.

Content Security Policy (CSP) → Website's "security rules list". Tells browser: "Only allow JavaScript from these safe places". Good defense against XSS, but sometimes attackers can bypass weak CSPs.

XSS contexts → Where your input ends up (inside <p>, inside <script>, inside attribute, etc.). You need different payloads depending on the context.
