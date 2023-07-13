---
description: Mechanisms implemented by Linux to prevent buffer overflows
---

# Prevention Mechanisms

Recent Linux kernels and compilers have implemented various memory protection techniques such as

* Data Execution Prevention (DEP)
* Address Space Layout Randomization (ASLR)
* Stack Canaries

## Data Execution Prevention

This mechanism mostly works like the [Windows version](../windows-buffer-overflow/prevention-mechanisms.md#data-execution-prevention-dep).

DEP marks memory regions as non-executable, such that an attempt to execute machine code in these regions will cause an exception.&#x20;

It makes use of hardware features such as the **NX bit** (no-execute bit), or in some cases software emulation of those features.

The Linux kernel supports the NX bit on x86-64 and IA-32 processors that support it, such as modern 64-bit processors made by AMD, Intel, Transmeta and VIA.
