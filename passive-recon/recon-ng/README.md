---
description: Utilizing the recon-ng web reconnaissance framework
---

# Recon-ng

Recon-ng is a full-featured Web Reconnaissance framework written in Python.

Recon-ng displays the results of a module to the terminal but it also stores them in a database. Much of the power of recon-ng lies in feeding the results of one module into another, allowing users to quickly expand the scope of their information gathering.

## Installing Modules

### Finding Modules

Modules can easily be searched using the `marketplace search KEYWORD` construct. It is recommended to keep the search terms as simple as possible.

#### Example

```
recon-ng][default] > marketplace search github
[*] Searching module index for 'github'...

+------------------------------------------------------------------------------------+
|                       Path                      | Version |     Status    | D | K |
+------------------------------------------------------------------------------------+
| recon/companies-multi/github_miner              | 1.0     | not installed |   | * |
| recon/profiles-contacts/github_users            | 1.0     | not installed |   | * |
| recon/profiles-profiles/profiler                | 1.0     | not installed |   |   |
| recon/profiles-repositories/github_repos        | 1.0     | not installed |   | * |
| recon/repositories-profiles/github_commits      | 1.0     | not installed |   | * |
| recon/repositories-vulnerabilities/github_dorks | 1.0     | not installed |   | * |
+------------------------------------------------------------------------------------+

D = Has dependencies. See info for details.
K = Requires keys. See info for details.
```

### Researching Modules

To learn more about a module use the `marketplace info path/to/module` construct.

#### Example

```
[recon-ng][default] > marketplace info recon/domains-hosts/google_site_web

+------------------------------------------------------------------------------+
| path          | recon/domains-hosts/google_site_web                          |
| name          | Google Hostname Enumerator                                   |
| author        | Tim Tomes (@lanmaster53)                                     |
| version       | 1.0                                                          |
| last_updated  | 2019-06-24                                                   |
| description   | Harvests hosts from Google.com by using the 'site' operator. |
| required_keys | []                                                           |
| dependencies  | []                                                           |
| files         | []                                                           |
| status        | not installed                                                |
+------------------------------------------------------------------------------+

[recon-ng][default] > 
```

### Installation

Once a module that meets a user's requirements has been found it can be installed with the `marketplace install path/to/module` construct.

#### Example

```
[recon-ng][default] > marketplace install recon/domains-hosts/google_site_web
[*] Module installed: recon/domains-hosts/google_site_web
[*] Reloading modules...
[recon-ng][default] > 
```

## Using Modules

### Loading Modules

Prior to use a module must be loaded using the `modules load path/to/module` construct.

After the module is loaded the `info` command is used to display information about the module (including options).

#### Example

```
[recon-ng][default] > modules load recon/domains-hosts/google_site_web

[recon-ng][default][google_site_web] > info

      Name: Google Hostname Enumerator
    Author: Tim Tomes (@lanmaster53)
   Version: 1.0

Description:
  Harvests hosts from Google.com by using the 'site' search operator. Updates the 
  'hosts' table with the results.

Options:
  Name    Current Value  Required  Description
  ------  -------------  --------  -----------
  SOURCE  default        yes       source of input (see 'show info' for details)

Source Options:
  default        SELECT DISTINCT domain FROM domains WHERE domain IS NOT NULL
  <string>       string representing a single input
  <path>         path to a file containing a list of inputs
  query <sql>    database query returning one column of inputs

[recon-ng][default][google_site_web] > 
```

### Configuring Options

Options can be set with the `options set OPTION value` command.

#### Example

```
[recon-ng][default][google_site_web] > options set SOURCE megacorpone.com
SOURCE => megacorpone.com
```

### Running Modules

Once the options are set the module is run with the `run` command.

#### Example

```
[recon-ng][default][resolve] > run
[*] www.megacorpone.com => 38.100.193.76
[*] vpn.megacorpone.com => 38.100.193.77
[*] www2.megacorpone.com => 38.100.193.79
[*] siem.megacorpone.com => 38.100.193.89
```

### Displaying Results

In order to display information about a particular set of results use the `show CATEGORY` command. In order to see a list of category options, run the `show` command with no options.

#### Example

In this example, the data will be stored under `hosts` category.

```
[recon-ng][default][google_site_web] > back

[recon-ng][default] > show
Shows various framework items

Usage: show <companies|contacts|credentials|domains|hosts|leaks|locations|netblocks|ports|profiles|pushpins|repositories|vulnerabilities>

[recon-ng][default] > show hosts

+--------------------------------------------------------------------------------+
| rowid |         host         | ip_address | region | country |      module     |
+--------------------------------------------------------------------------------+
| 1     | www.megacorpone.com  |            |        |         | google_site_web |
| 2     | vpn.megacorpone.com  |            |        |         | google_site_web |
| 3     | www2.megacorpone.com |            |        |         | google_site_web |
| 4     | siem.megacorpone.com |            |        |         | google_site_web |
+--------------------------------------------------------------------------------+

[*] 4 rows returned
[recon-ng][default] > 
```

The real benefit of recon-ng really starts to show itself here. If a user were to run a second module that returned data about the hosts category, this would be combined with the earlier data.

In this example the user installed, loaded, and ran the `recon/hosts-hosts/resolve` module. After it completes, running `show hosts` again will reveal the combined data of both modules.

```
[recon-ng][default][resolve] > show hosts

+-----------------------------------------------------------------------------------+
| rowid |         host         |   ip_address  | region | country |      module     |
+-----------------------------------------------------------------------------------+
| 1     | www.megacorpone.com  | 38.100.193.76 |        |         | google_site_web |
| 2     | vpn.megacorpone.com  | 38.100.193.77 |        |         | google_site_web |
| 3     | www2.megacorpone.com | 38.100.193.79 |        |         | google_site_web |
| 4     | siem.megacorpone.com | 38.100.193.89 |        |         | google_site_web |
+-----------------------------------------------------------------------------------+

[*] 4 rows returned
```
