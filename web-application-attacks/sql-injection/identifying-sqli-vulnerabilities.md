---
description: Techniques for locating SQLi vulnerabilities
---

# Identifying SQLi Vulnerabilities

Before finding SQL injection vulnerabilities, one must first identify locations where data might pass through a database. Some places on a web application that may require database interaction include:

* Authentication
* Products on an E-commerce site
* Message threads on a forum
* Sometimes URL parameters will plug directly into a database

## Basic Checks

### Single-Quote (') Check

SQL uses the single-quote (`'`) character as a string delimiter. Thus it can be used as a simple check for potential SQLi vulnerabilities.

If the application doesn’t handle this character correctly, it will likely result in a database error and can indicate that a SQL injection vulnerability exists.

#### Example

Some PHP code building a SQL query via concatenation would be vulnerable to this type of attack:

```php
$query = "select * from users where username = '$user' and password = '$pass'";
```

In this instance the developer is directly pulling the `$user` and `$pass` variables into the query.

If one were to submit a single-quote to either or both of those fields the concatenation would break:

```php
$query = "select * from users where username =''' and password = 'password123' ";
```

And yield an error message:

{% code overflow="wrap" %}
```
Notice: invalid query: You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'password123'' at line 1 in C:\xampp\htdocs\login.php on line 20
```
{% endcode %}

### URL Parameters

Some poorly designed web applications may take URL parameters and pass them to a database.

#### Example

Consider a news website that procures articles with a URL structure such as:

```
http://newspaper.com/items.php?id=2
```

One might imagine that one the backend a query is being constructed:

```sql
SELECT title, description, body FROM items WHERE ID = 2
```

An attacker might then submit a URL such as the following to test:

```
http://newspaper.com/items.php?id=2 and 1=2
```

This would be expected to return nothing, assuming that happens another query returning true may be tested:

```
http://newspaper.com/items.php?id=2 and 1=1
```

Assuming this returns correctly further exploration is warranted.

### Time-Based SQLi Detection

OWASP's [Blind SQLi](https://owasp.org/www-community/attacks/Blind\_SQL\_Injection) page includes a description of a technique for using time-based techniques to determine SQLi vulnerability.

Basically the technique requires on submitting delay commands as part of the query, running the queries a bunch of times, and using the large samples to spot timing differences in successful queries vs. failed ones.

For further details, see linked page.

## Sample Workflow

Below is a general-purpose example of how to find a SQLi vulnerability on a page/application. As SQLi testing is generally a black box this will mostly consist of trial and error:

1. Identify all places on the application that may require database interaction
2. Place a single-quote (`'`) character in each field identified in step 1 to see what happens
   * If able to examine the source code of the application many of these steps can be sped up
     * Of particular interest would be anything that shows SQL queries being built by string concatenation
3. If anything breaks continue pursuing that avenue
4. Examine URL structure to see if any parameters may be passed to queries
