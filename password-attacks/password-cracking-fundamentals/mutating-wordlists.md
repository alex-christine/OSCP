---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Mutating Wordlists

Password policies, which have grown in prevalence in recent years, dictate a minimum password length and the use of character derivations including upper and lower case letters, special characters, and numerical values.

Most passwords in the commonly-used wordlists will not fulfill these requirements. This can be addressed by automating the process of changing (or **mutating**) the wordlist before sending them to a particular target in what is known as a **rule-based attack**.

In this type of attack, individual rules are implemented through rule functions, which are used to modify existing passwords contained in a wordlist. An individual rule consists of one or more rule functions. Multiple rule functions will often be used in each rule.

In order to leverage a rule-based attack, one must create a rule file containing one or more rules and use it with a cracking tool.

## Hashcat Examples

Some examples are transcribed below, but the [Hashcat Wiki](https://hashcat.net/wiki/doku.php?id=rule\_based\_attack) provides a list of all possible rule functions with examples.

### Simple Transformations

A simple example would be appending a fixed set of characters to all passwords in a list. For this example, assume the attacker is **facing a password policy that requires an upper case letter, a special character, and a numerical value**.

```shell-session
kali@kali:~$ mkdir passwordattacks

kali@kali:~$ cd passwordattacks

kali@kali:~/passwordattacks$ head /usr/share/wordlists/rockyou.txt > demo.txt

kali@kali:~/passwordattacks$ sed -i '/^1/d' demo.txt 

kali@kali:~/passwordattacks$ cat demo.txt
password
iloveyou
princess
rockyou
abc123
```

The `sed` command is used to remove any sequences that are all numeric digits (e.g. "`123456`"). The resulting list of passwords still do not conform to the password policy but can be easily modified to do so.

The simplest way to add characters to a password is to prepend or append them to the existing password (though more complex rules can certainly be created).

One can use the `$` function to append a character or `^` to prepend a character. Both of these functions expect one character after the function selector. For example, in order to prepend a "3" to every password in a file, the corresponding rule function would be `^3`.

When generating a password with a numerical value, many users just add a "1" to the end of their password. In order to create a rule to append a "1" the attacker needs to create a rule file containing `$1` (note the `$` needs to be escaped with a `\` in the `echo` command in order to function as expected):

```shell-session
kali@kali:~/passwordattacks$ echo \$1 > demo.rule
```

This rule file can then be used with `hashcat` in order to create a new wordlist:

```shell-session
kali@kali:~/passwordattacks$ cat demo.rule     
$1

kali@kali:~/passwordattacks$ hashcat -r demo.rule --stdout demo.txt
password1
iloveyou1
princess1
rockyou1
abc1231
```

* `-r` indicates the rule file
* `--stdout` starts Hashcat in debugging mode. In this mode Hashcat will not attempt to crack any hashes, but merely display the mutated passwords

As shown above, this rule successfully appends a "1" to all passwords in the file.

#### Multiple Rule Files

Consider an example with 2 separate rule files, `demo1.rule` and `demo2.rule`. These will be used to illustrate how rules are interpreted by Hashcat.

Both of these examples will make use of the `c` rule function, which capitalizes the first character and converts the rest to lower case..&#x20;

In `demo1.rule`, the rule functions are on the same line separated by a space. In this case, Hashcat will use them consecutively on each password of the wordlist. The result is that the first character of each password is capitalized _and_ a "1" is appended to each password.

In `demo2.rule` the rule functions are on separate lines. Hashcat interprets the second rule function, on the second line, as new rule. In this case, each rule is used separately, resulting in _two mutated passwords_ for every password from the wordlist:

```shell-session
kali@kali:~/passwordattacks$ cat demo1.rule     
$1 c
       
kali@kali:~/passwordattacks$ hashcat -r demo1.rule --stdout demo.txt
Password1
Iloveyou1
Princess1
Rockyou1
Abc1231

kali@kali:~/passwordattacks$ cat demo2.rule   
$1
c

kali@kali:~/passwordattacks$ hashcat -r demo2.rule --stdout demo.txt
password1
Password
iloveyou1
Iloveyou
princess1
Princess
...
```

Using `demo1.rule`, two of the three password criteria have been met. The only one remaining is adding a special character. Luckily 2 new rule files can be used to achieve this:

{% code title="demo3.rule" %}
```
c $1 $!
```
{% endcode %}

{% code title="demo4.rule" %}
```
c $! $1
```
{% endcode %}

Which result in the following patterns:

```shell-session
kali@kali:~/passwordattacks$ hashcat -r demo3.rule --stdout demo.txt
Password1!
Iloveyou1!
Princess1!
Rockyou1!
Abc1231!


kali@kali:~/passwordattacks$ hashcat -r demo4.rule --stdout demo.txt
Password!1
Iloveyou!1
Princess!1
Rockyou!1
Abc123!1
```

Now that the password rules have been understood it is time to create a rule file that uses multiple rules for the file:

{% code title="demo5.rule" %}
```
$1 c $!
$2 c $!
$1 $2 $3 c $!
```
{% endcode %}

For each password in the wordlist this will generate 3 mutated passwords:

1. Capitalized, ending in `1!`
2. Capitalized, ending in `2!`
3. Capitalized, ending in `123!`

Assuming an MD5 password hash has been retrieved and is stored in hash.txt:

{% code title="hash.txt" %}
```
f621b6c9eab51a3e2f4e167fee4c6860
```
{% endcode %}

It can now be cracked via Hashcat:

```shell-session
kali@kali:~/passwordattacks$ hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt -r demo5.rule --force
hashcat (v6.2.5) starting
...
Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 43033155

f621b6c9eab51a3e2f4e167fee4c6860:Computer123!            
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: f621b6c9eab51a3e2f4e167fee4c6860
Time.Started.....: Tue May 24 14:34:54 2022, (0 secs)
Time.Estimated...: Tue May 24 14:34:54 2022, (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Mod........: Rules (demo3.rule)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  3144.1 kH/s (0.28ms) @ Accel:256 Loops:3 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
...
```

## crunch

[crunch](https://www.kali.org/tools/crunch/) is a command-line tool that can be used to help build a custom dictionary, usually based on a known portion of the password, for a brute force attack. This tool generally requires attackers to have a rough idea of the password's "base" and is mainly helpful for automatically adding formatted characters.

### Example

Consider an attacker who has reason to suspect the user `eve`'s password contains "`lab`" as its base. Let's say (in a highly simplified example) the password policy was 6 character minimum, one uppercase, one lowercase, and one digit. Combining this knowledge with the suspected base of the password, crunch could generate password guesses as follows:

```shell-session
$ crunch 6 6 -t Lab%%% > wordlist    
Crunch will now generate the following amount of data: 7000 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 1000
```

Resulting in the following file:

{% code title="wordlist" %}
```
Lab000
Lab001
Lab002
Lab003
Lab004
Lab005
Lab006
Lab007
Lab008
Lab009
...
```
{% endcode %}

This can then be fed into hydra as a regular wordlist:

```shell-session
$ hydra -l eve -P wordlist 192.168.210.214 ssh -o eve.hydra                        
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
...
[DATA] attacking ssh://192.168.210.214:22/
...
[22][ssh] host: 192.168.210.214   login: eve   password: Lab123
1 of 1 target successfully completed, 1 valid password found
...
```

While this is a simple example, more information about rule construction can be found using `man crunch`.
