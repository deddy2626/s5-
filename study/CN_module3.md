# Computer Networks — Module 3 (Network Layer)

## 1. Distance Vector Routing (4 routers)

**Idea:** Each router keeps a table of (destination, cost, next hop). Periodically it sends its whole vector to its *direct neighbours only*. On receiving a neighbour's vector, it applies **Bellman-Ford**:

`D_x(y) = min over neighbours v of { c(x,v) + D_v(y) }`

**Example topology** (link costs): A–B = 2, A–C = 5, B–C = 1, C–D = 3

**Step 0 – Initial tables** (only directly connected nodes known, others = ∞)

| | A | B | C | D |
|---|---|---|---|---|
| A | 0 | 2 | 5 | ∞ |
| B | 2 | 0 | 1 | ∞ |
| C | 5 | 1 | 0 | 3 |
| D | ∞ | ∞ | 3 | 0 |

**Round 1** (everyone uses the neighbours' *old* vectors)
- A: via B → C = 2+1 = 3 (better than 5, update, next hop B). D via C = 5+3 = 8.
- B: D via C = 1+3 = 4. A stays 2 (direct).
- C: A via B = 1+2 = 3 (better than 5). Rest unchanged.
- D: A via C = 3+5 = 8, B via C = 3+1 = 4.

**Round 2**
- A: D via B = 2 + 4 = 6 (better than 8, update, next hop B).
- D: A via C = 3 + 3 = 6 (C now knows A = 3).
- No other changes.

**Round 3** – no changes → converged.

**Final tables**

| Router | to A | to B | to C | to D |
|---|---|---|---|---|
| A | 0 | 2 (B) | 3 (B) | 6 (B) |
| B | 2 (A) | 0 | 1 (C) | 4 (C) |
| C | 3 (B) | 1 (B) | 0 | 3 (D) |
| D | 6 (C) | 4 (C) | 3 (C) | 0 |

*(Brackets = next hop.)*

**Points to write in the answer:** periodic updates, only neighbours exchanged, converges by Bellman-Ford; weakness is **count-to-infinity** (good news travels fast, bad news slow); fixes: split horizon, poison reverse, hold-down timers, max hop = 15 (RIP).

---

## 2. Optimality Principle

**Statement:** If router J is on the optimal path from router I to router K, then the optimal path from J to K lies along the same route.

**Proof (sketch):** Call the path I→J = r1 and J→K = r2. If a better route r2' from J to K existed, then r1 followed by r2' would beat r1 followed by r2, contradicting the assumption that I→J→K was optimal.

**Consequence – Sink tree:** The set of optimal paths from all sources to one destination forms a tree rooted at that destination (no loops). Routing algorithms discover the sink tree for each router. It is not necessarily unique (equal-cost paths), and it assumes that topology does not change during computation.

---

## 3. ARP vs RARP

| | ARP | RARP |
|---|---|---|
| Full form | Address Resolution Protocol | Reverse Address Resolution Protocol |
| Maps | IP address → MAC address | MAC address → IP address |
| Who asks | Host knows destination IP, needs MAC | Host knows its own MAC, needs its own IP (diskless machines) |
| Request | Broadcast | Broadcast |
| Reply | Unicast from owner of the IP | Unicast from RARP server |
| Needs server? | No – the target host replies | Yes – RARP server with MAC→IP table on each network |
| Opcode | 1 request / 2 reply | 3 request / 4 reply |
| Status | In use (cache kept) | Obsolete, replaced by BOOTP/DHCP |
| Layer | Between network and data link | Between network and data link |

---

## 4. IPv4 Fragmentation Problem

**Given:** Datagram = 2400 B (incl. 20 B header), MTU = 1000 B.

- Data in original datagram = 2400 − 20 = **2380 B**
- Max data per fragment = 1000 − 20 = 980 B
- Offset is measured in units of 8 bytes, so data per fragment must be a multiple of 8 → largest = **976 B** (976/8 = 122)
- Number of fragments = ⌈2380 / 976⌉ = **3**

| Fragment | Data | Total length | MF flag | Offset (units of 8 B) |
|---|---|---|---|---|
| 1 | 976 | 996 | 1 | 0 |
| 2 | 976 | 996 | 1 | 122 |
| 3 | 428 (2380 − 1952) | 448 | 0 | 244 |

Check: 976 + 976 + 428 = 2380 ✔. Offset of fragment 3 = 1952/8 = 244 ✔.

Same Identification value in all fragments; reassembly only at the final destination.

---

## 5. Congestion Control Techniques

Congestion = too many packets in the subnet, so performance degrades.

**A. Open loop (prevention)**
- Retransmission policy – timers that avoid unnecessary retransmits
- Window policy – selective repeat is better than go-back-N
- Acknowledgement policy – delayed/cumulative ACKs
- Discard policy – drop less important packets first
- Admission policy – virtual-circuit setup refused if congestion likely
- Traffic shaping (leaky bucket / token bucket)

**B. Closed loop (removal after it happens)**
- **Backpressure** – congested node tells the upstream node to slow down, hop by hop
- **Choke packet** – router sends a packet directly to the source asking it to reduce rate
- **Implicit signalling** – source guesses from delays/lost ACKs (TCP)
- **Explicit signalling** – bits in packets: FECN (forward), BECN (backward); ECN in IP/TCP
- **Hop-by-hop choke packets**
- **Load shedding** – router discards packets when overloaded (wine = old, milk = new policy)
- **RED (Random Early Detection)** – drop packets randomly before the queue is full

**TCP congestion control (mention for full marks):** slow start (cwnd doubles per RTT) → congestion avoidance (cwnd +1 per RTT after ssthresh) → on loss, ssthresh = cwnd/2 (fast retransmit/recovery).

---

## 6. Leaky Bucket vs Token Bucket

| | Leaky Bucket | Token Bucket |
|---|---|---|
| Output rate | Constant (fixed) | Variable, bursts allowed |
| Burst handling | Smooths bursts into steady flow | Permits bursts up to bucket size |
| Idle time | No credit saved | Tokens accumulate while idle |
| Full bucket | Excess packets **discarded** | Excess **tokens** discarded (packets wait/sent as tokens available) |
| Mechanism | Packets queue in bucket, leak at fixed rate | A token is generated every Δt; a packet needs a token to be sent |
| Use | Strict rate enforcement | Flexible shaping with bursts |
| Packet loss | Possible when bucket overflows | Packets are not lost because of tokens (queue may overflow) |

Both are traffic shaping tools used for open-loop congestion control.
