---
description: Using variables in Bash
---

# Variables

## Basic Variable Declarations

Variables are named places to temporarily store data.

The easiest method is to set the value directly with a simple `name=value` declaration.

* Notice that there are no spaces before or after the "`=`" sign

```bash
kali@kali:~$ first_name=Jim
```

Variables are later referenced with "`$`" and the variable name

```bash
kali@kali:~$ first_name=Jim
kali@kali:~$ last_name=Smith
kali@kali:~$ echo $first_name $ last_name
Jim Smith
```

Variable names may be uppercase, lowercase, or a mixture of both. However, Bash is case-sensitive so one must be consistent when declaring and expanding variables.

* As most Bash environment variables are uppercase (e.g. `$PATH`) it is generally recommended to user lowercase with underscores (`_`) as variable names

When declaring string variables (particularly ones with spaces in them) single (`'`) or double quotation (`"`) marks may be used, though Bash treats the two of them differently.

* When encountering single quotes, Bash interprets every enclosed character literally
* When enclosed in double quotes, all characters are viewed literally except `$`, `` ` ``, and _`\`_ meaning variables will be expanded in an initial substitution pass on the enclosed text

```bash
kali@kali:~$ greeting=Hello World
bash: World: command not found

kali@kali:~$ greeting='Hello World'
kali@kali:~$ echo $greeting
Hello World

kali@kali:~$ greeting2="New $greeting"
kali@kali:~$ echo $greeting2
New Hello World
```

## Command Substitution

_Command substitution_ is used to set the value of the variable to the result of a command or program.

It is important to note that command substitution happens in a subshell and changes to variables in the subshell will not alter variables from the master process.

{% code lineNumbers="true" %}
```bash
kali@kali:~$ cat ./subshell.sh
#!/bin/bash -x

var1=value1
echo $var1

var2=value2
echo $var2

$(var1=newvar1)
echo $var1

`var2=newvar2`
echo $var2

kali@kali:~$ ./subshell.sh 
+ var1=value1
+ echo value1
value1
+ var2=value2
+ echo value2
value2
++ var1=newvar1
+ echo value1
value1
++ var2=newvar2
+ echo value2
value2
```
{% endcode %}

Notice the added `-x` flag in the example. This adds debug information making it easier to see the variable  being declared in a subshell (line 26 var2 is overwritten but the original value is retained in line 28).&#x20;

Shell vs. subshell is indicated by the `+` sign(s)

* Lines shown with a single `+` sign were executed in the shell
* Lines shown with double `++` signs were executed in the subshell

### Parentheses Syntax

The basic syntax for command substitution is to place the command inside parentheses preceded by a dollar sign `$(command)`.

```bash
kali@kali:~$ user=$(whoami)
kali@kali:~$ echo $user
kali
```

### Backtick Syntax

An alternative syntax for command substitution using the backtick, or grave, character `` `command` ``. The backtick **method is older and typically discouraged** as there are differences in how the two methods of command substitution behave.

```bash
kali@kali:~$ user2=`whoami`
kali@kali:~$ echo $user2
kali
```
