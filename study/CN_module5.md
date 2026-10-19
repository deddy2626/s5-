# Computer Networks — Module 5 (Application Layer & Security)

## 1. Functions and Purpose of Application Layer Protocols

The application layer is the topmost layer; it gives **user applications access to network services**. In the TCP/IP model it also covers the OSI session and presentation functions.

**Purpose / functions**
- Interface between user software and the network (web, mail, file transfer, remote login)
- **Name resolution** (DNS) and **address configuration** (DHCP)
- Data formatting, encoding and compression of messages (MIME, HTML)
- Session setup/teardown for applications, user **authentication** and access control
- Resource sharing, remote file access, directory services, network management (SNMP)
- Define message syntax, order of messages and rules (request/response); client–server or peer-to-peer model

| Protocol | Service | Transport / Port |
|---|---|---|
| HTTP / HTTPS | Web | TCP 80 / 443 |
| FTP | File transfer | TCP 20 (data), 21 (control) |
| TFTP | Simple file transfer | UDP 69 |
| SMTP | Send mail | TCP 25 (587 submission) |
| POP3 / IMAP | Retrieve mail | TCP 110 / 143 |
| DNS | Name → IP | UDP/TCP 53 |
| DHCP / BOOTP | IP config | UDP 67, 68 |
| Telnet / SSH | Remote login | TCP 23 / 22 |
| SNMP | Network management | UDP 161 |

---

## 2. DNS (Domain Name System)

- A **distributed, hierarchical database** that maps **host names ↔ IP addresses**, so people use `www.example.com` instead of numbers.
- Uses **UDP port 53** for queries; **TCP 53** for zone transfers and responses larger than 512 bytes.

**Name space (tree):** root (.) → **Top-Level Domains** (generic: .com .org .edu; country: .in .uk) → second-level (example) → subdomains. **FQDN** ends with a dot, e.g. `www.example.com.`

**Name servers**
- **Root servers** – know TLD servers.
- **TLD servers** – know authoritative servers for each domain.
- **Authoritative servers** – hold the actual records of a zone.
- **Local/Resolver (ISP) server** – receives the client's query, caches answers.
- A **zone** = portion of the tree managed by one server (primary + secondary for redundancy).

**Resolution**
- **Recursive:** server does all work and returns the final answer to the client.
- **Iterative:** server returns a referral ("ask this server"), client follows referrals.
- Typical case: host → local resolver (recursive) → root → TLD → authoritative (iterative).
- **Caching** with TTL reduces traffic and delay.

**Resource Record types:** A (name→IPv4), AAAA (IPv6), NS (name server), CNAME (alias), MX (mail server), PTR (IP→name, reverse lookup via `in-addr.arpa`), SOA (zone authority), TXT.

**Message format:** Header 12 bytes (Identification, Flags [QR, Opcode, AA, TC, RD, RA, RCODE], counts of questions/answers/authority/additional) followed by Question, Answer, Authority and Additional sections.

---

## 3. MIME (Multipurpose Internet Mail Extensions)

**Problem:** SMTP can only send **7-bit NVT ASCII text**; it cannot carry non-English characters, images, audio, video or binary files.

**MIME** is a supplementary protocol that lets **non-ASCII data be sent through SMTP** by encoding it into ASCII at the sender and decoding at the receiver. It adds headers to the normal RFC 822 message (headers, blank line, body).

**MIME headers**
- `MIME-Version: 1.0`
- `Content-Type: type/subtype` – nature of data
- `Content-Transfer-Encoding:` – how the data is encoded
- `Content-ID` – unique id for a part
- `Content-Description` – human readable description

**Content types:** text (plain, html), image (jpeg, gif), audio, video (mpeg), application (pdf, octet-stream), **multipart** (mixed, alternative, parallel, digest – several parts separated by a `boundary`), message (rfc822, partial).

**Transfer encodings:** 7bit, 8bit, binary (not allowed by SMTP as-is), **base64** (3 bytes → 4 ASCII chars, ~33% overhead), **quoted-printable** (mostly ASCII text with a few special characters shown as `=XX`).

**Example**
```
From: a@x.com
To: b@y.com
MIME-Version: 1.0
Content-Type: image/jpeg
Content-Transfer-Encoding: base64

/9j/4AAQSkZJRgAB...
```

---

## 4. FTP (File Transfer Protocol)

- Transfers files between a client and a server over **TCP**; reliable, supports authentication.
- **Two connections:**
  - **Control connection – port 21**: stays open during the session, carries commands and replies (NVT ASCII).
  - **Data connection – port 20**: opened for each file transfer, closed afterwards.
- **Active mode:** server initiates the data connection from port 20 to client's port. **Passive mode:** client initiates to a server port (works through firewalls/NAT).

**Common commands:** USER, PASS, LIST, CWD, PWD, RETR (download), STOR (upload), DELE, MKD, QUIT.
**Replies** are 3-digit codes: 220 service ready, 230 login ok, 150 file status ok, 226 transfer complete, 331 password needed, 425 can't open data connection, 530 not logged in.

**Data representation**
- File type: ASCII, EBCDIC, Image (binary)
- Data structure: file, record, page
- Transmission mode: stream (default), block, compressed

**Anonymous FTP** – public access with user `anonymous`.
**Drawback:** username/password sent in clear text → use FTPS (FTP over TLS) or SFTP (over SSH).

---

## 5. Electronic Mail

**Architecture**
- **User Agent (UA)** – mail program (Outlook, Thunderbird) to compose/read.
- **Message Transfer Agent (MTA)** – mail server that transfers mail with SMTP.
- **Message Access Agent (MAA)** – POP3 / IMAP that fetches mail from the mailbox.

**Flow:** Sender UA → (SMTP push) sender's MTA → (SMTP) → receiver's MTA → receiver's mailbox → (POP3/IMAP pull) receiver's UA.

**Address format:** `local-part@domain`. The MTA finds the destination server with a DNS **MX** record.

**SMTP (Simple Mail Transfer Protocol)** – TCP port 25 (587), push protocol.
- Commands: HELO/EHLO, MAIL FROM, RCPT TO, DATA, RSET, QUIT.
- Replies: 220 ready, 250 OK, 354 start mail input, 221 closing, 550 mailbox unavailable.
- Phases: connection setup → mail transfer → termination.

**POP3 vs IMAP**

| | POP3 (110) | IMAP4 (143) |
|---|---|---|
| Mail storage | Downloaded to client, usually deleted from server | Stays on server |
| Multiple devices | Poor | Good |
| Folders on server | No | Yes |
| Partial download | No | Yes (headers first, search) |
| Complexity | Simple | More complex |

**Webmail** (Gmail) uses HTTP between browser and server, SMTP between servers.
**Message format:** header (From, To, Date, Subject, Cc) + blank line + body; MIME extends it.

---

## 6. Types of Cyber Attackers

1. **Hackers** – *White hat* (ethical, authorized), *Black hat* (malicious, for gain), *Grey hat* (no permission, but usually no harm).
2. **Script kiddies** – inexperienced; use readymade tools and scripts.
3. **Cyber criminals** – organized crime, financial gain (fraud, ransomware, theft).
4. **Hacktivists** – political/social cause (defacement, DDoS).
5. **State-sponsored / nation-state actors** – espionage and cyber warfare, well funded (APTs).
6. **Insiders** – employees or contractors misusing access (disgruntled or careless).
7. **Cyber terrorists** – aim to cause fear/disruption to critical infrastructure.
8. **Competitors / industrial spies** – steal trade secrets.

---

## 7. Types of Security Threats in Networks

**Security goals (CIA):** Confidentiality, Integrity, Availability (plus authentication, non-repudiation).

**Passive attacks** (don't change data; hard to detect): eavesdropping/**sniffing**, traffic analysis.

**Active attacks** (modify or disrupt): masquerading/**spoofing** (IP, ARP, DNS, MAC), **replay**, message modification, **Denial of Service (DoS/DDoS)**.

**Common threats**
- **Malware:** virus (needs host file), worm (self-replicates over network), Trojan (disguised), ransomware, spyware, rootkit, botnet.
- **Phishing / social engineering** – trick users into revealing credentials.
- **Man-in-the-Middle (MITM)** – attacker intercepts/alters communication.
- **DoS / DDoS** – flood target (SYN flood, ping flood) to exhaust resources.
- **Password attacks** – brute force, dictionary.
- **SQL injection / XSS** – web application attacks.
- **Session hijacking**, **IP spoofing**, **DNS poisoning**.
- **Zero-day exploits** – unknown vulnerabilities.
- **Insider threats.**

**Countermeasures:** firewalls, IDS/IPS, encryption (SSL/TLS, IPsec, VPN), strong authentication/MFA, antivirus, patching, backups, user awareness.
