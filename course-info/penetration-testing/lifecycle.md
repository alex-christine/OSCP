---
description: Overview of the Penetration Testing Lifecycle
---

# Lifecycle

To keep a company's security posture as tightly controlled as possible, they should conduct penetration testing on a regular cadence and after every time there's a significant shift in the target's IT architecture.

A typical penetration test comprises the following stages:

1. Defining the scope
2. Information gathering
3. Vulnerability detection
4. Initial foothold
5. Privilege escalation
6. Lateral movement
7. Reporting/Analysis
8. Lessons Learned/Remediation

## Defining the Scope

The scope of a penetration test engagement defines which IP ranges, hosts, and applications should be test subjects during the engagement, as compared to out-of-scope items that should not be tested.

## Information Gathering

During this step, testers aim to collect as much data about the target as possible.

To begin information gathering, they typically perform reconnaissance to retrieve details about the target organization's infrastructure, assets, and personnel. This can be done either **passively** or **actively**. While the former technique aims to retrieve the target's information with almost no direct interaction, the latter probes the infrastructure directly. Active information gathering reveals a bigger footprint, so it is often preferred to avoid exposure by gathering information passively.

It's important to note that information gathering (also known as enumeration) does not end after the initial reconnaissance. Testers will need to continue collecting data as the penetration test progresses, building knowledge of the target's attack surface as new information is discovered by gaining a foothold or moving laterally.
