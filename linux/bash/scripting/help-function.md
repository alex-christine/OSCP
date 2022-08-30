---
description: Writing a custom help function for Bash scripts
---

# Help Function

A help function should be written as documentation for the code. It can then be arranged to be displayed when either a `-h` flag or invalid arguments are provided.

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

Help
echo "Hello world!"
```

A user can then declare a -h flag and a default case where options are displayed:

```bash
while getopts ":h" option; do
   case $option in
      h) # display Help
         Help
         exit;;
     \?) # Invalid option
         echo "Error: Invalid option"
         Help
         exit;;
   esac
done
```
