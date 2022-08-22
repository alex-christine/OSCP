# Piping and Redirection

Every program run from the command line has three data streams connected to it that serve as communication channels with the external environment.

| Stream   | ID | Description                                    |
| -------- | -- | ---------------------------------------------- |
| `STDIN`  | 0  | Data fed into the program                      |
| `STDOUT` | 1  | Output from the program (defaults to terminal) |
| `STDERR` | 2  | Error messages (defaults to terminal)          |

Piping (using the `|` operator) and redirection (using the `>` and `<` operators) connects these streams between programs and files.

## Redirecting

### Output

Output of one command can be redirected to a file (or another command) using the `>` operator.

```bash
$ echo "text" > test.txt    # Writes "text" to a file called test.txt
```

* When redirecting output to a file that does not exist, the file will automatically be created by Bash
* When redirecting to an existing file with the `>` operator, the original contents of the file will be overwritten

In order to append text to an existing file via redirection use the `>>` operator.&#x20;

```bash
$ echo "text" >> existing_file.txt    # Appends "text" to existing file
```

### Input

The input operator (`<`) can be used to send output of one command into the input of another.

```bash
$ wc -m < test.txt    # Counts the characters in test.txt file
```

This effectively "connects" the inputs and outputs of commands.

### Error

STDERR can be redirected by specifically calling out its stream and sending it somewhere else.

E.g. a `find` command run against `/` from a user level account will often return a bunch of "Permission Denied" errors as it attempts to search through root-owned directories. These can be redirected to `/dev/null` (path used to "sinkhole" output) as follows

```bash
$ find / -type f -name "test.txt" 2>/dev/null    # Sends STDERR to /dev/null
```

## Piping

Pipes are used to "pipe" the output of one command into the input of another.

* Can be thought of as similar in functionality to `<` operator
  * Ordering of arguments is different

```bash
$ cat text.txt | wc -m    # Counts the characters in test.txt file
```
