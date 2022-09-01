---
description: Solutions to Bash Scripting exercises
---

# Bash Scripting

## VM1

For this first challenge, you simply need to write a Bash script that prints _Hello World!_. The script must print this exactly and nothing else. As described earlier, this script must start with `#!/bin/bash` and be executable. Once created, upload this script to the upload your script to the Kali VM #1 student's home folder and run `/challenge/hello_world` with your script's location as the first argument to get the flag.

### Response

{% code title="hello.sh" %}
```bash
#!/bin/bash
echo "Hello World!"
```
{% endcode %}

File was "uploaded" by SSH into the machine, `touch hello.sh`, writing script in `nano` (or copying from attacking machine into nano).

* No idea if this is what they had in mind but it worked
* Assume this is repeated for all remaning machines unless specified

## VM2

Create a Bash script that simply prints the name of the script and nothing else. Upload your script to the Kali VM #2 student's home folder and run _/challenge/scriptname_ with your script's location as the first argument to get the flag.

### Response

{% code title="print_self.sh" %}
```bash
#!/bin/bash
echo $0
```
{% endcode %}

## VM3

Create a Bash script that prints the argument count (i.e. the number of arguments passed to the Bash script). The script should print exactly _This script has \_ arguments_ where the _\__ (the blank) is the number of arguments and nothing else. Upload or your script to Kali VM #3 student's home folder and run `/challenge/argument-count` with your script's location as the first argument to get the flag.

### Response

{% code title="count_args.sh" %}
```bash
#!/bin/bash
echo "This script has $# arguments"
```
{% endcode %}

## VM5

Create a Bash script which extracts JavaScript file names from any access log file. This access-log file will be the first argument to this script. Make sure the file names _DO NOT_ include the path, are unique, and are sorted. Upload your script to the Kali VM #5 student's home folder and run _/challenge/access-log_ with your script's location as the first argument to get the flag.

### Response

{% code title="extract_js.sh" %}
```bash
#!/bin/bash

if [ $# -ge 1 ]
then
    log_path=$1
else
    echo "Please include file path as argument"
    exit
fi

if ! [ -e $log_path ]
then
    echo "Log file does not exist at path: $log_path"
fi

cat $log_path | grep -oP '[^\/]+\.js' | sort | uniq
```
{% endcode %}

## VM7

Create a short Bash script that will validate a user's membership in a specified group. This script will not take any arguments and, instead, will prompt the user to enter a username and a group. This script will first check to see if the username and group are found on this system (simply exist in their respective _/etc/_ files). If BOTH ARE NOT FOUND, the script will respond _Both are not found - why are you even asking me this?_. If ONLY ONE IS FOUND, it will respond _One exists, one does not. You figure out which_. If BOTH ARE FOUND, it will also check to see if the user is a part of the group. If the USER IS A MEMBER OF THE GROUP, the script will respond _Membership valid!_; otherwise, it will respond _Membership invalid but available to join_. To be clear, the script will initially prompt twice for user input (the prompt does not matter) and then only respond once with one of the four specified responses. Once complete, upload your script to the Kali VM #7 student's home folder and run _/challenge/group-membership_ with your script's location as the first argument to get the flag.

### Response

{% code title="check_user.sh" %}
```bash
#!/bin/bash

##############################  HELPER FUNCTIONS  ##############################

function check_user {
    # Expects username to be passed as argument $1
    # Returns 1 if user exists and 0 if not
    cat /etc/passwd | cut -d : -f 1 | grep -qw $1 && return 1 || return 0
}

function check_group {
    # Expects group to be passed as argument $1
    # Returns 1 if user exists and 0 if not
    cat /etc/group | cut -d : -f 1 | grep -qw $1 && return 1 || return 0
}

function check_user_in_group {
    # Used to check if a particular user is in a group
    # Expects username in $1 and group in $2
    # Returns 1 if the user is a member of the group and 0 if not
    awk -v group=$2 -F':' '{ if($1 == group) {print $4} }' /etc/group | grep -qw $1 && return 1 || return 0

}
###############################  MAIN FUNCTION  ################################

read -p 'Please enter a username: ' user
read -p 'Now enter a group: ' group

# Check User
check_user $user
if [ $? -eq 1 ]
then
    user_exists=true
else
    user_exists=false
fi

# Check Group
check_group $group
if [ $? -eq 1 ]
then
    group_exists=true
else
    group_exists=false
fi

if ! $group_exists && ! $user_exists; then  # Case neither exist
    echo "Both are not found - why are you even asking me this?"
elif $group_exists && $user_exists; then    # Both exist 
    # echo "Both exist"
    check_user_in_group $user $group
    if [ $? -eq 1 ]; then                   # Membership valid
        echo "Membership valid!"
    else                                    # Membership invalid
        echo "Membership invalid but available to join."
    fi
else                                        # Group or user exists but not both
    echo "One exists, one does not. You figure out which."
fi
```
{% endcode %}
