---
description: Description of how to write a penetration test report
---

# Report Writing

The purpose of a pen-test is to provide the client with a report that allows them to understand and remediate issues within their IT infrastructure. It is often incorrectly assumed that a pentest is just a client paying an ethical hacker to attack their enterprise. While this is often a necessary step, the real _product_ of the pentest is the report, and the action items contained therein.&#x20;

An effective pen-test report should include sufficient technical difficulty that the client's technical teams can understand and resolve the problem, while still being understandable for the non-technical executives.

Keep in mind the clients are the experts in their particular industry. They will often (though not always) be aware of the security concerns of that industry and will expect us to have done our homework to also be aware of them. In practice, this means having a deep understanding of what would cause concern to the client in the event of an attack. In other words, understanding their key business goals and objectives.

## Report Structure

1. Executive Summary
2. Testing Environment Considerations
3. Technical Summary
4. Technical Findings and Recommendations
5. Appendices, Further Information, and References

### Executive Summary

The first section of the report should be an executive summary. This enables senior management to understand the scope and outcomes of the testing at a sufficient level to understand the value of the test, and to approve remediation.

#### Structure

1. Scope outline
2. Time frame of test
3. Rules of engagement
4. Supporting infrastructure and accounts
5. Long form summary
6. Wrap-up

The Executive Summary should start with outlining the scope of the engagement. Having a clear scope agreed upon in advance of the testing defines the bounds of what will be covered. Pentesters then want to be very clear as to what exactly was tested and whether anything was dropped from the scope. Timing issues, such as insufficient testing time due to finding too many vulnerabilities to adequately report on, should be included to ensure that the scope statement for any subsequent test is appropriate.

Second, include the time frame of the test. This includes the length of time spent on testing, the dates, and potentially the testing hours as well.

Third, we should refer to the Rules of Engagement and reference the referee report if a referee was part of the testing team. If denial of service testing was allowed, or social engineering was encouraged, that should be noted here. If a specific testing methodology was followed, it should also be indicated here.

Then, include supporting infrastructure and accounts. Using the example of a web application, if testers were given user accounts by the client, include them here along with the IP addresses that the attacks came from (i.e our testing machines).

At this point the short-form executive summary is complete. The **long form summary** consists of a high-level overview of each step of the engagement and establishes severity, context, and a "worst-case scenario" for the key findings from the testing. It should also make note of any trends that were observed in the testing to provide strategic advice. To highlight trends, it is recommended to group findings with similar vulnerabilities (E.g. list all SQLi vulnerabilities together).&#x20;

It is useful to mention things that the client has done well. This is especially true because while management may be paying for the engagement, the testers' working relationship is often with the technical security teams and it is beneficial to make sure that they are not personally looked down upon. Even those penetration tests that find severe vulnerabilities will likely also identify one or two areas that were hardened.

Finally the Executive Summary should conclude with an engagement wrap-up. Something along the lines of "this summary is expanded upon in more detail later in the report. If there are further questions we are happy to help."

### Testing Environment Considerations

This section of the report should detail any issues that affected the testing. This is usually a fairly small section. At times, there are mistakes or extenuating circumstances that occur during an engagement. While those directly involved will already be aware of them, they should be documented in the report to demonstrate transparency.

When writing the report, consider three potential states with regard to extenuating circumstances (each includes an example comment that could be included in a report):

1. **Positive Outcome**: "There were no limitations or extenuating circumstances in the engagement. The time allocated was sufficient to thoroughly test the environment."
2. **Neutral Outcome**: "There were no credentials allocated to the tester in the first two days of the test. However, the attack surface was much smaller than anticipated. Therefore, this did not have an impact on the overall test. We recommend that communication of credentials occurs immediately before the engagement begins for future contracts, so that we can provide as much testing as possible within the allotted time."
3. **Negative Outcome**: "There was not enough time allocated to this engagement to conduct a thorough review of the application, and the scope became much larger than expected. It is recommended that more time is allocated to future engagements to provide more comprehensive coverage."

### Technical Summary

This section is simply a list of all of the key findings in the report, written out with a summary and recommendation for a technical person, like a security architect, to learn at a glance what needs to be done.

This section should group findings into common areas. For example, all weak account password issues that have been identified would be grouped, regardless of the testing timeline. An example of the structure of this section might be:

* User and Privilege Management
* Architecture
* Authorization
* Patch Management
* Integrity and Signatures
* Authentication
* Access Control
* Audit, Log Management and Monitoring
* Traffic and Data Encryption
* Security Misconfigurations

The section should finish with a risk heat map based on vulnerability severity adjusted as appropriate to the client's context, and as agreed upon with a client security risk representative if possible.

### Technical Findings and Recommendations

This sections is where testers include the full technical details relating to their pentest as well as what they consider to be the appropriate next steps for remediation of each issue. While this is a technical section, the writers should not assume that the audience will be composed of fellow pen-testers.

Not everyone, even those who work within the technologies that were being tested, will fully understand the nuances of the vulnerabilities. While a deep technical dive into the root causes of an exploit is not always necessary, a broad overview of how it was able to take place should usually be provided. It is better to assume less background knowledge on behalf of the audience and give too much information, rather than the opposite.

This section is often presented in tabular form and provides full details of the findings. A finding might cover one vulnerability that has been identified, or may cover multiple vulnerabilities of the same type.

It's important to note that there might be a need for an attack narrative. This narrative describes, in story format, exactly what happened during the test. This is typically done for a simulated threat engagement, but is also useful at times to describe the more complex exploitation steps required for a regular penetration test. If it is necessary, then writing out the attack path step-by-step, with appropriate screenshots, is generally sufficient. An extended narrative could be placed in an Appendix and referenced from the findings table.

#### Sample



<table><thead><tr><th width="80.33333333333331" align="center">REF</th><th width="71" align="center">RISK</th><th width="327">DESCRIPTION &#x26; IMPLICATIONS</th><th>RECOMMENDATIONS</th></tr></thead><tbody><tr><td align="center">1</td><td align="center">H</td><td>Account, Password, and Privilege Management is inadequate. Account management is the process of provisioning new accounts and removing accounts that are no longer required. The following issues were identified by performing an analysis of 122,624 user accounts post-compromise: 722 user accounts were configured to never expire; 23,142 users had never logged in; 6 users were members of the domain administrator group; default initial passwords were in use for 968 accounts.</td><td>All accounts should have passwords that are enforced by a strict policy. All accounts with weak passwords should be forced to change them. All accounts should be set to expire automatically. Accounts no longer required should be removed.</td></tr><tr><td align="center">2</td><td align="center">H</td><td>Information enumerated through an anonymous SMB session. An anonymous SMB session connection was made, and the information gained was then used to gain unauthorized user access as detailed in Appendix E.9.</td><td>To prevent information gathering via anonymous SMB sessions: Access to TCP ports 139 and 445 should be restricted based on roles and requirements. Enumeration of SAM accounts should be disabled using the Local Security Policy > Local Policies > Security Options</td></tr><tr><td align="center">3</td><td align="center">M</td><td>Malicious JavaScript code can be run to silently carry out malicious activity. A form of this is reflected cross-site scripting (XSS), which occurs when a web application accepts user input with embedded active code and then outputs it into a webpage that is subsequently displayed to a user. This will cause attacker-injected code to be executed on the user's web browser. XSS attacks can be used to achieve outcomes such as unauthorized access and credential theft, which can in some cases result in reputational and financial damage as a result of bad publicity or fines. As shown in Appendix E.8, the [client] application is vulnerable to an XSS vulnerability because the username value is displayed on the screen login attempt fails. A proof-of-concept using a maliciously crafted username is provided in Appendix E.</td><td>Treat all user input as potentially tainted, and perform proper sanitization through special character filtering. Adequately encode all user-controlled output when rendering to a page. Do not include the username in the error message of the application login.</td></tr></tbody></table>

It's important to understand that what we identify as the severity of an issue based on its vulnerability score is not context-specific business risk. It only represents technical severity, even if we adjust it based on likelihood.

Start a finding's description with a sentence or two describing what the vulnerability is, why it is dangerous, and what an attacker can accomplish with it. This can be written in such a way to provide insight into the immediate impact of an attack. There is often no need to go into overwhelming detail; simply explain at a basic level what the vulnerability is and how to exploit it.

Writers also need to include evidence to prove the vulnerability identified is exploitable, along with any further relevant information. If this is simple, it can be included inline as per the first entry above. Otherwise, it can be documented in an appendix as shown in the second entry.

Once the basic details of a vulnerability have been covered, describe the specific finding that was identified in the system or application. Use the notes that taken during testing and the screenshots that support them to provide a detailed account. Although this is more than a few sentences, writers will want to summarize it in the table and reference an appendix for the full description.

The remediation advice should be detailed enough to enable system and application administrators to implement it without ambiguity. The remediation should be clear, concise, and thorough. It should be sufficient to remove the vulnerability in a manner acceptable to the client and relevant to the application. Presenting remediation that is excessive, unacceptably costly, or culturally inappropriate (e.g. not allowing remote logins for a remote working environment) will lead to the fix never being implemented. A strong understanding of the needs of the client is necessary here.

### Appendices, Further Information, and References

#### Appendices

The final part of the report is the **Appendices** section. Things that go here typically do not fit anywhere else in the report, or are too lengthy or detailed to include inline. A good rule to follow is if it's necessary for the report but would break the flow of the page, put it in an appendix. The types of things that may need to go in an appendix are:

* Long lists of compromised users or affected areas
* Large proof-of-concept code blocks
* Expanded methodology or technical write-ups

#### Further Information

This section is optional, and includes things that may not be necessary for the main write-up but could reasonably provide value for the client. Examples would include:

* Articles that describe the vulnerability in more depth
* Standards for the remediation recommendation for the client to follow
* Other methods of exploitation.&#x20;

If there is nothing that can add enough value, there is no reason to necessarily include this section.

#### References

This section can be a useful way to provide more insight for the client in areas not directly relevant to the testing we carried out. When providing references, writers need to ensure they only use the most authoritative sources, and should also ensure that they are cited properly.
