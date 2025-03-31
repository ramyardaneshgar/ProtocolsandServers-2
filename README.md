# THM-Writeup-Protocols-and-Servers-2
Writeup for TryHackMeProtocols and Servers 2 Lab - network-layer attack vectors and defense mechanisms using tools like Hydra, tcpdump, SSH, and secure protocols (TLS, POP3S, IMAPS, FTPS).

By Ramyar Daneshgar 

--

## Objective

The purpose of this lab is to explore insecure network protocols and understand how common attacks like sniffing, Man-in-the-Middle (MITM), and password-based attacks occur in unencrypted environments. I also examine how encryption-based protocols such as SSH and TLS serve as mitigations. Throughout the lab, I interact with various services, execute CLI commands, and analyze packet-level network traffic.

---

## Task 1: Introduction

This task lays the foundation by mapping protocol vulnerabilities against the CIA Triad (Confidentiality, Integrity, Availability). I reviewed how different protocols like Telnet, FTP, POP3, and SMTP inherently lack encryption, making them susceptible to eavesdropping, data tampering, and impersonation attacks.

I took note that:
- Disclosure aligns with confidentiality attacks (e.g., sniffing cleartext passwords)
- Alteration is tied to integrity issues (e.g., MITM modifying messages)
- Destruction affects availability (e.g., DoS attacks)

Understanding this model helped guide my threat classification and reasoning throughout the remainder of the lab.

---

## Task 2: Sniffing Attack

### Objective:
Capture cleartext credentials using a packet sniffer.

### Steps Taken:
1. I initiated the AttackBox and started the VM to ensure I had network access.
2. I used `tcpdump` with the following command:
   ```bash
   sudo tcpdump port 110 -A
   ```
   - I used `sudo` to run with root privileges since packet capture requires elevated access.
   - I filtered for port `110` because POP3 operates on this port.
   - I added `-A` to display printable ASCII characters to visually extract credentials.

### Output Analysis:
The capture revealed two key packets:
- `USER frank`
- `PASS D2xc9CgD`

These were transmitted in plaintext, which confirmed the lack of encryption in the POP3 protocol.

### Lessons Learned:
- Any attacker with network-level access (e.g., switch port mirroring, wireless interception) can easily collect login credentials from protocols that transmit data in cleartext.
- Use of `tcpdump` and filtering ports helps narrow down relevant traffic in busy environments.

---

## Task 3: MITM Attack

### Objective:
Understand how MITM attacks alter data integrity when protocols do not use encryption.

### Tools Referenced:
- Ettercap
- Bettercap

### Key Findings:
I learned that:
- MITM attacks require the attacker to intercept traffic between client and server.
- These tools make it trivial to inject or modify data in real-time.

I answered the questions by examining the tools:
- Ettercap offers 3 interfaces: text, graphical, and daemon.
- Bettercap can be invoked in 3 ways: interactive, scripting, and remote control mode.

### Lessons Learned:
- MITM attacks are highly effective against unencrypted protocols.
- Defense involves strong authentication and encryption (TLS/SSL).

---

## Task 4: TLS (Transport Layer Security)

### Objective:
Understand the role of TLS in mitigating MITM and sniffing attacks.

### Key Concepts:
- TLS encrypts application-layer data using a handshake and symmetric encryption.
- TLS upgrades protocols like HTTP, SMTP, and POP3 to secure versions (HTTPS, SMTPS, POP3S).

I referred to RFC 6101 and internalized the TLS handshake process:
1. ClientHello
2. ServerHello + Certificate
3. ClientKeyExchange
4. ChangeCipherSpec + Finished messages

This process ensures:
- Confidentiality: encryption of data in transit
- Integrity: detection of tampering
- Authenticity: server certificate validation

### Answer:
- DNS over TLS is abbreviated as `DoT`

### Lessons Learned:
- TLS is a critical mitigation against sniffing and MITM attacks.
- Server certificates and CAs play a pivotal role in building trust.

---

## Task 5: SSH (Secure Shell)

### Objective:
Use SSH for secure remote access and file transfers.

### Step-by-Step:
1. I connected to the remote machine:
   ```bash
   ssh mark@MACHINE_IP
   ```
   - I used the provided credentials: username `mark` and password `XBtc49AB`
   - I accepted the server fingerprint to proceed (this validates the host key and prevents MITM attacks).

2. I confirmed the system's kernel release:
   ```bash
   uname -r
   ```
   - Output: `5.4.0-84-generic`

3. I downloaded the file `book.txt` using SCP:
   ```bash
   scp mark@MACHINE_IP:/home/mark/book.txt .
   ```
   - Output displayed: `415KB`

### Lessons Learned:
- SSH encrypts all traffic, including login credentials and command output.
- SCP leverages the SSH protocol for secure file transfers.
- First-time SSH connections should verify server fingerprints to avoid MITM risk.

---

## Task 6: Password Attack

### Objective:
Conduct a dictionary attack using Hydra against an SSH login.

### Step-by-Step:
1. I located the user `frank` from previous tasks.
2. I ran Hydra against the SSH service:
   ```bash
   hydra -l frank -P /usr/share/wordlists/rockyou.txt MACHINE_IP ssh -t 4 -V
   ```
   - `-l frank` specifies the username
   - `-P` points to a wordlist
   - `ssh` is the target service
   - `-t 4` sets concurrency
   - `-V` shows progress (helpful for debugging)

3. Once the correct password was found, I was able to login and verify credentials.

4. I used IMAP to confirm:
   ```bash
   telnet MACHINE_IP 143
   USER lazie
   PASS butterfly
   ```

### Lessons Learned:
- Password reuse and weak passwords remain major security risks.
- Dictionary attacks are fast when common passwords are used.
- Mitigations include 2FA, account lockout, and strict password policies.

---

## Task 7: Summary

I reviewed the protocols and attacks discussed:
- Sniffing affects confidentiality
- MITM impacts integrity
- Password attacks threaten authentication mechanisms

I also reinforced the mapping of insecure protocols (Telnet, FTP, POP3) to their secure alternatives (SSH, FTPS/SFTP, POP3S).

