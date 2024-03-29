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

# Authentication

Active Directory supports multiple authentication protocols and techniques that implement authentication to Windows computers as well as those running Linux and macOS. While older protocols (such as [WDigest](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc778868\(v=ws.10\)?redirectedfrom=MSDN)) may be helpful for Windows 7-era machines, this section will primarily focus on the more modern methods: **NTLM** and **Kerberos** authentication.

## NTLM

NT (New Technology) LAN Manager (NTLM) is a suite of Microsoft security protocols intended to provide authentication, integrity, and confidentiality to users.

### Protocol Operation

The following steps present an outline of NTLM non-interactive authentication. The first step provides the user's NTLM credentials and occurs only as part of the interactive authentication (logon) process.

1. (Interactive authentication only) A user accesses a client computer and provides a domain name, user name, and password. The client computes a cryptographic hash of the password and discards the actual password. This is known as the **NTLM hash**
2. The client sends the user name to the server (in plaintext).
3. The server generates a 16-byte random number, called a challenge or nonce, and sends it to the client.
4. The client encrypts this challenge with the hash of the user's password and returns the result to the server. This is called the response.
5. The server sends the following three items to the domain controller:
   * Username
   * Challenge sent to the client
   * Response received from the client
6. The domain controller uses the user name to retrieve the hash of the user's password from the Security Account Manager database. It uses this password hash to encrypt the challenge.
7. The domain controller compares the encrypted challenge it computed (in step 6) to the response computed by the client (in step 4). If they are identical, authentication is successful.

<figure><img src="../../../.gitbook/assets/AD-NTLMDiagram.png" alt=""><figcaption><p>NTLMv2 Authentication in AD environment</p></figcaption></figure>

### Versions

#### NTLMv1

The server authenticates the client by sending an 8-byte random number, the challenge. The client performs an operation involving the challenge and a secret shared between client and server, specifically one of the two password hashes described above. The client returns the 24-byte result of the computation. In fact, in NTLMv1 the computations are usually made using both hashes and both 24-byte results are sent. The server verifies that the client has computed the correct result, and from this infers possession of the secret, and hence the authenticity of the client.

Both the hashes produce 16-byte quantities. Five bytes of zeros are appended to obtain 21 bytes. The 21 bytes are separated in three 7-byte (56-bit) quantities. Each of these 56-bit quantities is used as a key to [DES](https://en.wikipedia.org/wiki/Data\_Encryption\_Standard) encrypt the 64-bit challenge. The three encryptions of the challenge are reunited to form the 24-byte response.

The exchange looks like this:

```
C = 8-byte server challenge, random
K1 | K2 | K3 = NTLM-Hash | 5-bytes-0
response = DES(K1,C) | DES(K2,C) | DES(K3,C)
```

#### NTLMv2

NTLMv2 is a challenge-response authentication protocol. It is intended as a cryptographically strengthened replacement for NTLMv1, enhancing NTLM security by hardening the protocol against many spoofing attacks and adding the ability for a server to authenticate to the client.

NTLMv2 sends two responses to an 8-byte _server challenge_. Each response contains a 16-byte [HMAC](https://en.wikipedia.org/wiki/HMAC)-[MD5](https://en.wikipedia.org/wiki/MD5) hash of the server challenge, a fully/partially randomly generated _client challenge_, and an HMAC-MD5 hash of the user's password and other identifying information. The two responses differ in the format of the client challenge. The shorter response uses an 8-byte random value for this challenge. In order to verify the response, the server must receive as part of the response the client challenge. For this shorter response, the 8-byte client challenge appended to the 16-byte response makes a 24-byte package which is consistent with the 24-byte response format of the previous NTLMv1 protocol.

The second response sent by NTLMv2 uses a variable-length client challenge which includes (1) the current time in [NT Time](https://en.wikipedia.org/wiki/NT\_Time) format, (2) an 8-byte random value (CC2 in the box below), (3) the domain name and (4) some standard format stuff. The response must include a copy of this client challenge, and is therefore variable length.

The challenge response exchange follows this format:

```
SC = 8-byte server challenge, random
CC = 8-byte client challenge, random
CC* = (X, time, CC2, domain name)
v2-Hash = HMAC-MD5(NT-Hash, user name, domain name)
LMv2 = HMAC-MD5(v2-Hash, SC, CC)
NTv2 = HMAC-MD5(v2-Hash, SC, CC*)
response = LMv2 | CC | NTv2 | CC*
```

#### NTLMv2 Session

The NTLM2 Session protocol is similar to [MS-CHAPv2](https://en.wikipedia.org/wiki/MS-CHAP). It consists of authentication from NTLMv1 combined with session security from NTLMv2.

Briefly, the NTLMv1 algorithm is applied, except that an 8-byte client challenge is appended to the 8-byte server challenge and MD5-hashed. The least 8-byte half of the hash result is the challenge utilized in the NTLMv1 protocol. The client challenge is returned in one 24-byte slot of the response message, the 24-byte calculated response is returned in the other slot.

This is a strengthened form of NTLMv1 which maintains the ability to use existing Domain Controller infrastructure yet avoids a dictionary attack by a rogue server.

The format followed for this version is:

```
NTLMv1
  Client<-Server:  SC
  Client->Server:  H(P,SC)
  Server->DomCntl: H(P,SC), SC
  Server<-DomCntl: yes or no

NTLM2 Session
  Client<-Server:  SC
  Client->Server:  H(P,H'(SC,CC)), CC
  Server->DomCntl: H(P,H'(SC,CC)), H'(SC,CC)
  Server<-DomCntl: yes or no
```

### Prevalence

Since 2010, Microsoft no longer recommends NTLM in applications. Despite these recommendations, NTLM is still widely deployed on systems. A major reason is to maintain compatibility with older systems.

Also, **NTLM authentication is used when a client authenticates to a server by IP address** (instead of by hostname), or if the user attempts to authenticate to a hostname that is not registered on the Active Directory-integrated DNS server. Likewise, third-party applications may choose to use NTLM authentication instead of Kerberos.

Even with its relative weaknesses, completely disabling and blocking NTLM authentication requires extensive planning and preparation as it's an important fallback mechanism and used by many third-party applications. Therefore, it's fairly common to encounter enabled NTLM authentication in a target environment.

### Drawbacks

As with any other cryptographic hash, NTLM cannot be reversed. However, it is considered a _fast-hashing_ algorithm since short passwords can be cracked quickly using modest equipment.

By using cracking software like Hashcat with top-of-the-line graphic processors, it is possible to test over 600 billion NTLM hashes every second. This means that eight-character passwords may be cracked within 2.5 hours and nine-character passwords may be cracked within 11 days.

## Kerberos

Computer-network authentication protocol that works on the basis of **tickets** to allow nodes communicating over a non-secure network to prove their identity to one another in a secure manner.Its designers aimed it primarily at a **client–server model**, and it provides **mutual authentication**, where both the user and the server verify each other's identity. Kerberos protocol messages are **protected against eavesdropping and replay attacks**.Kerberos builds on symmetric-key cryptography and requires a trusted third party, and optionally may use public-key cryptography during certain phases of authentication. Kerberos uses **UDP port 88** by default. Protocol was originally developed by MIT.Windows 2000 and later versions use Kerberos as the default authentication method. Microsoft added some features to Kerberos (documented in RFC 3244)

### Protocol Overview <a href="#protocol-overview" id="protocol-overview"></a>

The client authenticates itself to the **Authentication Server** (**AS**) which forwards the username to a [**key distribution center**](https://en.wikipedia.org/wiki/Key\_distribution\_center) (**KDC**).&#x20;

The KDC issues a **ticket-granting ticket** (**TGT**), which is time stamped and encrypts it using the **ticket-granting service's** (**TGS**) secret key and returns the encrypted result to the user's workstation. This is _done infrequently_, typically at user logon; the TGT expires at some point although it may be transparently renewed by the user's session manager while they are logged in.

When the client needs to communicate with a service on another node (a "principal", in Kerberos parlance), the client sends the TGT to the TGS, which usually shares the same host as the KDC. The service must have already been registered with the TGS with a Service Principal Name (SPN). The client uses the SPN to request access to this service. After verifying that the TGT is valid and that the user is permitted to access the requested service, the TGS issues ticket and session keys to the client. The client then sends the ticket to the service server (SS) along with its service request.

### Operation Details

The protocol is detailed below. There is also a [diagram](./#diagram) illustrating the concepts below. Keep in mind in an AD environment the KDC (made up of the AS and TGS) is the domain controller.

#### Client Authentication to AS

When a client attempts to authenticate to the AS, the following steps are used:

1. Client sends a clear-text message of the user ID to the **Authentication Server** (**AS**) requesting services on behalf of the user. This message is known as the _AS Request_ (`AS-REQ`)
2. If client is in AS's database, the AS generates the secret key by hashing the password of the user found at the database (e.g. Active Directory in Windows Server) and sends two messages back to the client known as the _AS Reply_ (`AS-REP`):
   * **Message A:** Client/Ticket-Granting-Service (TGS) Session Key. This is a symmetric key that can be used for the rest of the session
     * **Direction:** KDC (AS) to client
     * **Encrypted With:** Client's secret key
       * The "secret" key is actually based on the user's password. On the client end the password is entered by the user. On the server's end, it has access to the AD database and can obtain the user's (hashed) password and thus the key
       * The user's password is looked up in the [`Ntds.dit`](https://attack.mitre.org/techniques/T1003/003/) file
   * **Message B:** Ticket Granting Ticket (TGT)
     * **Direction:** KDC (AS) to client
     * **Encrypted With:** TGS's secret key
       * The TGT is not decrypted by the client, it is just stored and presented whenever it is needed in later interactions
       * The key is the (NTLM hash of the [`KRBTGT`](./#krbtgt) account)
     * **Consists Of:** Client ID, client [network address](https://en.wikipedia.org/wiki/Network\_address), ticket validity period, and the Client/TGS Session Key. This is all then encrypted with the TGS's secret key
3. Once the client receives messages A and B, it attempts to decrypt message A with the secret key generated from the password entered by the user
   * If the user entered password does not match the password in the AS database, the client's secret key will be different and thus unable to decrypt message A
   * With a valid password and secret key the client decrypts message A to obtain the **Client/TGS Session Key**
     * This session key is used for further communications with the TGS

#### Client Service Authorization

When a client wants to use a service on the network it must request authorization via the steps:

1. Client sends the following messages to the TGS known as the _TGS Request_ (`TGS-REQ`):
   * **Message C:** Service request
     * **Direction:** Client to KDC (TGS)
     * **Composed Of:** TGT and ID of requested service
       * ID of requested service is a Service Principal Name (SPN) of the form `spn/host`
         * E.g. `MSSqlSvc/SQL.domain.com`. requests the MSSQL service via its SPN (`MSsqlSvc`) on the host machine `SQL.domain.com`.
     * **Encrypted With:** Message C as a whole is not encrypted, but recall the TGT is encrypted with the TGS's secret key and is basically just an encrypted blob to the client
   * **Message D:** Authenticator
     * **Direction:** Client to KDC (TGS)
     * **Composed Of:** Client ID and timestamp
     * **Encrypted With:** Client/TGS Session Key (established in Message A)
2. The TGS conducts the following steps for validation:
   1. Extracts the "Message B" portion from Message C and decrypts it using its (the TGS's) secret key
      * This gives the TGS access to the Client/TGS session key and the client ID (both of which are contained in the unencrypted TGT)
   2. Message D is then decrypted using the Client/TGS session key
      * This yields another copy of the client ID and a timestamp
      * The TGS ensures the timestamp is not a duplicate of one it has seen before as that could be an indication of a replay attack
   3. The client IDs retrieved from messages C and D are compared and if they match the TGS sends the following 2 messages to the client known as the _TGS Reply_ (`TGS-REP`):
      * **Message E:** Client-To-Service Key
        * **Direction:** KDC (TGS) to client
        * **Consists Of:** client ID, client network address, validity period, and _Client/Service Session Key_
        * **Encrypted With:** Service's secret key
          * TGS has access to service's key in the same way it has access to the user's
          * Because this is encrypted with the service's key it will just be an encrypted blob to the client. Which is fine because the purpose of this message is mainly to safely transport the Client/Service Session Key to the service's server
      * **Message F:** Client/Service Session Key
        * **Direction:** KDC (TGS) to client
        * **Encrypted With:** Client/TGS Session Key (established in Message A)
          * This message makes the Client/Service Session Key accessible to the client machine because the contents can be decrypted as the client has access to the client/TGS session key

#### Client Service Request

At this point the client has enough information to authenticate with the service server (SS), also known as the application server (AS). To do so:

1. Client sends the SS two messages. This is known as the _AS Request_ (`AS-REQ`)
   * **Message E:** Exact copy of message from previous step. The message is effectively just forwarded from the TGS to the SS via the client
   * **Message G:** Service Authenticator
     * **Direction:** Client to SS
     * **Composed Of:** Client ID and timestamp
     * **Encrypted With:** Client/Service Session Key (passed to client in Message F)
2. The SS performs the following steps for authentication
   1. Decrypts Message E using its (the SS's) own secret key
      * This gives the SS a copy of the Client/Service session key as well as the client's ID
   2. Decrypts Message G using the Client/Service session key (extracted from Message E)
      * This gives the SS another copy of the client ID
   3. Compares the client IDs from Messages E and G, if they match the server sends the following message to confirm its identity. This message is known as the _AS Response_ (`AS-REP`):
      * **Message H:** Timestamp found in Message G&#x20;
        * **Encrypted With:** Client/Service Session Key (from decrypted Message E)
   4. Client decrypts Message H and validates the timestamp matches what was in its service authenticator message (Message G). If the timestamps match the client is authenticated and can trust the server
   5. Authentication is complete and client can begin using service

#### Diagram

The steps described above are visualized in the diagram below:

<figure><img src="../../../.gitbook/assets/AD-KerberosDiagram.png" alt=""><figcaption><p>Client authenticating to a KDC then an SS</p></figcaption></figure>

### Concepts

#### KRBTGT

The [`KRBTGT`](https://adsecurity.org/?p=483) account is a local default account that acts as a service account for the Key Distribution Center (KDC) service. This account cannot be deleted, and the account name cannot be changed. The KRBTGT account cannot be enabled in Active Directory.

KRBTGT is also the security principal name used by the KDC for a Windows Server domain, as specified by [RFC 4120](http://www.ietf.org/rfc/rfc4120.txt). The KRBTGT account is the entity for the KRBTGT security principal, and it is created automatically when a new domain is created.

Windows Server Kerberos authentication is achieved by the use of a special Kerberos ticket-granting ticket (TGT) enciphered with a symmetric key. This key is derived from the password of the server or service to which access is requested. The TGT password of the KRBTGT account is known only by the Kerberos service. In order to request a session ticket, the TGT must be presented to the KDC. The TGT is issued to the Kerberos client from the KDC.

### Drawbacks

* Kerberos has strict time requirements, which means that the clocks of the involved hosts must be synchronized within configured limits.
  * The tickets have a time availability period, and if the host clock is not synchronized with the Kerberos server clock, the authentication will fail
  * The default configuration per MIT requires that clock times be no more than five minutes apart
* The administration protocol is not standardized and differs between server implementations
  * Password changes are described in RFC 3244
* In case of symmetric cryptography adoption (Kerberos can work using symmetric or asymmetric (public-key) cryptography), since all authentications are controlled by a centralized key distribution center (KDC), compromise of this authentication infrastructure will allow an attacker to impersonate any user.
* Each network service that requires a different host name will need its own set of Kerberos keys. This complicates virtual hosting and clusters.
* Kerberos requires user accounts and services to have a trusted relationship to the Kerberos token server.
* The required client trust makes creating staged environments (e.g., separate domains for test environment, pre-production environment and production environment) difficult: Either domain trust relationships need to be created that prevent a strict separation of environment domains, or additional user clients need to be provided for each environment.

## PKI in AD

[**Public key infrastructure**](https://en.wikipedia.org/wiki/Public\_key\_infrastructure) (**PKI**) is a set of roles, policies, hardware, software and procedures needed to create, manage, distribute, use, store and revoke digital certificates and manage public-key encryption. The purpose of a PKI is to facilitate the secure electronic transfer of information for a range of network activities such as e-commerce, internet banking and confidential email.

### AD CS

Microsoft provides the AD role [`Active Directory Certificate Services`](https://learn.microsoft.com/en-us/training/modules/implement-manage-active-directory-certificate-services/)  (`AD CS`) to implement a PKI, which exchanges digital certificates between authenticated users and trusted resources. AD CS provides all PKI-related components as role services. Each role service is responsible for a specific portion of the certificate infrastructure while working together to form a complete solution.

The AD CS role includes the following role services:

* **Certification Authority:** The main purposes of CAs are to issue certificates, to revoke certificates, and to publish authority information access (AIA) and revocation information.
* **Certification Authority Web Enrollment:** This component provides a method to issue and renew certificates in scenarios where users use devices that are not joined to the domain or are running operating systems other than Windows.
* **Online Responder:** This component can be used to configure and manage Online Certificate Status Protocol (OCSP) validation and revocation checking.
* **Network Device Enrollment Service (NDES):** With this component, routers, switches, and other network devices can obtain certificates from AD CS.
* **Certificate Enrollment Web Service** **(CES):** This component works as a proxy client between a computer running Windows and the CA. CES enables users, computers, or applications to connect to a CA by using web services
* **Certificate Enrollment Policy Web Service:** This component enables users to obtain certificate enrollment policy information. Combined with CES, it enables policy-based certificate enrollment in scenarios where user devices are not joined to the domain or can't connect to a domain controller

#### Deployment Types

When using AD CS, one can deploy two types of CAs: **standalone** and **enterprise**. These types of CAs are not about hierarchy, but instead, about functionality and integration with AD DS. A standalone CA doesn't depend on AD DS. An enterprise CA requires AD DS, to provide additional functionality, such as autoenrollment. Autoenrollment allows domain users and domain-joined devices to enroll automatically for certificates after you enable automatic certificate enrollment through Group Policy.
