---
description: Describes various approaches to cybersecurity
---

# Security Strategies

## Shift-Left Security

The idea of shift-left security is to consider security engineering from the outset when designing a product or system, rather than attempt to bake it in after the product has been built.

Without shift-left security, there might be developers shipping products without security, and then need to add in additional layers of security on top of, or along with, the product. If the security team is involved in the development process, there is a better chance of creating a product with controls built in, making a more seamless user experience as well as reducing the need for additional security services.

Most applications do not have security built in and instead rely on platform-level security controls surrounding the services. This can work well; however, it can result in security being weaker or easier to bypass.

## Administrative Segregation

It may seem okay to have an administrator bypass security controls based on their role and functional needs. However, when a threat is internal or otherwise able to obtain valid administrative credentials, the security posture becomes weaker.

In order to defeat internal threats and threats that have acquired valid credentials or authentication capability, a business must segment controls so that no single authority can bypass all controls.

In order to accomplish this, they may need to split controls between application teams and administrators, or split access for administration between multiple administrators, as with [Shamir's Secret Sharing (SSS)](https://en.wikipedia.org/wiki/Shamir's\_secret\_sharing).

### Shamir's Secret Sharing (SSS)

SSS is an efficient secret sharing algorithm for distributing private information (the "secret") among a group so that the secret cannot be revealed unless a quorum of the group acts together to pool their knowledge. This prevents any single administrator from gaining root access and instead requires a group of multiple admins to validate any root access.

This is like a bank vault that requires two separate keys to be turned at the same time to be opened.

## Threat Modelling and Intelligence

After completing an inventory of systems and software, it is important to understand likely attack scenarios.&#x20;

### Threat Modelling

Taking data from real-world adversaries and evaluating those attack patterns and techniques against our people, processes, systems, and software. It is important to consider how the compromise of one system in the network might impact others.

### Threat Intelligence

Data that has been refined in the context of the organization: actionable information that an organization has gathered via threat modelling about a valid threat to that organization's success.&#x20;

Information isn't considered threat intelligence unless it results in an **action item** for the organization. The existence of an exploit is not threat intelligence; however, it is potentially useful information that might lead to threat intelligence.

An example of threat intelligence occurs when a relevant adversary's attack patterns are learned, _and_ those attack patterns could defeat the current controls in the organization, _and_ when that adversary is a potential threat to the organization.

When real threat intelligence is gathered, an organization can take informed action to improve their processes, procedures, tactics, and controls.

## Table-Top Tactics

After concerning threat intelligence or other important information is received, enterprises may benefit from immediately scheduling a _cross organization_ discussion. One type of discussion is known as a **table-top**, which brings together engineers, stakeholders, and security professionals to discuss how the organization might react to various types of disasters and attacks.&#x20;

Conducting regular table-tops to evaluate different systems and environments is a great way to ensure that all teams know the **Tactics, Techniques, and Procedures (TTPs)** for handling various scenarios. Often organizations don't build out proper TTPs, resulting in longer incident response times.

Table-top security sessions are part of **Business Continuity Planning (BCP)**. BCP also includes many other aspects such as live drill responses to situations like ransomware and supply-chain compromise. BCP extends outside of cybersecurity emergencies to include processes and procedures for natural disasters and gun violence.&#x20;

Routine table-top sessions and continuous gathering of relevant intelligence provides a proactive effort for mitigating future issues as well as rehearsing tactics, processes, and procedures.

## Continuous Patching and Supply Chain Validation

### Continuous Automated Patching

Another defensive technique known as **continuous automated patching** is accomplished by pulling the upstream source code and applying it to the lowest development environment. Next, the change is tested, and only moved to production if it is successful.

Rather than continuously running a full patch test environment, an organization can create one with relative ease using our cloud provider, run the relevant tests, then delete it. The primary risk of this approach is supply chain compromise.

### Continuous Supply Chain Validation

occurs when people and systems validate that the software and hardware received from vendors is the expected material and that it hasn't been tampered with, as well as ensuring output software and materials are verifiable by customers and business partners.

This is difficult, and sometimes requires more than software checks, such as physical inspections of equipment ordered.

On the software side of supply chain security, one can use deeper testing and inspection techniques to evaluate upstream data more closely. One might opt to increase the security testing duration to attempt to detect sleeper malware implanted in upstream sources.

Utilizing a **software bill of materials (SBOM)** as a way to track dependencies automatically in the application build process greatly helps us evaluate supply chain tampering. If an organization identifies the software dependencies, creates an SBOM with them, and package the container and SBOM together in a cryptographically-verifiable way, then they can verify the container's SBOM signature before loading it into to production. This kind of process presents additional challenges for adversaries.

## Logging and Chaos Testing

Being able to access granular data quickly is of great benefit to an organization. Well-engineered logging is one of the most important security aspects of application design. With consistent, easy to process, and sufficiently-detailed logging, an operations team can more quickly respond to problems, meaning incidents can be detected and resolved faster.

### Chaos Testing

A type of BCP or **disaster recovery (DR)** practice that is often handled via automation. For example, one might leverage a virtual machine that has valid administrative credentials in the production network to cause intentional disasters from within.&#x20;

Chaos engineering includes a variety of different approaches, such as having red teams create chaos in the organization to test how well the organization is able to handle it, scheduling programmed machine shutdowns at various intervals, or having authenticated malicious platform API commands sent in. The goal is to truly test our controls during messy and unpredictable situations. If a production system and organization can handle chaos with relative grace, then it is an indication that it will be robust and resilient to security threats.
