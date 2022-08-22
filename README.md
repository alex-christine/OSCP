---
description: Information about the OSCP Exam
---

# Exam

The OSCP certification exam simulates a live network in a private VPN that contains a small number of vulnerable machines (details [below](./#exam-network-structure)).

Students must **score 70 points in order to pass**. Points are given for limited access and full system compromise on the exam network.

Students are given **23 hours and 45 minutes to complete** the exam. An additional **24 hours** are given directly following the exam for students to compile and submit their exam report.

## Exam Network Structure

| Points    | Section                              | Number of Machines                 |
| --------- | ------------------------------------ | ---------------------------------- |
| 60 Points | [Independent Targets](./#undefined)  | 3                                  |
| 40 Points | [Active Directory Set](./#undefined) | 3 (2 clients, 1 domain controller) |

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
