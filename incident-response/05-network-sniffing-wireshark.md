# Lab 05 — Network Sniffing with Wireshark

**Source:** CompTIA Security+ CertMaster Labs — Data Sources (Section 12.1.11)
**Environment:** KALI Linux → Wireshark → dvwa.structureality.com
**Tool:** Wireshark
**Techniques:** Packet capture, display filters, TCP/HTTP stream analysis
**Security+ Objective:** 4.9 — Given a scenario, use data sources to support an investigation
**Status:** ✅ Complete

---

## Scenario

In this lab, network sniffing and packet capture techniques were explored using Wireshark. The exercises covered three progressively deeper capabilities: capturing live traffic on a network interface, applying display filters to isolate specific packets from a large capture, and using Wireshark's Follow Stream function to reconstruct full TCP and HTTP conversations.

---

## Environment

| System | Role | IP |
|--------|------|----|
| KALI | Capture workstation | 10.1.16.66 |
| dvwa.structureality.com | Target web application | — |
| www.structureality.com | Company website | 172.16.0.201 |

---

## Background: Packets vs Frames

While the term "packet" is commonly used in network sniffing contexts, Wireshark captures **Ethernet frames** — the full data unit at Layer 2 (Data Link). The distinction matters:

| OSI Layer | PDU Name | Example |
|-----------|----------|---------|
| Layer 2 — Data Link | Frame | Ethernet frame |
| Layer 3 — Network | Packet | IP packet |
| Layer 4 — Transport (TCP) | Segment | TCP segment |
| Layer 4 — Transport (UDP) | Datagram | UDP datagram |

The Wireshark three-pane display:
- **Top (Packet List):** Frame summary — source, destination, protocol, info
- **Middle (Packet Details):** Expandable protocol headers (Ethernet, IPv4, TCP, HTTP)
- **Bottom (Packet Bytes):** Raw hexadecimal + ASCII representation of the frame

---

## Exercise 1 — Capture Network Traffic

### Step 1 — Initiate a Capture

Launched Wireshark from the Kali Applications menu. Selected the **eth0** interface and started a live capture.

With the capture running, navigated Firefox to `www.structureality.com` to generate HTTP traffic.

Stopped the capture after a few seconds.

### Step 2 — Identify the Target IP

Applied a simple HTTP display filter to locate the initial GET request from Kali to www.structureality.com:

**Filter used:** Wireshark display filter targeting HTTP traffic from the Kali IP (10.1.16.66)

**Finding:**

| Field | Value |
|-------|-------|
| IP address of www.structureality.com | **172.16.0.201** |

> Scored question answered correctly: 172.16.0.201

### Step 3 — Check for DNS Traffic

Applied DNS display filter:

```
dns
```

DNS query and response packets were visible in the capture — confirming DNS resolution activity was recorded.

---

## Exercise 2 — Using Display Filters

Started a new capture and visited `dvwa.structureality.com`, then stopped the capture.

### Filter 1 — Filter by Destination IP

```
ip.dst==10.1.16.66
```

Result: Only frames with Kali (10.1.16.66) as the destination were displayed — inbound traffic only.

### Filter 2 — Filter by TTL Value

```
ip.ttl<128
```

Result: Only frames with an IP TTL (Time to Live) value below 128 were displayed.

### Filter 3 — Add ARP with OR Logic

```
ip.ttl<128 or arp
```

Result: Frames with TTL < 128 **or** ARP communications — combined filter using logical OR.

> Note: ARP packets cannot contain both an IP header (with a TTL) and ARP simultaneously. OR logic is required here — AND would return zero results.

### Filter 4 — Exclude a Specific IP (NOT logic)

```
ip.addr!=10.1.16.66
```

Result: All frames that do **not** contain 10.1.16.66 as either source or destination.

### Filter 5 — TCP FIN Flag (Display Filter Expression)

Built using the Display Filter Expression GUI:
- Protocol: TCP
- Field: `tcp.flags.fin`
- Relation: `==`
- Value: `1`

Resulting filter: `tcp.flags.fin == 1`

Result: Only frames with the TCP FIN flag set — connection teardown packets.

**Finding:**

| Field | Value |
|-------|-------|
| IPv4 source address of first FIN frame | **10.1.16.68** |

> Scored question answered correctly: 10.1.16.68

### Filter 6 — FIN set, ACK not set

Modified filter to exclude ACK flags:

```
tcp.flags.fin == 1 and tcp.flags.ack == 0
```

Result: No packets — it is rare to see a FIN without an ACK, as TCP connection teardown typically involves FIN+ACK exchanges.

### Filter 7 — FIN set, SYN not set

```
tcp.flags.fin == 1 and tcp.flags.syn == 0
```

Result: FIN packets that are not part of a SYN handshake — clean FIN teardown frames.

---

## Exercise 3 — Follow a TCP/HTTP Stream

### Step 1 — Follow TCP Stream

With dvwa.structureality.com traffic captured, applied a display filter to find the first frame of the communication:

```
tcp contains "dvwa.structureality.com"
```

Selected the first result, then: **Analyze → Follow → TCP Stream**

**Observation:** The TCP stream payload was compressed/encoded — mostly unreadable in raw form. The stream is colour-coded: **red = client (Kali)**, **blue = server (DVWA)**.

Closed the Follow TCP Stream window.

### Step 2 — Follow HTTP Stream

Opened a **Follow HTTP Stream** from a frame containing the HTTP request to dvwa.structureality.com.

**Observation:** HTTP payload now visible in ASCII/plaintext — the HTTP stream decompresses and decodes the content automatically.

Located the HTML code line from the web server's response:

```html
<h1>Welcome to Damn Vulnerable Web Application!</h1>
```

| Field | Value |
|-------|-------|
| Colour (server side) | **Blue** |
| Side | **Server** |

> Scored question answered correctly: Server / Blue

> **Important limitation:** Follow HTTP Stream only works for unencrypted HTTP traffic. If the session used HTTPS (TLS), the HTTP payload would not be readable. The TCP stream would still be visible (TCP headers are plaintext), but the payload would appear as encrypted ciphertext.

---

## Display Filter Reference

| Filter | Effect |
|--------|--------|
| `ip.dst==10.1.16.66` | Destination IP match |
| `ip.ttl<128` | TTL below 128 |
| `ip.ttl<128 or arp` | TTL < 128 OR ARP frames |
| `ip.addr!=10.1.16.66` | Exclude all frames involving this IP |
| `tcp.flags.fin == 1` | TCP FIN flag set |
| `tcp.flags.fin == 1 and tcp.flags.ack == 0` | FIN without ACK |
| `tcp.flags.fin == 1 and tcp.flags.syn == 0` | FIN without SYN |
| `dns` | DNS protocol frames |
| `tcp contains "string"` | TCP payload contains string |

---

## Key Takeaway

Wireshark is one of the most powerful investigation tools available to a security professional — and one of the most misused. Raw packet captures contain the full network conversation including credentials, session tokens, and sensitive data if sent over unencrypted protocols. In this lab, the HTTP stream of DVWA traffic was fully readable because it used plain HTTP. In a production environment, any sensitive application transmitting over HTTP rather than HTTPS represents a serious risk — a network-level attacker with access to the same segment can capture and reconstruct the entire session, as demonstrated here.

---

*Write-up by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/) | [Back to Incident Response](./README.md)*
