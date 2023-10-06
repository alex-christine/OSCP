---
description: Linux commands for various SQL databases
---

# Database-Specific Instructions

## Connect

### MySQL

```shell-session
kali@kali:~$ mysql -u root -p'root' -h 192.168.50.16 -P 3306
```

### Microsoft SQL Server (MSSQL)

{% code overflow="wrap" %}
```shell-session
kali@kali:~$ impacket-mssqlclient user:password@192.168.50.18 -windows-auth
```
{% endcode %}

## List Databases

Commands to list all databases in a SQL instance

### MySQL

```sql
mysql> SHOW DATABASES;
```

### MSSQL

```sql
MSSQL> SELECT name, database_id, create_date FROM sys.databases;
```

## List Tables in Database

### MySQL

```sql
mysql> USE example;    # Select a database (called "example")
mysql> SHOW TABLES;    # Can also use "SHOW FULL TABLES" for more detail
```

### MSSQL

```sql
SELECT TABLE_NAME 
FROM [<DATABASE_NAME>].INFORMATION_SCHEMA.TABLES 
WHERE TABLE_TYPE = 'BASE TABLE';
```

## Table Enumeration

### MySQL

```sql
mysql> DESCRIBE table_name;            # Shows columns for table called "table_name"
mysql> SHOW COLUMNS FROM table_name;   # Shows columns for table called "table_name"
```

### MSSQL

<pre class="language-sql"><code class="lang-sql"><strong>SELECT
</strong>        COLUMN_NAME, ORDINAL_POSITION, DATA_TYPE
    FROM
        INFORMATION_SCHEMA.COLUMNS
    WHERE
        TABLE_NAME = 'Example'
    ORDER BY 2
GO
</code></pre>
