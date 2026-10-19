# Computer Networks — Module 4

## 1. DHCP (Dynamic Host Configuration Protocol)

- Application-layer client–server protocol (RFC 2131) that **automatically assigns IP configuration** to hosts: IP address, subnet mask, default gateway, DNS server, lease time.
- Runs over **UDP**: server port **67**, client port **68**. Extension of BOOTP.
- Needed because manual configuration is slow and error-prone, and addresses can be reused (pool) when hosts leave.

**Allocation methods**
1. **Manual** – admin maps a MAC address to a fixed IP.
2. **Automatic** – permanent IP taken from the pool the first time.
3. **Dynamic** – IP **leased** for a limited time, returned to pool afterwards.

**Operation – DORA**
1. **DHCP DISCOVER** – client broadcasts (src 0.0.0.0 → dst 255.255.255.255) to find servers.
2. **DHCP OFFER** – server(s) offer an IP + parameters + lease time.
3. **DHCP REQUEST** – client broadcasts which offer it accepts (others withdraw their offers).
4. **DHCP ACK** – server confirms and lease starts. (NAK if the request is invalid.)

**Other messages:** DECLINE (address already in use), RELEASE (client gives address back), INFORM (client already has an IP, wants other parameters).

**Lease timers:** at **T1 = 50 %** of lease the client unicasts a renewal to the server; if no answer, at **T2 = 87.5 %** it broadcasts to any server (rebinding); if the lease expires the client must stop using the address.

**Relay agent:** DHCP discover is a broadcast and does not cross routers. A relay agent on the subnet forwards it as unicast to the DHCP server, so one server can serve many networks.

**Message format fields:** Op, Htype, Hlen, Hops, Transaction ID (xid), Secs, Flags, ciaddr (client IP), yiaddr (your IP), siaddr (server IP), giaddr (relay gateway IP), chaddr (client MAC), sname, file, Options.

---

## 2. OSPF (Open Shortest Path First)

- **Interior Gateway Protocol (IGP)** used inside one Autonomous System. **Link-state** protocol (RFC 2328 for v2).
- Carried directly over IP, **protocol number 89** (no TCP/UDP). Multicast addresses **224.0.0.5** (all OSPF routers) and **224.0.0.6** (DR/BDR).
- Every router builds an identical **Link State Database (LSDB)** (a map of the whole area) and runs **Dijkstra's SPF algorithm** to get shortest-path tree.
- **Metric = cost**, usually `reference bandwidth (100 Mbps) / link bandwidth`.

**Hierarchy / Areas**
- AS divided into **areas**; **Area 0 = backbone**, all other areas connect to it.
- Router types: **Internal router**, **Backbone router**, **Area Border Router (ABR)** (connects areas), **AS Boundary Router (ASBR)** (connects to other ASes).
- Areas reduce LSDB size, flooding, and SPF load.

**Packet types (5)**
1. **Hello** – discover neighbours, keepalive (every 10 s, dead interval 40 s on broadcast links), elect DR/BDR.
2. **DBD (Database Description)** – summary of LSDB.
3. **LSR (Link State Request)** – ask for specific LSAs.
4. **LSU (Link State Update)** – carries LSAs.
5. **LSAck** – acknowledges LSUs (reliable flooding).

**Operation**
1. Hello exchange → neighbour adjacency formed.
2. On multi-access (Ethernet) networks a **Designated Router (DR)** and **Backup DR** are elected so routers sync only with DR (reduces traffic).
3. Routers exchange DBD/LSR/LSU until LSDBs are identical.
4. Each router runs SPF → routing table.
5. Any link change is flooded as LSA; SPF is rerun.

**Link types:** point-to-point, transient (broadcast network), stub, virtual link.

**Advantages:** fast convergence, no count-to-infinity, supports VLSM/CIDR, equal-cost multipath, authentication, hierarchical.
**Disadvantages:** complex to configure, memory/CPU heavy.

---

## 3. BGP (Border Gateway Protocol)

- **Exterior Gateway Protocol (EGP)** that exchanges routing between **Autonomous Systems** — the routing protocol of the Internet (BGP-4, RFC 4271).
- **Path-vector** protocol: advertises the whole **AS path** to a destination, which gives loop prevention (reject route if own AS number is in the path) and allows **policy-based** routing (business rules, not just shortest path).
- Uses **TCP port 179** (reliable; no need for its own retransmission).
- Peers are manually configured neighbours.

**Types:** **eBGP** between different ASes, **iBGP** between routers inside the same AS (spreads external routes within the AS).

**AS types:** stub, multihomed, transit.

**Messages**
1. **OPEN** – start session; AS number, hold time, router ID.
2. **UPDATE** – advertise new route or withdraw old ones (NLRI + path attributes).
3. **KEEPALIVE** – keeps session alive (default 60 s, hold time 180 s).
4. **NOTIFICATION** – error; session closed.

**Path attributes:** ORIGIN, **AS_PATH**, **NEXT_HOP**, LOCAL_PREF, MED, community, atomic aggregate.

**Route selection (order):** highest LOCAL_PREF → shortest AS_PATH → lowest ORIGIN → lowest MED → prefer eBGP over iBGP → lowest IGP cost to next hop → lowest router ID.

**Features:** policy control, CIDR/route aggregation, scalable, slower convergence than IGPs.

---

## 4. IPv6 — Features and Header

**Why:** IPv4 (32-bit, ~4.3 billion addresses) ran out.

**Features**
- **128-bit address** (2^128), written as 8 groups of 4 hex digits, e.g. `2001:0db8:0000:0000:0000:ff00:0042:8329` → compressed `2001:db8::ff00:42:8329` (leading zeros dropped, one `::` for consecutive zero groups).
- **Fixed 40-byte, simplified header** – faster routing.
- **No header checksum** and **no fragmentation by routers** (only the source fragments; uses Path MTU Discovery).
- **Extension headers** replace IPv4 options.
- **Autoconfiguration** (SLAAC, plus DHCPv6) – plug and play.
- **Built-in security** – IPsec (AH, ESP) support.
- **Better QoS** – Traffic Class and Flow Label.
- **Address types:** unicast, multicast, anycast. **No broadcast**.
- **Neighbour Discovery (ICMPv6)** replaces ARP.
- Better mobility support, allows larger payloads (jumbograms).

**Header fields (40 bytes)**

| Field | Size | Purpose |
|---|---|---|
| Version | 4 bits | = 6 |
| Traffic Class | 8 bits | priority / DSCP |
| Flow Label | 20 bits | identifies a flow needing special handling |
| Payload Length | 16 bits | length of data after header (incl. extension headers) |
| Next Header | 8 bits | type of next header (extension or TCP/UDP) |
| Hop Limit | 8 bits | like TTL; discarded at 0 |
| Source Address | 128 bits | |
| Destination Address | 128 bits | |

**Extension headers:** Hop-by-Hop, Routing, Fragment, Destination Options, Authentication (AH), ESP.

**IPv4 fields removed:** header length, identification/flags/fragment offset (moved to fragment extension header), header checksum, options.

---

## 5. TCP (Transmission Control Protocol)

- Transport layer protocol: **connection-oriented, reliable, byte-stream, full-duplex, process-to-process** delivery using port numbers (socket = IP + port).

**Segment header** (20–60 bytes)
- Source port (16), Destination port (16)
- Sequence number (32), Acknowledgement number (32)
- Header length/data offset (4), Reserved
- **Flags (6):** URG, ACK, PSH, RST, SYN, FIN
- Window size (16), Checksum (16), Urgent pointer (16)
- Options + padding

**Connection establishment – three-way handshake**
1. Client → **SYN** (seq = x)
2. Server → **SYN + ACK** (seq = y, ack = x+1)
3. Client → **ACK** (ack = y+1)

**Connection termination – four-way**
FIN → ACK → FIN → ACK (each side closes its direction independently; client waits in TIME_WAIT).

**Flow control:** **sliding window**; receiver advertises window size (rwnd) so sender does not overflow it. Zero-window handled by probes.

**Error control:** checksum, sequence numbers, cumulative ACKs, retransmission on timeout (RTO from RTT estimate), **fast retransmit** after 3 duplicate ACKs.

**Congestion control:** window = min(rwnd, cwnd). **Slow start** (exponential) → **congestion avoidance** (linear after ssthresh) → on timeout cwnd = 1 and ssthresh = cwnd/2; on 3 dup ACKs, **fast recovery**.

**Uses:** HTTP, FTP, SMTP, SSH, where reliability matters.

---

## 6. TCP vs UDP

| | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (handshake) | Connectionless |
| Reliability | Reliable: ACK, retransmission | Unreliable, best effort |
| Ordering | In-order delivery | No ordering |
| Header | 20–60 bytes | 8 bytes |
| Flow & congestion control | Yes | No |
| Speed / overhead | Slower, more overhead | Faster, less overhead |
| Data unit | Byte stream (segments) | Datagrams (message boundaries kept) |
| Broadcast/multicast | No | Yes |
| Examples | HTTP, FTP, SMTP, Telnet | DNS, DHCP, VoIP, streaming, TFTP, SNMP |

---

## 7. BOOTP (Bootstrap Protocol)

- Application-layer protocol (RFC 951) that lets a **diskless workstation** find **its IP address, the server's IP address and the name of its boot file** at startup. Runs over **UDP** (server 67, client 68). It was designed as an improvement on RARP, and DHCP later replaced it.

**Operation**
1. Client broadcasts **BOOTREQUEST** (src 0.0.0.0, dst 255.255.255.255) with its MAC address in `chaddr`.
2. BOOTP server looks up the MAC in its **static table** and replies with **BOOTREPLY** (client IP, server IP, boot file name).
3. If server is on another network, a **relay agent** forwards the request.
4. Client then downloads the boot image using **TFTP**.

**Message fields:** Op, Htype, Hlen, Hops, Transaction ID, Seconds, Flags, ciaddr, yiaddr, siaddr, giaddr, chaddr (16), sname (64), file (128), vendor-specific (64).

**Limitations:** static one-to-one MAC→IP mapping set by hand, no lease or dynamic allocation, no reuse of addresses.

**BOOTP vs DHCP**

| | BOOTP | DHCP |
|---|---|---|
| Allocation | Static/manual table | Dynamic, automatic, manual |
| Lease | No | Yes |
| Address reuse | No | Yes |
| Options | Limited (64-byte vendor field) | Large variable options field (312 bytes) |
| Boot file | Gives boot file name | Mostly gives network settings |
