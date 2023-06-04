---
description: How arguments are declared and used in Bash
---

# Arguments

Not all Bash scripts require arguments. However, it is extremely important to understand how they are interpreted by Bash and how to use them.

<table><thead><tr><th width="180">Variable</th><th>Description</th></tr></thead><tbody><tr><td><code>$0</code></td><td>Name of the Bash script</td></tr><tr><td><code>$1</code>-<code>$9</code></td><td>First 9 arguments of the Bash script</td></tr><tr><td><code>$#</code></td><td>Number of arguments passed to the Bash script</td></tr><tr><td><code>$@</code></td><td>All arguments passed to the Bash script</td></tr><tr><td><code>$?</code></td><td>Exit status of the most recently run process</td></tr><tr><td><code>$$</code></td><td>Process ID of the current script</td></tr><tr><td><code>$USER</code></td><td>Username of the user running the script</td></tr><tr><td><code>$HOSTNAME</code></td><td>Hostname of the machine</td></tr><tr><td><code>$RANDOM</code></td><td>A random number</td></tr><tr><td><code>$LINENO</code></td><td>Current line number in the script</td></tr><tr><td><code>$OPTARG</code></td><td>Variable name used for each new option argument (see <a href="arguments.md#undefined">below</a>)</td></tr></tbody></table>

## Named Arguments

This example will be a Hello World script that defaults to output "Hello world!" but an optional `-n` flag can be used to add a name and output "Hello name!" instead. Additionally, the `-h` flag can be used to display a help message.

{% code title="hello.sh" lineNumbers="true" %}
```bash
#!/bin/bash
############################################################
# Help                                                     #
############################################################
Help()
{
   # Display Help
   echo "Add description of the script functions here."
   echo
   echo "Syntax: scriptTemplate [-g|h|v|V]"
   echo "options:"
   echo "g     Print the GPL license notification."
   echo "h     Print this Help."
   echo "v     Verbose mode."
   echo "V     Print software version and exit."
   echo
}

############################################################
############################################################
# Main program                                             #
############################################################
############################################################

# Set variables
Name="world"

############################################################
# Process the input options. Add options as needed.        #
############################################################
# Get the options
while getopts ":hn:" option; do
   case $option in
      h) # display Help
         Help
         exit;;
      n) # Enter a name
         Name=$OPTARG;;
     \?) # Invalid option
         echo "Error: Invalid option"
         Help
         exit;;
   esac
done

echo "hello $Name!"
```
{% endcode %}

* Note how the value of argument `-n` is retained with the `$OPTARG` special variable in line 38
* Additional arguments may be added
  * First to arguments list in line 32 (E.g. change `":hn:"` to `":hnv:"` to add a `-v` flag)
  * Then add a case (E.g. `v)` to declare a `-v` flag) to handle the new flag
* `/?` is the default case
  * In this instance it will display the help message (due to the `Help` call on line 41)
