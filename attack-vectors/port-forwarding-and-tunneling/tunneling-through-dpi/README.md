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

# Tunneling Through DPI

## Deep Packet Inspection

[Deep Packet Inspection](https://en.wikipedia.org/wiki/Deep\_packet\_inspection) (DPI) is a technology that's implemented to monitor traffic based on a set of rules. DPI examines the contents of packets passing through a given checkpoint and makes real-time decisions depending on what a packet contains and based on rules assigned by a network manager. It is most often used on a network perimeter, where it can highlight patterns that are indicative of compromise.

DPI is mainly used by firewalls that include an intrusion detection system (IDS) feature and by standalone IDSes that are intended to both detect attacks and protect the network.

It can be used for benevolent purposes as a network security tool to detect and intercept viruses, worms, spyware and other forms of malicious traffic and intrusion attempts.

For example, a network administrator could create a rule that terminates any outbound SSH traffic. If they implemented that rule, all connections that use SSH for transport would fail, including any SSH port redirection and tunneling strategies.

### Deep Packet Inspection Techniques

#### Pattern or Signature Matching&#x20;

A firewall with IDS capability analyzes each packet against a database of known network attacks. It looks for specific patterns that are known to be malicious and blocks the traffic if it finds such a pattern.&#x20;

The disadvantage of this approach is that its effectiveness depends on the signatures being updated regularly. This method only works against known threats or attacks. As new threats are discovered daily, ongoing signature updates are critical to ensure that the firewall can detect the threats and continue to protect the network.

#### Protocol Anomaly

Protocol anomaly detection follows a default deny approach. The firewall determines which content/traffic should be allowed based on protocol definitions and only allows matching traffic through. Thus, unlike signature matching, this method also protects the network against unknown attacks.

#### Intrusion Prevention System

IPS solutions can block detected attacks in real time by preventing malicious packets from being delivered based on their contents. Thus, if a particular packet represents a known security threat, the IPS will proactively deny network traffic based on a defined rule set.

One drawback of IPS is that the cyberthreat database must be regularly updated with information about new threats. The risk of false positives is also high but can be mitigated by establishing proper baseline behaviors for network components, creating conservative policies and custom thresholds, and regularly reviewing alerts and logged incidents to improve monitoring and alerting.

The following pages will go over some techniques to tunnel past DPI.
