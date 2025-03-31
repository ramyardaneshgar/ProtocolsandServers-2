# Writeup for TryHackMeProtocols and Servers 2 Lab - network-layer attack vectors and defense mechanisms using tools like Hydra, tcpdump, SSH, and secure protocols (TLS, POP3S, IMAPS, FTPS).

By Ramyar Daneshgar 

## Objective

The purpose of this lab was to explore insecure network protocols and understand how common attacks like sniffing, Man-in-the-Middle (MITM), and password-based attacks occur in unencrypted environments. I also examine how encryption-based protocols such as SSH and TLS serve as mitigations. Throughout the lab, I interact with various services, execute CLI commands, and analyze packet-level network traffic.

---

## Task 1: Introduction

This task lays the foundation by mapping protocol vulnerabilities against the CIA Triad (Confidentiality, Integrity, Availability). I reviewed how different protocols like Telnet, FTP, POP3, and SMTP inherently lack encryption, making them susceptible to eavesdropping, data tampering, and impersonation attacks. I noted how sniffing attacks compromise confidentiality, MITM attacks undermine integrity, and denial-of-service threats impact availability. This understanding helped me classify threats and reason through each protocol's risks throughout the lab.

---

## Task 2: Sniffing Attack

To demonstrate a sniffing attack, I started the AttackBox and launched the VM. I ran the following command to capture POP3 traffic:

```bash
sudo tcpdump port 110 -A
```

- `sudo`: required for raw packet capture.
- `port 110`: filters traffic to the POP3 service.
- `-A`: prints each packet in ASCII for easier credential inspection.

The output revealed:
```
USER frank
PASS D2xc9CgD
```

This showed how attackers with network access can harvest credentials from insecure protocols.

---

## Task 3: MITM Attack

This task illustrated how attackers can intercept and modify data between two parties when encryption is absent. I explored MITM tools such as Ettercap and Bettercap. Ettercap supports three interfaces: text, graphical, and daemon. Bettercap can be invoked in interactive mode, scripting mode, and remote control mode. These tools allow adversaries to manipulate real-time communications, highlighting how unencrypted protocols leave users vulnerable to silent tampering.

---

## Task 4: TLS (Transport Layer Security)

I explored TLS as a mitigation strategy to encrypt and authenticate application-layer data. Referencing RFC 6101, I studied the TLS handshake process:

1. ClientHello
2. ServerHello + Certificate
3. ClientKeyExchange
4. ChangeCipherSpec

TLS enables encrypted communication through symmetric key exchange, ensuring confidentiality, integrity, and authenticity. Cleartext protocols such as HTTP, POP3, and SMTP can be upgraded to HTTPS, POP3S, and SMTPS. DNS over TLS is abbreviated as:

```
DoT
```

---

## Task 5: SSH (Secure Shell)

I connected to the remote machine using:

```bash
ssh mark@MACHINE_IP
```

When prompted, I entered the password:
```
XBtc49AB
```

To confirm the system's kernel version:

```bash
uname -r
```

Output:
```
5.4.0-84-generic
```

I securely downloaded a file using SCP:

```bash
scp mark@MACHINE_IP:/home/mark/book.txt .
```

The displayed transfer size was:
```
415KB
```

SSH encrypted the session, protecting both authentication and terminal traffic.

---

## Task 6: Password Attack

I launched a dictionary attack using Hydra against the SSH service. The command I used:

```bash
hydra -l frank -P /usr/share/wordlists/rockyou.txt MACHINE_IP ssh -t 4 -V
```

- `-l frank`: specifies the login name.
- `-P`: points to the RockYou wordlist.
- `ssh`: defines the target service.
- `-t 4`: runs 4 parallel threads.
- `-V`: enables verbose output.

Once the password was found, I confirmed login success. Additionally, I tested IMAP credentials via Telnet:

```bash
telnet MACHINE_IP 143
```
```
USER lazie
PASS butterfly
```

This demonstrated the ease of cracking weak passwords using widely available tools.

Sniffing attacks compromise confidentiality by exposing data in transit. MITM attacks break integrity by altering data mid-transmission. Password attacks challenge authentication mechanisms by exploiting user behavior. Insecure protocols were evaluated and replaced with their secure counterparts, such as SSH for Telnet, FTPS for FTP, and IMAPS/POP3S for email access.

