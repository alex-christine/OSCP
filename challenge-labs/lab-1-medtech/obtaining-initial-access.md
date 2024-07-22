---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Obtaining Initial Access

As it stands there are not that many potential attack vectors. The most readily apparent is the SQLi that was found on WS2 (`192.168.X.121`) (I currently call it `WS2` because it is the second Web Server in the network). Given that this seems the most exploitable it is probably worth starting there.

## SQLi

The first step is to figure out how to get "normal" SQL execution. I mess around with the parameters for awhile and find that putting anything in the username field followed by `' --` results in normal execution:

<figure><img src="../../.gitbook/assets/CL1-121-SQLi_Normal.png" alt=""><figcaption><p>Getting normal operation</p></figcaption></figure>

Obviously the login fails but it tells me how the query works. In this case invalid credentials is a good thing. It means the query ran "correctly" from the web application's perspective.

### Column Count

I start by trying to figure out the number of columns in the exposed query:

```sql
1' ORDER BY 1 -- -
```

This works correctly (invalid credentials) and I increment the number until it fails which happens at `3`:

<figure><img src="../../.gitbook/assets/CL1-121-SQLi_OrderByFail.png" alt=""><figcaption><p>Failed ORDER BY</p></figcaption></figure>

This tells me there are 2 columns in the output. The issue now arises that there is no way to view the output of the query. Failed logins just get tossed into the "Invalid Credentials" bin.

### Time-Based

Unfortunately the blindness means that this is likely a time-based scenario. I test if time-based works with the query:

```sql
'; IF (1=1) WAITFOR DELAY '0:0:10'; -- -
```

```sql
'; IF (1=2) WAITFOR DELAY '0:0:10'; -- -
```

The page hung on the first for the expected 10 seconds and loaded instantly on the second. Seems like I'm in business here.

#### Finding a Credentials Table

To find a table name I am going to use the same method as in the [Butch lab](../../practice-labs/windows/butch.md#finding-credentials-table):

{% code overflow="wrap" %}
```sql
'; IF ((SELECT count(name) FROM sys.tables WHERE name = 'usernames' )=1) WAITFOR DELAY '0:0:10'; -- -
```
{% endcode %}

I just try different things until I get a hit. Luckily it happens on my second try with `users`, the page takes the 10 seconds to load and I know a table name.

