---
description: Use of conditional statements in Bash
---

# Conditionals

## If Statements

The `if` statement is relatively simple, it checks to see if a condition is true, but it requires a very specific syntax:

```bash
if [ <some test> ]
then
  <perform an action>
fi
```

* Pay careful attention to the use of required spaces

### Conditional Operators

| Operator             | Description                               |
| -------------------- | ----------------------------------------- |
| `!EXPRESSION`        | `EXPRESSION` is false                     |
| `-n STRING`          | `STRING` length is greater than 0         |
| `-z STRING`          | `STRING` length is 0 (empty)              |
| `STRING1 != STRING2` | `STRING1` is not equal to `STRING2`       |
| `STRING1 = STRING2`  | `STRING1` is equal to `STRING2`           |
| `INT1 -eq INT2`      | `INT1` is equal to `INT2`                 |
| `INT1 -ne INT2`      | `INT1` is not equal to `INT2`             |
| `INT1 -gt INT2`      | `INT1` is greater than `INT2`             |
| `INT1 -lt INT2`      | `INT1` is less than `INT2`                |
| `INT1 -ge INT2`      | `INT1` is greater than or equal to `INT2` |
| `INT1 -le INT2`      | `INT1` is less than equal to `INT2`       |
| `-d FILE`            | `FILE` exists and is a directory          |
| `-e FILE`            | `FILE` exists                             |
| `-r FILE`            | `FILE` exists and has read permission     |
| `-s FILE`            | `FILE` exists and is not empty            |
| `-w FILE`            | `FILE` exists and has write permission    |
| `-x FILE`            | `FILE` exists and has execute permission  |

#### Example

{% code title="if.sh" %}
```bash
#!/bin/bash
# if statement example

read -p "What is your age: " age

if [ $age -lt 16 ]
then
  echo "You might need parental permission to take this course!"
fi
```
{% endcode %}

```bash
kali@kali:~$ ./if.sh 
What is your age: 15
You might need parental permission to take this course!
```

## Else and Else-If Statements

These statements follow the same basic syntax as the `if` statement.

```bash
# Simple If-Else
if [ <some test> ]
then
  <perform action>
else
  <perform another action>
fi

# If, Else-If Statement
if [ <some test> ]
then
  <perform action>
elif [ <some test> ]
then
  <perform different action>
else
  <perform yet another different action>
fi
```

#### Example

{% code title="elif.sh" %}
```bash
#!/bin/bash
# elif example

read -p "What is your age: " age

if [ $age -lt 16 ]
then
  echo "You might need parental permission to take this course!"
elif [ $age -gt 60 ]
then
  echo "Hats off to you, respect!"
else
  echo "Welcome to the course!"
fi
```
{% endcode %}

```bash
kali@kali:~$ ./elif.sh
What is your age: 65
Hats off to you, respect!
```
