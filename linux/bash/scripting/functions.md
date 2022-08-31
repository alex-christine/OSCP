---
description: Creating functions in the Bash environment
---

# Functions

A function is a subroutine, or a code block that implements a set of operations. Functions may be written in one of two formats:

```bash
function function_name {
commands...
}
```

```bash
function_name () {
commands...
}
```

Either format is valid though the **first is more common** amongst Bash programmers.

#### Example

{% code title="function.sh" %}
```bash
#!/bin/bash
# function example

print_me () {
  echo "You have been printed!"
}

print_me
```
{% endcode %}

```
kali@kali:~$ ./func.sh
You have been printed!
```

## Arguments

Functions are capable of accepting arguments (details here).

## Return Values

Bash functions do not actually allow you to return an arbitrary value in the traditional sense. Instead, a Bash function can _return_:

* An exit status
  * Zero (`0`) for success
  * Non-zero for failure
* Some other arbitrary value that can later be accessed from the `$?` global variable
  * Alternatively, one can set a global variable inside the function or use command substitution to simulate a traditional return

#### Example

A simple example that returns a random number into `$?`.

{% code title="funcrvalue.sh" %}
```bash
#!/bin/bash
# function return value example

return_me() {
  echo "Oh hello there, I'm returning a random value!"
  return $RANDOM
}

return_me

echo "The previous function returned a value of $?"
```
{% endcode %}

* Notice the value `$RANDOM` is returned (into the global variable `$?`)

```
kali@kali:~$ ./funcrvalue.sh 
Oh hello there, I'm returning a random value!
The previous function returned a value of 198
```

## Variable Scope

The scope of a variable is simply the context in which it has meaning. By default, a variable has a **global** scope, meaning it can be accessed throughout the entire script.&#x20;

In contrast, a **local** variable can only be seen within the function, block of code, or subshell in which it is defined.&#x20;

One can "overlay" a global variable, giving it a local context, by preceding the declaration with the `local` keyword, leaving the global variable untouched.

#### Example

{% code title="varscope.sh" %}
```bash
#!/bin/bash
# var scope example

name1="John"
name2="Jason"

name_change() {
  local name1="Edward"
  echo "Inside of this function, name1 is $name1 and name2 is $name2"
    name2="Lucas"
}

echo "Before the function call, name1 is $name1 and name2 is $name2"

name_change

echo "After the function call, name1 is $name1 and name2 is $name2"
```
{% endcode %}

```
kali@kali:~$ ./varscope.sh 
Before the function call, name1 is John and name2 is Jason
Inside of this function, name1 is Edward and name2 is Jason
After the function call, name1 is John and name2 is Lucas
```
