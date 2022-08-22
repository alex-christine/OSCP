---
description: Information about the techniques for searching and manipulating text with Bash
---

# Text Search and Manipulation

## `grep`

searches text files for the occurrence of a given regular expression and outputs any line containing a match to the standard output, which is usually the terminal screen.

### Common Flags

| Flag | Meaning                                                                       |
| ---- | ----------------------------------------------------------------------------- |
| `-i` | Ignore case in of search term                                                 |
| `-r` | Searches recursively through a directory for files containing the search term |

## `sed`

At a very high level, `sed` performs text editing on a stream of text.

{% code title="Using sed to find and replace" lineNumbers="true" %}
```bash
$ echo "I need to try hard" | sed 's/hard/harder/'
I need to try harder
```
{% endcode %}

## `cut`

Used to extract a section of text from a line and output it to the standard output.

### Common Flags

| Flag | Description                                                                                                                                                          |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-b` | Select using a specified byte, a byte set, or a byte range                                                                                                           |
| `-c` | Select using a specified character, a character set, or a character range                                                                                            |
| `-d` | Used to specify a delimiter to use instead of the default TAB delimiter                                                                                              |
| `-f` | Select using a specified field, a field set, or a field range                                                                                                        |
| `-s` | <p>Instructs cut not to print the lines that don't contain delimiters.<br><br>The default setting is to print the lines that don't contain delimiter characters.</p> |

* The `-f`, `-b`, and `-c` options take a LIST argument, which is one of the following
  1. An integer `N` representing a byte, field or character, starting from 1
  2. Multiple integers, comma-separated
  3. A range of integers - Each range can be one of the following
     1. `N-` Starts from the integer N (field, byte or character) up to the end of the line
     2. `N-M` From the integer N up to integer M, inclusive
     3. `-M` From the first field, byte, or character, up to the specified M field, byte, or character
     4. Multiple integer ranges, comma-separated

### Examples

Can be used to extract specific fields from text based on fields and delimiters.

{% code overflow="wrap" lineNumbers="true" %}
```bash
$ echo "I hack binaries,web apps,mobile apps, and just about anything else"| cut -f 2 -d ","
web apps
```
{% endcode %}

In more practical examples, a list of users is extracted from `/etc/passwd` by using `:` as a delimiter and retrieving the first field.

```bash
$ cut -d ":" -f 1 /etc/passwd
root
daemon
bin
sys
sync
games
...
```

## `sort`

Used to sort a file, arranging the records in a particular order.

* By default, the sort command sorts file assuming the contents are ASCII
  * Using options in the sort command can also be used to sort numerically

The sort command follows these features:

1. Lines starting with a number will appear before lines starting with a letter
2. Lines starting with a letter that appears earlier in the alphabet will appear before lines starting with a letter that appears later in the alphabet
3. Lines starting with a uppercase letter will appear before lines starting with the same letter in lowercase

All sort commands write to standard output by default.

### Common Flags

| Flag        | Description                                                                                                                                     |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `-c`        | <p>Used to check if the file given is already sorted or not.<br><br>Will write to standard output if there are lines that are out of order.</p> |
| `-k <col>`  | Sorts by column number specified in `col`                                                                                                       |
| `-o <name>` | Send output to a file called `name`                                                                                                             |
| `-n`        | Sort numerically                                                                                                                                |
| `-r`        | Sort in reverse                                                                                                                                 |
| `-u`        | Unique. Sorts and removes duplicates.                                                                                                           |

## `awk`

AWK is a programming language designed for text processing and is typically used as a data extraction and reporting tool. It is extremely powerful, but can be quite complex so this is only the surface and more research should be done.

```bash
$ echo "hello::there::friend" | awk -F "::" '{print $1, $3}'
hello friend
```

## Practical Example

This is simply to serve as an illustration of how these tools can be used in a real-life scenario.

* Given an Apache HTTP server log that contains evidence of an attack
  * [http://www.offensive-security.com/pwk-files/access\_log.txt.gz](http://www.offensive-security.com/pwk-files/access\_log.txt.gz)
* Task is to use Bash commands to inspect the file and discover various pieces of information
  * Who the attackers were
  * What exactly happened on the server

#### Steps

First, use the `head` and `wc` commands to take a quick peek at the log file to understand its structure

* `head` command displays the first 10 lines in a file
* `wc` command, along with the `-l` option, displays the total number of lines in a file

{% code overflow="wrap" %}
```bash
$ head access.log
201.21.152.44 - - [25/Apr/2013:14:05:35 -0700] "GET /favicon.ico HTTP/1.1" 404 89 "-" "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.31 (KHTML, like Gecko) Chrome/26.0.1410.64 Safari/537.31" "random-site.com"
70.194.129.34 - - [25/Apr/2013:14:10:48 -0700] "GET /include/jquery.jshowoff.min.js HTTP/1.1" 200 2553 "http://www.random-site.com/" "Mozilla/5.0 (Linux; U; Android 4.1.2; en-us; SCH-I535 Build/JZO54K) AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Mobile Safari/534.30" "www.random-site.com"
...

$ wc -l access.log
1173 access.log
```
{% endcode %}

Notice that the log file is text-based and contains different fields (IP address, timestamp, HTTP request, etc.) that are delimited by spaces. This makes the file fairly "grep-friendly"

Begin by searching through the HTTP requests made to the server for all the IP addresses recorded in this log file.

* Do this by piping the output of the `cat` command into the `cut` and `sort` commands
* This should help get a sense of how many attackers are being dealt with

```bash
$ cat access.log | cut -d " " -f 1 | sort -u
201.21.152.44
208.115.113.91
208.54.80.244
208.68.234.99
70.194.129.34
72.133.47.242
88.112.192.2
98.238.13.253
99.127.177.95
```

See that less than ten IP addresses were recorded in the log file.

Next, use `uniq` and `sort` to show unique lines, further refine the output, and sort the data by the number of times each IP address accessed the server. The `-c` option of `uniq` will prefix the output line with the number of occurrences.

```bash
$ cat access.log | cut -d " " -f 1 | sort | uniq -c | sort -urn
   1038 208.68.234.99
     59 208.115.113.91
     22 208.54.80.244
     21 99.127.177.95
      8 70.194.129.34
      1 201.21.152.44
```

A few IP addresses stand out but we will focus on the address that has the highest access frequency first.

To filter out the 208.68.234.99 address and display and count the resources that were being requested by that IP, can use the following sequence:

```bash
$ cat access.log | grep '208.68.234.99' | cut -d "\"" -f 2 | uniq -c
   1038 GET //admin HTTP/1.1
```

From this output, it seems that the IP address at 208.68.234.99 was accessing the /admin directory exclusively. This should be examined further:

```bash
cat access_log.txt| grep '208.68.234.99' | grep '/admin' | sort | uniq -c 
   1037 208.68.234.99 - - [22/Apr/2013:07:51:20 -0500] "GET //admin HTTP/1.1" 401 742 "-" "Teh Forest Lobster"
      1 208.68.234.99 - admin [22/Apr/2013:07:51:25 -0500] "GET //admin HTTP/1.1" 200 575 "-" "Teh Forest Lobster"

# sort is necessary before uniq in order to ensure alike lines are grouped
```

Apparently 208.68.234.99 has been involved in an HTTP brute force attempt against this web server. Furthermore, after about 1000 attempts, it seems like the brute force attempt succeeded, as indicated by the "HTTP 200" message.
