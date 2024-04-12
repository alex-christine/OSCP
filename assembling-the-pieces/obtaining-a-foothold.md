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

# Obtaining a Foothold

[Recall](introduction.md) the attacker can only see 2 machines from outside the network. Presumably there will be more inside but first the attacker must find a way into a public-facing machine and then pivot inside. Based on the findings in the [last section](public-network-enumeration.md) the most promising avenue are the outdated WordPress plugins. To save time this example will just focus on the one that will bear fruit, Duplicator.
