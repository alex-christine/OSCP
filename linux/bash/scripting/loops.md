---
description: Loop types in Bash
---

# Loops

## For Loop

### Basic Syntax

The for loop general syntax is roughly similar to any other programming language:

```bash
for var-name in <list>
do
  <action to perform>
done
```

The `for` loop will:

1. Take each item in the `list` (in order) and assign that item as the value of the variable `var-name`
2. Perform the given action between `do` and `done`
3. Go back to the top, grab the next item in the list, and repeat the steps until the list is exhausted

#### Example

In this example a for loop will be used to generate a list of IPs in a specific subnet.

```bash
kali@kali:~$ for ip in $(seq 1 10); do echo 10.11.1.$ip; done
10.11.1.1
10.11.1.2
10.11.1.3
10.11.1.4
10.11.1.5
10.11.1.6
10.11.1.7
10.11.1.8
10.11.1.9
10.11.1.10
```

* `seq` command to print a sequence of numbers (in this case, one through ten)

### Brace Expansion

Another way of writing for loops involves _brace expansion_ with **ranges**.

* Brace expansion using ranges is written giving the **first and last values of the range**
  * Can be a sequence of numbers or characters
  * This is known as a **sequence expression**

#### Example

```bash
kali@kali:~$ for i in {1..10}; do echo 10.11.1.$i;done
10.11.1.1
10.11.1.2
10.11.1.3
10.11.1.4
10.11.1.5
10.11.1.6
10.11.1.7
10.11.1.8
10.11.1.9
10.11.1.10
```

## While Loop

### Basic Syntax

_While_ loops are also fairly common and execute code while an expression is true. _While_ loops have a simple format and, like _if_, use the square brackets (`[]`) for the test.

```bash
while [ <some test> ]
do
  <perform an action>
done
```

#### Example

{% code title="while.sh" %}
```bash
#!/bin/bash
# while loop example

counter=1

while [ $counter -le 10 ]
do
  echo "10.11.1.$counter"
  ((counter++))
done
```
{% endcode %}

```bash
kali@kali:~$ ./while.sh
10.11.1.1
10.11.1.2
10.11.1.3
10.11.1.4
10.11.1.5
10.11.1.6
10.11.1.7
10.11.1.8
10.11.1.9
10.11.1.10
```

* The `((counter++))` line uses the double-parenthesis `(( ))` construct to perform arithmetic expansion and evaluation at the same time
