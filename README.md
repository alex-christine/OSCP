---
description: Information about the OSCP Exam
---

# Exam

The OSCP certification exam simulates a live network in a private VPN that contains a small number of vulnerable machines (details [below](./#exam-network-structure)).

Students must **score 70 points in order to pass**. Points are given for limited access and full system compromise on the exam network.

Students are given **23 hours and 45 minutes to complete** the exam. An additional **24 hours** are given directly following the exam for students to compile and submit their exam report.

## Exam Network Structure

<table><thead><tr><th width="136">Points</th><th>Section</th><th>Number of Machines</th></tr></thead><tbody><tr><td>60 Points</td><td><a href="./#undefined">Independent Targets</a></td><td>3</td></tr><tr><td>40 Points</td><td><a href="./#undefined">Active Directory Set</a></td><td>3 (2 clients, 1 domain controller)</td></tr></tbody></table>

### Independent Targets

* Three two-step targets
  * Low and high privileges on each machine
* Buffer overflow **may or may not** be included as a low level attack vector
* 20 points per machine
  * 10 points for **low-privilege** access
  * 10 points for **privilege escalation**

### Active Directory Set

New addition to the OSCP exam.

* Consists of three machines
  * 2 clients
  * 1 domain controller
* Points are **only awarded for the full exploit chain** of the domain
  * **No partial credit** for this section

## Bonus Points

Requires completion of at least **10** PWK lab machines along with a detailed report, including all of the PWK course exercise solutions for a total value of **10 Bonus Points**.

The 10 PWK lab machines reported on **must include Active Directory targets**.
