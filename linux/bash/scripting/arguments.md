---
description: How arguments are declared and used in Bash
---

# Arguments

Not all Bash scripts require arguments. However, it is extremely important to understand how they are interpreted by Bash and how to use them.

| Variable    | Description                                                                           |
| ----------- | ------------------------------------------------------------------------------------- |
| `$0`        | Name of the Bash script                                                               |
| `$1`-`$9`   | First 9 arguments of the Bash script                                                  |
| `$#`        | Number of arguments passed to the Bash script                                         |
| `$@`        | All arguments passed to the Bash script                                               |
| `$?`        | Exit status of the most recently run process                                          |
| `$$`        | Process ID of the current script                                                      |
| `$USER`     | Username of the user running the script                                               |
| `$HOSTNAME` | Hostname of the machine                                                               |
| `$RANDOM`   | A random number                                                                       |
| `$LINENO`   | Current line number in the script                                                     |
| `$OPTARG`   | Variable name used for each new option argument (see [below](arguments.md#undefined)) |

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
