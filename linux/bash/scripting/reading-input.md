---
description: Reading user input with bash
---

# Reading Input

Focus on reading input from terminal. For notes on command line arguments see [linked](arguments.md).

## `read`

Used to capture interactive user input while a script is running.

#### Example

An incredibly simple example of using `read` to take a value from the user.

```bash
kali@kali:~$ cat ./input.sh
#!/bin/bash

echo "Hello there, would you like to learn how to hack: Y/N?"

read answer

echo "Your answer was $answer"

kali@kali:~$ chmod +x ./input.sh 

kali@kali:~$ ./input.sh
Hello there, would you like to learn how to hack: Y/N?
Y
Your answer was Y
```

### Important Flags

<table><thead><tr><th width="184">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>-p '{PROMPT}'</code></td><td>Specify a prompt to be displayed while asking for input</td></tr><tr><td><code>-s</code></td><td>Make user input silent.<br><br>Particularly useful for capturing user credentials</td></tr></tbody></table>

```bash
kali@kali:~$ cat ./input2.sh
#!/bin/bash
# Prompt the user for credentials

read -p 'Username: ' username
read -sp 'Password: ' password

echo "Thanks, your creds are as follows: " $username " and " $password

kali@kali:~$ chmod +x ./input2.sh

kali@kali:~$ ./input2.sh
Username: kali
Password: 
Thanks, your creds are as follows:  kali  and  nothing2see!
```
