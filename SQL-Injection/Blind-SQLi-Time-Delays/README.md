Today I learned and worked on one of the key subtopics in web application vulnerability testing: Blind SQL Injection with Time Delays and Information Retrieval from PortSwigger's Web Security Academy.Goal: Exploit a blind SQL injection vulnerability in the TrackingId cookie to steal the administrator user's password from the users table in a PostgreSQL database. Since it's blind (no query results or error messages are shown on the page), we use time delays to figure out if our injected conditions are true or false. The password is 20 characters long (lowercase letters a–z and digits 0–9). I win by extracting the full password and logging in as the administrator.
![WhatsApp Image 2026-01-12 at 19 31 50](https://github.com/user-attachments/assets/8a87eedb-3d4a-4542-96bc-3837646ebecd)

Link to the lab : https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval
How to solve the lab :
1) "Imagine there's a big estate full of houses, and each person inside has their own secret entry code (like a password) to get in.The gateman is super strict. His only job is:
Show your correct entry code → you enter.
Wrong code → you don't enter.
And he never talks! No matter what you ask, he just stands there quietly. He won't say yes, no, maybe, nothing.But someone discovered a clever trick.He said to the gateman:
'Listen, I won't ask you to talk. But if the answer to my question is YES, just wait 10 full seconds before you nod your head once.
If the answer is NO, nod your head immediately.'The gateman can't refuse this trick because he already knows how to wait/delay sometimes (like when he's checking a code or thinking). So he has to obey.Now the clever person starts asking questions:'Is there someone living here called administrator?'
→ Gateman waits 10 seconds… then nods.
→ Yes! There is!'For administrator's entry code… is the first letter a?'
→ Gateman nods immediately → No, not a.
'Is it b?' → Waits 10 seconds → Yes, it's b!And he keeps asking like this, letter by letter, until he figures out the whole 20-letter secret entry code.Then he uses that code to walk into the estate pretending to be administrator!That's exactly how the "time delay blind SQL injection" trick works.

The computer/database is the quiet gateman who never talks, but you can make him wait when the answer is yes — and that's how you steal the secret password without him ever saying a word."
a)The estate + houses = database + tables/rows (very visual).

b)Gateman who never talks = blind (no output, no errors shown)

c)The "wait 10 seconds if yes" rule = the pg_sleep(10) delay function.

d)He has to obey because "waiting" is already part of his normal job = why the database executes the sleep command.

e)Asking letter by letter = the SUBSTRING + CASE WHEN brute-forcing

f)Finally entering as the victim = logging in as administrator

2) How I started: Identifying the Vulnerability and Database Type
I used Burp Suite, a powerful tool for intercepting and modifying HTTP traffic. When you access a site on Burp Suite browser, Burp Suite intercepts the request before it reaches the server. This lets me view and edit the request (including cookies).I turned on Intercept in Burp's Proxy tab, loaded the page, and caught the GET request containing the TrackingId cookie.
To confirm injection that i can actually communicate with the database and make it respond to my queries, I modified the TrackingId to test basic boolean conditions:```text TrackingId=xyz' AND '1'='1 (always true) → page loads normally.
```text TrackingId=xyz' AND '1'='2 (always false) → page also loads normally (no visible difference or error).
In this fully blind lab, both load the same way but When you inject a single quote ', in a normal (non-blind) SQLi vuln, you'd expect a syntax error → visible error message or 500 server error, Here, since no error appears, i was thinking "maybe it's not injectable".
But the fact that the page still loads fine (no crash) after injecting ' proves the app is tolerating the quote — it's likely escaping/handling it in a way that is kind of like saying, the syntax you entered works but the app has be secured to hide/suppress any error and still return a normal 200 OK response with the usual page content but the fact that the site didnt crash just meant that yes i can communicate with it, The real value comes from the boolean tests: the code ```text' AND '1'='1 injects a condition that's always true → the query should behave like the original (normal load).  
```text ' AND '1'='2 injects a condition that's always false → if the database is executing the code, this should make the query return no rows (or change the display content page logic).
Even though the page looks the same, the fact that both payloads are accepted without crashing shows the backend is actually running the injected SQL,it's not just filtering/rejecting it.

In this fully blind lab, both load the same way but When you inject a single quote ', in a normal (non-blind) SQLi vuln, you'd expect a syntax error → visible error message or 500 server error, Here, since no error appears, i was thinking "maybe it's not injectable".
But the fact that the page still loads fine (no crash) after injecting ' proves the app is tolerating the quote — it's likely escaping/handling it in a way that is kind of like saying, the syntax you entered works but the app has be secured to hide/suppress any error and still return a normal 200 OK response with the usual page content but the fact that the site didnt crash just meant that yes i can communicate with it, The real value comes from the boolean tests: the code ' AND '1'='1 injects a condition that's always true → the query should behave like the original (normal load).  
```text' AND '1'='2 injects a condition that's always false → if the database is executing the code, this should make the query return no rows (or change the display content page logic).
Even though the page looks the same, the fact that both payloads are accepted without crashing shows the backend is actually running the injected SQL,it's not just filtering/rejecting it.

Although, it was stated in the lab description that we are dealing with a PostgreSQL Database but it's important to be sure about the exact type of Database the application is using so i would know the exact syntax to query the database with, the SQLi payload i'm injecting through the TrackingId to communicate with the database must match what the database uses or understands, you can think of it like a language used in communicating with the database, if the database only understands English and i'm trying to communicate with it in German, then it would be difficult to get a correct response, so to be sure i need to find out what type of database the app is using like MySQL / MariaDB,Microsoft SQL Server (MSSQL),Oracle,SQLite,PostgreSQL, these are the most common type of databases used in most web applications so it made sense knowing what the database it is first so its easier to craft a payload or language that matches the database syntax or let me say the language that matches the same language that the database understands before i can communicate with it, so since it's blind (no error messages leak even with a single quote '), i tested for time-delay functions specific to each major Databases, where im kind of like telling the database that if a condition is true then it should wait for 10 seconds before loading the page, so to test for MySQL / MariaDB database, i used ```textxyz' AND SLEEP(10)--, but the page loaded instantly within 197 milliseconds cause i can see the response time on Burp Suite showing in milliseconds, so that means the database i'm dealing with is not MySQL/MariaDB, so i tested to see if it was using  Microsoft SQL Server (MSSQL) database with these specific time delay function ```textxyz'; WAITFOR DELAY '0:0:10'--  , am actually editing and entering these codes in the TrackingId inside the cookie request from the intercepted traffic, so these one too loaded quickly, so i knew it was not the one, so i tried the function for Oracle database ```text xyz'||DBMS_LOCK.SLEEP(10)--, same thing too, loaded quickly in <10 seconds, so i tried for another called SQLite with this specific function ```text xyz' AND 123=LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(1000000000/2))))-- but this one too loaded the page quickly before 10 seconds so i tried the last one which is PostgreSQL with these function ```text xyz'||pg_sleep(10)-- and it responded by waiting after 10 seconds before loading the page, so now am sure the database is PostgreSQL.

Next i used these conditional delays to check if administrator user exists with these payload   ```text xyz'|| (SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)--     ,these payload means 1) (SELECT ... FROM users) → Look inside the users table.
2) CASE WHEN (condition) THEN ... ELSE ... END → Like an "if-else" statement when you are programming : If true →```text pg_sleep(10) (wait 10 seconds),If false →```text pg_sleep(0) (no delay).
3)username='administrator' → The condition we're testing.
4) Delay = true (user exists); no delay = false.

I used the same method to confirm the password column exists.

3) To optimize, I found the exact length for the whole password first with these payload: 
```textxyz'|| (SELECT CASE WHEN LENGTH(password)>N THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--    where  LENGTH(password) → is a Built-in function that counts characters in the password.
> N → "Is the password longer than N?"
I tested N = 1, 2, 3... with Burp Repeater.
Delay happens when password is longer than N; no delay when N is equal or larger.
Stopped at N=20 → password is exactly 20 characters.

4)Now the fun (slow) part: is to brute-force each position. with these  payload:

```textxyz'|| (SELECT CASE WHEN SUBSTRING(password, POS, 1)='CHAR' THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--    where 1) SUBSTRING(password, POS, 1) → means the database should take 1 character from the password at position POS.

2) POS → is the current position we're guessing (1 = first char, 2 = second, ..., up to 20).
3)='CHAR' →is like asking the database that  "Does this position match the character we're testing?" (then i start trying and changing the character value im guessing it to be from a–z,and also from  0–9).
4) then ```textCASE WHEN ... THEN pg_sleep(10) ELSE pg_sleep(0) END → means that i'm telling the database that if the first character in the password is that value i entered maybe a,b or c, then it should  wait for 10 seconds before it loads the page again but if its not that value, then it should load the page instantly.
5)WHERE username='administrator' → means i'm telling it to only check the admin row.

![WhatsApp Image 2026-01-12 at 19 32 47](https://github.com/user-attachments/assets/a7ab4797-75a0-4f7d-a77c-f2f83e8911f9)

5)so i did these to get the whole 20 characters of the password, then i went to the login page, entered administrator as username + the extracted password → logged in successfully → lab solved!
