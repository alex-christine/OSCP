---
description: Example of discovering a real Windows buffer overflow vulnerability
---

# Windows Buffer Overflow

This page will focus on an example of a real buffer overflow vulnerability that existed in the login protocol of SyncBreeze (a software to sync files across machines) version 10.0.28. This example will follow the steps to "discover" the vulnerability without relying on published research.

## Discovery Techniques

Generally speaking, there are three primary techniques for identifying flaws in applications:

1. Source code review
2. Reverse engineering
3. Fuzzing

This example will use fuzzing. The goal of fuzzing is to **provide the target application with input that is not handled correctly, resulting in an application crash**. If a crash occurs as the result of processing malformed input data, it may indicate the presence of a potentially exploitable vulnerability, such as a buffer overflow.

The following pages will walk through the steps of finding and then exploiting the buffer overflow.
