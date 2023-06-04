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

<table><thead><tr><th width="231">Operator</th><th>Description</th></tr></thead><tbody><tr><td><code>!EXPRESSION</code></td><td><code>EXPRESSION</code> is false</td></tr><tr><td><code>-n STRING</code></td><td><code>STRING</code> length is greater than 0</td></tr><tr><td><code>-z STRING</code></td><td><code>STRING</code> length is 0 (empty)</td></tr><tr><td><code>STRING1 != STRING2</code></td><td><code>STRING1</code> is not equal to <code>STRING2</code></td></tr><tr><td><code>STRING1 = STRING2</code></td><td><code>STRING1</code> is equal to <code>STRING2</code></td></tr><tr><td><code>INT1 -eq INT2</code></td><td><code>INT1</code> is equal to <code>INT2</code></td></tr><tr><td><code>INT1 -ne INT2</code></td><td><code>INT1</code> is not equal to <code>INT2</code></td></tr><tr><td><code>INT1 -gt INT2</code></td><td><code>INT1</code> is greater than <code>INT2</code></td></tr><tr><td><code>INT1 -lt INT2</code></td><td><code>INT1</code> is less than <code>INT2</code></td></tr><tr><td><code>INT1 -ge INT2</code></td><td><code>INT1</code> is greater than or equal to <code>INT2</code></td></tr><tr><td><code>INT1 -le INT2</code></td><td><code>INT1</code> is less than equal to <code>INT2</code></td></tr><tr><td><code>-d FILE</code></td><td><code>FILE</code> exists and is a directory</td></tr><tr><td><code>-e FILE</code></td><td><code>FILE</code> exists</td></tr><tr><td><code>-r FILE</code></td><td><code>FILE</code> exists and has read permission</td></tr><tr><td><code>-s FILE</code></td><td><code>FILE</code> exists and is not empty</td></tr><tr><td><code>-w FILE</code></td><td><code>FILE</code> exists and has write permission</td></tr><tr><td><code>-x FILE</code></td><td><code>FILE</code> exists and has execute permission</td></tr></tbody></table>

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
