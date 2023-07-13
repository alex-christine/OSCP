---
description: Examination of a real life Linux buffer overflow exploit
---

# Linux Buffer Overflow

This example will demonstrate Linux buffer overflows by exploiting **Crossfire**, a Linux-based online multiplayer role playing game.

Specifically, Crossfire 1.9.0 is vulnerable to a network-based buffer overflow when passing a string of more than 4000 bytes to the setup sound command.&#x20;

In order to debug the application, the example will use the Evans Debugger (EDB), written by Evan Teran, which provides a familiar-looking debugging environment, inspired by Ollydbg.
