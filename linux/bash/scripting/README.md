# Scripting

A competent security professional skillfully leverages Bash scripting to streamline and automate many Linux tasks and procedures.

## Intro to Scripting

A Bash script is a plain-text file that contains a series of commands that are executed as if they had been typed at a terminal prompt.&#x20;

Generally speaking:

* Bash scripts have an optional extension of `.sh` (for ease of identification)
* Scripts begin with `#!/bin/bash`&#x20;
* Scripts must have executable permissions set before they can be executed

#### Hello World

{% code title="hello-world.sh" %}
```bash
#!/bin/bash
# Hello World Bash Script
echo "Hello World!"
```
{% endcode %}

Assuming it has execute permissions (`chmod +x` if not) this script can be run

```bash
kali@kali:~$ ./hello-world.sh
```
