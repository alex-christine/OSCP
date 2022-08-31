---
description: Performing boolean logic operations in Bash
---

# Boolean Logic

## Logical Operators

| Operator | Description                                                   |
| -------- | ------------------------------------------------------------- |
| `&&`     | <p>Logical and<br><br>Also used in <em>command lists</em></p> |
| `\|\|`   | Logical or                                                    |

Logical operators can be tricky because Bash uses them in a variety of ways

## And (`&&`)

### Logic

Performs the typical logical and operation (all conditions must be `true`)

#### Example

{% code title="and.sh" %}
```bash
#/bin/bash
# and example

if [ $USER == 'kali' ] && [ $HOSTNAME == 'kali' ]
then
  echo "Multiple statements are true!"
else
  echo "Not much to see here..."
fi
```
{% endcode %}

```bash
kali@kali:~$ ./and.sh 
Multiple statements are true!
```

### Command List

Used to execute multiple commands within a single line.&#x20;

Executes a command **only if the previous command succeeds** (returns `True` or `0`).

#### Example

```bash
kali@kali:~$ user2=kali
kali@kali:~$ grep $user2 /etc/passwd && echo "$user2 found!"
kali:x:1000:1000:,,,:/home/kali:/bin/bash
kali found!
```

## Or (`||`)

### Logic

Used in the traditional logic understanding (if a OR b THEN c).

#### Example

{% code title="or.sh" %}
```bash
#!/bin/bash
# or example

if [ $USER == 'kali' ] || [ $HOSTNAME == 'pwn' ]
then
  echo "One condition is true, this line is printed"
else
  echo "You are out of luck!"
fi
```
{% endcode %}

```bash
kali@kali:~$ echo $USER && echo $HOSTNAME
kali
kali

kali@kali:~$ ./or.sh
One condition is true, this line is printed
```

### Command List

When used in a command list, the _OR_ (`||`) operator is the opposite of _AND_ (`&&`); it executes the next command **only if the previous command failed** (returned `False` or non-zero).

#### Example

```bash
kali@kali:~$ echo $user2
bob

kali@kali:~$ grep $user2 /etc/passwd && echo "$user2 found!" || echo "$user2 not found!"
bob not found!
```

* When `grep` fails to find a match for `$user2` (bob) it returns `false` and the second `echo` command is executed
