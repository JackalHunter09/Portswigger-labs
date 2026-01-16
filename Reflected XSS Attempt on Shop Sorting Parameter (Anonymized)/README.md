Target: Live WordPress + WooCommerce shop page (name blurred for safety)
Difficulty: Advanced (real-world with WAF)
Type: Reflected XSS attempt (in JSON context)
Goal: Chain reflection → execution (didn't fully succeed due to escaping/WAF)



Analogy (super simple)
Imagine a librarian who copies your book title request and writes it inside a list in a book:
"Favorite book": "Harry Potter"If you say: "Favorite book": "Harry Potter"]; alert("hacked") //
The librarian copies it raw → the book now says:
"Favorite book": "Harry Potter"]; alert("hacked") //"If the librarian didn't properly protect the quotes → the list breaks and your code runs.
That's JSON breakout XSS — user input inside a JS string that gets parsed.



What actually happened?  
Parameter: orderby (?orderby=...) — used to sort products (popularity, price, etc.).
The value is reflected raw in a client-side JSON object inside <script> tag:js

var ecs_ajax_params = { ... "posts": "{\"orderby\":\"popularity\", ...}" };

Test marker: ?orderby="><testmarker
→ Reflected raw as "orderby":"\"><testmarker" (quotes escaped with \, but < > literal).


Attempts & Results  
* Tag-based payloads ("><img src=x onerror=alert(1)>, "><script>alert(1)</script>) → 400 Bad Request (WAF/LiteSpeed + Cloudflare blocks common XSS signatures).

* Structure-closing payloads ("}];alert(1);//, popularity\"}];alert(document.cookie);//) → 200 OK + reflection, but no execution (server escapes quotes → stays inside string literal).

* Console showed no errors or alerts → payload trapped as data.



Impact & Risk  
* Current: Reflected but not executable (medium risk).
* Potential: High if escaping is removed or value is used unsafely later (e.g., innerHTML = params.orderby or eval()).
* WAF is effective against obvious attacks, but incomplete sanitization remains.


Fix Recommendation
Use json_encode() with proper flags:  

php
json_encode($query_params, JSON_HEX_TAG | JSON_HEX_APOS | JSON_HEX_QUOT | JSON_HEX_AMP);

Escape all special characters before insertion into JSON.

Key Lesson
Real sites often have layers (WAF + partial escaping) that stop basic payloads.
But reflection in JS/JSON context is still dangerous — always test breakout + structure closing.



