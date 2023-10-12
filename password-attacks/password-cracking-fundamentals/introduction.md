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

# Introduction

It is important to understand:

* Symmetric Encryption
* Asymmetric Encryption
* Cryptographic Hashing

For background on these topics, see the relevant [Security Now!](https://www.grc.com/securitynow.htm) podcast episodes.

## Tools

Two of the primary password cracking tools are [Hashcat](https://hashcat.net/hashcat/) and [John the Ripper](https://www.openwall.com/john/) (JtR). It's important to become familiar with different tools since they don't support the same algorithms.

### John the Ripper

**JtR is more of a CPU-based cracking tool**, which also supports GPUs.

JtR can be run without any additional drivers using only CPUs for password cracking.

### Hashcat

**Hashcat is mainly a GPU-based cracking** tool that also supports CPUs.&#x20;

Hashcat requires `OpenCL` or `CUDA` for the GPU cracking process. For most algorithms, a GPU is much faster than a CPU since modern GPUs contain thousands of cores, each of which can share part of the workload. However, some slow hashing algorithms (like _bcrypt_) work better on CPUs.
