---
description: Commands for file comparison in bash
---

# Comparing Files

File comparison may seem irrelevant, but system administrators, network engineers, penetration testers, IT support technicians and many other technically-oriented professionals rely on this skill pretty often.

## `comm`

Compares two text files, displaying the lines that are unique to each one, as well as the lines they have in common.&#x20;

It outputs three space-offset columns:

1. Contains lines that are unique to the first file or argument
2. Contains lines that are unique to the second file or argument
3. Contains lines that are shared by both files

* The `-n` switch can be used to suppress one or more columns, depending on the need
  * `n` is either 1, 2, or 3

## `diff`

Used to detect differences between files, similar to the comm command. However, diff is much more complex and supports many output formats.

### Output Formats

| Format Flag | Description    |
| ----------- | -------------- |
| `-c`        | Context format |
| `-u`        | Unified format |

#### Example

```bash
kali@kali:~$ diff -c scan-a.txt scan-b.txt
*** scan-a.txt	2018-02-07 14:46:21.557861848 -0700
--- scan-b.txt	2018-02-07 14:46:44.275002421 -0700
***************
*** 1,5 ****
  192.168.1.1
- 192.168.1.2
  192.168.1.3
  192.168.1.4
  192.168.1.5
--- 1,5 ----
  192.168.1.1
  192.168.1.3
  192.168.1.4
  192.168.1.5
+ 192.168.1.6

kali@kali:~$ diff -u scan-a.txt scan-b.txt
--- scan-a.txt	2018-02-07 14:46:21.557861848 -0700
+++ scan-b.txt	2018-02-07 14:46:44.275002421 -0700
@@ -1,5 +1,5 @@
 192.168.1.1
-192.168.1.2
 192.168.1.3
 192.168.1.4
 192.168.1.5
+192.168.1.6
```

The output uses the "-" indicator to show that the line appears in the first file, but not in the second. Conversely, the "+" indicator shows that the line appears in the second file, but not in the first.

The most notable difference between these formats is that the _unified format_ does not show lines that match between files, making the results shorter. The indicators have identical meaning in both formats.

## `vimdiff`

vimdiff opens vim[1](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/command-line-fun/comparing-files/vimdiff#fn1) with multiple files, one in each window. The differences between files are highlighted, which makes it easier to visually inspect them.

### Shortcuts

There are a few shortcuts that may be useful.

| Shortcut   | Action                                                      |
| ---------- | ----------------------------------------------------------- |
| `do`       | Gets changes from the other window into the current one     |
| `dp`       | Puts the changes from the current window into the other one |
| `]c`       | Jumps to the next change                                    |
| `[c`       | Jumps to the previous change                                |
| `Ctrl + W` | Switches to the other split window                          |

![vimdiff Example](../../.gitbook/assets/Example\_vimdiff.png)
