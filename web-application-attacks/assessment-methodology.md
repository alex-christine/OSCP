---
description: Techniques for assessing a web application's vulnerability to exploitation
---

# Assessment Methodology

Web applications can be written in a variety of programming languages and frameworks, each of which can introduce specific types of vulnerabilities. However, the most common vulnerabilities are similar in concept, regardless of the underlying technology stack.

The first step in assessment is always **information gathering**. Initially questions such as the following will be posed:

* What does the application do?
* What language is it written in?
* What server software is the application running on?

These initial questions will guide the next steps. Basically from here the methodology follows the basic steps:

1. Enumerate functions/services available to the current privilege level
2. Exploit vulnerable functionality/configuration to escalate (or gain initial) privilege
3. Repeat
   * With each cycle through step 2 more things should become visible as privileges escalate
     * Enumerating again after each escalation is important for this reason
