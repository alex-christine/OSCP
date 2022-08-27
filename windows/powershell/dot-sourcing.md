---
description: Brief description of dot-sourcing in PowerShell
---

# Dot-Sourcing

Dot-sourcing is a concept in PowerShell that allows you to reference code defined in one script in a separate one.

This will make all variables and functions declared in the dot-sourced script available in the current PowerShell scope.

Scripts loaded in this way will **only be available in the current PowerShell instance** and will need to be reloaded each time we restart PowerShell.

#### Example

Lets say there are some helper functions created in a file `Functions.ps1`.

{% code title="C:\Functions.ps1" %}
```powershell
function Do-Something {

  param($Thing)

  Write-Output "I did something to $Thing"

  }

function Set-Something {

  param($Thing)

  Write-Output "I set something on $Thing"

  }
```
{% endcode %}

And a user is wants to reference them in a separate file (`DoStuff.ps1`), that could be done as follows:

<pre class="language-powershell" data-title="C:\DoStuff.ps1"><code class="lang-powershell">. C:\Functions.ps1

<strong>$thing = 'SomeThing'
</strong>
  Do-Something -Thing  $thing

  Set-Something -Thing  $thingC:\CC::.ps1</code></pre>

The dot-inclusion can be sine on line 1 of `DoStuff.ps1`.

#### Sources

* [MCPMag](https://mcpmag.com/articles/2017/02/02/exploring-dot-sourcing-in-powershell.aspx)
