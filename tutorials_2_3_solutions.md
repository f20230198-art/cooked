# Tutorials 2 & 3 — Combined Solutions

> **Topics:** Layered protocols, OSI encapsulation, packet vs circuit switching, store-and-forward delay, congestion, latency calculations, cut-through switching, Synchronous TDM
> **Reference guide:** [introduction_osi_performance_guide.md](introduction_osi_performance_guide.md)
>
> **Tutorial 2** = Q1–Q10 (layers, ACKs, troubleshooting, basic switching)
> **Tutorial 3** = Q11–Q14 (advanced delay calculations, cut-through, TDM)

---

## 📚 Concepts You Need First (Read Before the Problems)

These weren't fully covered in your PPT but you need them to solve the problems below.

### Concept A — Header Overhead in Layered Protocols

When data goes down through layers, **each layer adds a header**. So the actual bytes on the wire are MORE than the original data:

```
Original message:                M bytes (the payload)
After Layer N adds header:       M + h bytes
After Layer N-1 adds header:     M + h + h = M + 2h bytes
...
After all n layers:              M + n·h bytes
```

**Useful fraction (payload / total):**
$$\text{Useful} = \frac{M}{M + n \cdot h}$$

**Header overhead fraction (wasted on headers):**
$$\text{Overhead} = \frac{n \cdot h}{M + n \cdot h}$$

### Concept B — Store-and-Forward Delay

When a packet passes through a switch/router, the switch:
1. **Receives the entire packet** (waits for all bits to arrive)
2. **Then forwards** it on the next link

This means **each intermediate hop adds one transmission delay** (not just propagation).

For a packet of size L over a link of bandwidth R:
- Per-hop delay = $T_{trans} + T_{prop} = L/R + d/s$

Across **N links** (N−1 intermediate switches):
- Total transmission delay = $N \times L/R$ (each hop must fully receive then send)
- Total propagation delay = $N \times T_{prop}$

### Concept C — ACK Strategies (Stop-and-Wait vs Go-Back-N / Selective Repeat)

When a sender splits a file into packets, it has two main strategies for ACKs:

| Strategy | How it works | Pros | Cons |
|---|---|---|---|
| **Per-packet ACK** (Stop-and-Wait, like) | Send packet, wait for ACK, send next | Simple, no buffer needed | SLOW — RTT wait between every packet |
| **Cumulative ACK** (Go-Back-N) | Send many packets, one ACK covers them all; on loss, resend from the lost one | Fast pipelining | If 1 lost, must resend many |
| **Selective ACK** (Selective Repeat) | ACK each packet individually but allow many in-flight | Fast + only resend what's lost | Complex, needs buffer at receiver |

### Concept D — Default Gateway

The **default gateway** is your router's IP address — the device that forwards your packets out to other networks. If `ping default-gateway` returns "Request timed out," your computer **can't even reach the router** → the problem is on your local LAN (Layer 1 or 2), not somewhere on the Internet.

---

# 🧮 PROBLEMS

---

## Q1. Header Overhead in n-Layer Protocol

> A system has an **n-layer protocol** hierarchy. Applications generate messages of length **M bytes**. At each layer, an **h-byte header** is added. What fraction of network bandwidth is filled with headers?

### Concept refresher
Each layer adds h bytes → after n layers, total headers = **n·h bytes**. Payload = M bytes. Total on wire = M + n·h.

### Solution

$$\text{Fraction of bandwidth used by headers} = \frac{n \cdot h}{M + n \cdot h}$$

### Example sanity check
If M = 100 bytes, n = 5 layers, h = 20 bytes/header:
- Header overhead = (5 × 20) / (100 + 5 × 20) = 100/200 = **50%**

So half the bandwidth is wasted on headers! This shows why **small messages are very inefficient** — most of the bandwidth goes to overhead, not data.

✅ **Answer: $\dfrac{nh}{M + nh}$**

---

## Q2. Two ACK Strategies — Discussion

> When a file is transferred between two computers, two ACK strategies are possible:
> - **Strategy 1:** File chopped into packets, each ACK'd individually, but the file as a whole is NOT ACK'd
> - **Strategy 2:** Packets NOT ACK'd individually, but the entire file IS ACK'd when it arrives
>
> Discuss these two approaches.

### Solution / Discussion

**Strategy 1 — Per-packet ACK (no end ACK):**

✅ **Pros:**
- **Fast loss recovery** — if a packet is lost, you know within ~1 RTT and can retransmit just that one
- **Bounded buffer at sender** — only need to remember unACKed packets (small window)
- Works well for **lossy links** (wireless, long-distance)

❌ **Cons:**
- ACK overhead — every packet generates a return ACK = doubles small-packet traffic
- No confirmation that the **whole file** arrived intact — the last packet's ACK could itself be lost and the sender thinks "all good" while receiver got nothing for the final piece
- More CPU work at both ends

**Strategy 2 — End-of-file ACK only:**

✅ **Pros:**
- **Minimal ACK overhead** — only one ACK for the whole transfer
- Confirms the entire file arrived
- Less CPU work during transfer

❌ **Cons:**
- **Catastrophic loss recovery** — if any packet is lost, you have to **resend the ENTIRE file** (the receiver can't tell sender which one was lost)
- **Huge buffer needed** at sender (must keep entire file until ACKed)
- Terrible for large files on lossy networks
- No early warning of trouble — you find out at the end

### Real-world verdict
Neither is great alone. **TCP combines the best of both** with cumulative ACKs + window-based sending. For huge files: per-packet ACKs are essential (otherwise one bit error = restart everything).

---

## Q3. Which OSI Layer Header Has the Destination Network Address?

### Solution

**Network Layer (Layer 3)** — its header contains the **logical (IP) addresses** of source and destination, including hosts on remote networks.

### Why?
- **Data Link (L2)** has MAC addresses → only useful within the local LAN; changes at every router hop
- **Network (L3)** has IP addresses → identifies hosts across networks, **stays end-to-end**
- **Transport (L4)** has port numbers → identifies the process, not the host

So when a packet needs to reach a host on **another network**, the destination IP in the **L3 (Network) header** is what routers use to figure out the path.

✅ **Answer: Network Layer (Layer 3)**

---

## Q4. OSI Encapsulation — Choose TWO Correct Steps

Which correctly describe the OSI encapsulation process?

(a) Transport layer divides data stream into segments, may add reliability/flow control info.
(b) Data Link layer adds physical source and destination addresses and an FCS to the segment.
(c) Packets are created when network layer encapsulates a frame with source and dest host addresses + protocol info.
(d) Packets are created when network layer adds Layer 3 addresses + control info to a segment.
(e) Presentation layer translates bits into voltages for transmission across the physical link.

### Solution — Analyze each option

| Option | Verdict | Why |
|---|---|---|
| **(a)** | ✅ **CORRECT** | Transport layer does segment data + adds reliability/flow control (TCP) |
| (b) | ❌ Wrong | Data Link wraps a **packet** (from L3), not a segment (from L4); also the unit becomes a **frame** at L2 |
| (c) | ❌ Wrong | Backwards! Network layer encapsulates a **segment** (from L4), not a frame |
| **(d)** | ✅ **CORRECT** | Network layer takes the segment, adds L3 addresses (IP) + control info → creates a **packet** |
| (e) | ❌ Wrong | The **Physical layer** does bit ↔ voltage conversion, not Presentation |

### The correct encapsulation flow

```
Application data
     │
     ▼
[Transport]  →  adds TCP/UDP header → SEGMENT       (a is correct here)
     │
     ▼
[Network]    →  adds IP header → PACKET             (d is correct here)
     │
     ▼
[Data Link]  →  adds MAC header + FCS trailer → FRAME
     │
     ▼
[Physical]   →  bits → voltages on wire
```

✅ **Answer: (a) and (d)**

---

## Q5. Which Layer Determines Resource Availability for Communication?

> Which layer is responsible for determining the availability of the receiving program and checking to see if enough resources exist for that communication?

### Solution

**Application Layer (Layer 7)**

### Why?
- Before two end-systems start a real conversation, the **application** has to confirm:
  - Is the receiving program running?
  - Does it have enough buffers/memory/connections to handle this?
  - Is it authorized?

This is **not** a job for lower layers — they just move data. Lower layers don't know what the apps need.

(Note: some textbooks split this between Application and Session layers — but the **Application layer** is the standard answer for resource availability checking.)

✅ **Answer: Application Layer**

---

## Q6. Which is a Physical Layer Vulnerability?

(a) MAC Address Spoofing
(b) Physical Theft of Data
(c) Route Spoofing
(d) Weak or non-existent authentication

### Solution

**(b) Physical Theft of Data**

### Why?
- The **Physical layer** deals with the actual cable, signals, hardware. Threats at this layer are **physical** in nature: cable taps, stealing hard drives, plugging in unauthorized devices.
- (a) MAC spoofing → **Data Link** layer (L2) attack
- (c) Route spoofing → **Network** layer (L3) attack
- (d) Weak auth → **Application/Session** layer (L7/L5) issue

✅ **Answer: (b) Physical Theft of Data**

---

## Q7. How Data Breaks Down at Each Layer (Top to Bottom)

> Show how data breaks down at each layer from top to bottom.

### Solution — The PDU (Protocol Data Unit) at each layer

```
┌────────────────────────────────────────────────────────────────┐
│ Layer 7  Application   →  DATA          (user message)         │
├────────────────────────────────────────────────────────────────┤
│ Layer 6  Presentation  →  DATA          (formatted/encrypted)  │
├────────────────────────────────────────────────────────────────┤
│ Layer 5  Session       →  DATA          (with session info)    │
├────────────────────────────────────────────────────────────────┤
│ Layer 4  Transport     →  SEGMENT       (adds TCP/UDP header,  │
│                                          ports, seq #)         │
├────────────────────────────────────────────────────────────────┤
│ Layer 3  Network       →  PACKET        (adds IP header,       │
│                                          source/dest IP)       │
├────────────────────────────────────────────────────────────────┤
│ Layer 2  Data Link     →  FRAME         (adds MAC header +     │
│                                          FCS trailer)          │
├────────────────────────────────────────────────────────────────┤
│ Layer 1  Physical      →  BITS          (electrical/optical    │
│                                          signals on wire)      │
└────────────────────────────────────────────────────────────────┘
```

### Visual encapsulation

```
At sender (going DOWN):

Original Data:                          [DATA]
After Transport (L4):              [TCP|DATA]
After Network (L3):           [IP|TCP|DATA]
After Data Link (L2):    [MAC|IP|TCP|DATA|FCS]
After Physical (L1):     010110100110... (bits on wire)
```

### Memorize the PDU names (top→bottom)
**D**ata → **D**ata → **D**ata → **S**egment → **P**acket → **F**rame → **B**its

✅ **Answer:** Data (L7,6,5) → Segment (L4) → Packet (L3) → Frame (L2) → Bits (L1)

---

## Q8. Ping the Default Gateway Fails — Which OSI Layer?

> Admin pings the default gateway at 10.10.10.1 and gets:
> ```
> Request timed out
> Request timed out
> Request timed out
> Request timed out
> Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
> ```
> Which OSI layer is the problem?

### Solution

**Layer 1 (Physical) and/or Layer 2 (Data Link)** — most likely a **physical** problem.

### Why?
- The default gateway is on your **same LAN** (it's the router on your local subnet)
- Reaching the gateway requires only:
  - Working physical connection (Layer 1)
  - Working Data Link layer (frames being sent/received)
  - Maybe Layer 3 (ARP to find router's MAC)
- 100% packet loss to your own router means:
  - **Cable unplugged or broken** (Layer 1)
  - **NIC failed** (Layer 1)
  - **Switch port down** (Layer 1/2)
  - **VLAN misconfiguration** (Layer 2)
  - **Wrong IP/subnet config** (Layer 3, but less likely)

### Troubleshooting order
1. Check cable / WiFi connection (L1)
2. Check NIC lights / link status (L1/L2)
3. Check IP configuration (L3)
4. Check switch/router (L1/L2/L3)

If you couldn't even reach an IP on your **own subnet**, the issue is local — not somewhere on the Internet.

✅ **Answer: Physical Layer (Layer 1)** — most likely a cable, NIC, or local connectivity problem (could also be Layer 2 / VLAN issue)

---

## Q9. Circuit-Switched File Transfer

> File = **640,000 bits**, all links = **1.536 Mb/s**, TDM with **24 slots/sec**, **500 ms** to set up the end-to-end circuit.
> How long to send the file from A to B?

### Concept refresher (Circuit Switching with TDM)

In TDM, the link is divided into N time slots per second. Your call gets one slot per frame. So your **effective bandwidth** is:
$$\text{Effective BW} = \frac{\text{Link BW}}{\text{Number of slots}}$$

### Solution

**Step 1:** Effective bandwidth (one TDM slot)
$$\text{BW per slot} = \frac{1.536 \text{ Mbps}}{24} = \frac{1{,}536{,}000}{24} = 64{,}000 \text{ bps} = 64 \text{ Kbps}$$

**Step 2:** Transmission time for the file
$$T_{trans} = \frac{640{,}000 \text{ bits}}{64{,}000 \text{ bps}} = 10 \text{ s}$$

**Step 3:** Add the circuit setup time
$$T_{total} = T_{setup} + T_{trans} = 0.5 + 10 = 10.5 \text{ s}$$

✅ **Answer: 10.5 seconds**

### 🔑 Key insight
In **circuit switching**, the transmission time is **independent of the number of links** because the circuit is dedicated end-to-end. Whether the path has 1 hop or 100 hops, transmission time is the same.

---

## Q10. Packet Switching with Store-and-Forward (TWO PARTS)

> Hosts A and B connected to switch S via **100-Mbps** links. Propagation delay on each link = **20 µs**. S is a **store-and-forward device**; it begins **retransmitting a received packet 35 µs after it has finished receiving it**. Calculate total time to transmit **10,000 bits** from A to B as:
>
> (a) A single packet
> (b) Two 5000-bit packets sent right after the other

### Concept refresher (Store-and-Forward)

Path: **A → Switch S → B** (2 links: A-to-S and S-to-B)

For a single packet of size L:
- Push out of A onto first link: $L/R$
- Wait for first bit to reach S: $T_{prop}$ (but bits arrive in pipeline as they're sent)
- S must **fully receive** before forwarding: needs another $L/R$ accumulating (but happens in parallel with sending)
- S adds **35 µs processing delay** before retransmitting
- Push out of S onto second link: $L/R$
- Wait for last bit to reach B: $T_{prop}$

### Setup numbers
- R = 100 Mbps = 10⁸ bps
- Propagation per link = $T_{prop}$ = 20 µs
- Switch processing delay = 35 µs

---

### Part (a) — Single 10,000-bit packet

**Step 1:** Transmission time of one packet on one link
$$T_{trans} = \frac{10{,}000 \text{ bits}}{10^8 \text{ bps}} = 10^{-4} \text{ s} = 100\,\mu s$$

**Step 2:** Trace the packet

```
Time 0:     A starts sending the packet
Time 100µs: A finishes sending (last bit on the wire to S)
Time 100+20 = 120µs: Last bit arrives at S (S has now fully received)
Time 120+35 = 155µs: S starts retransmitting (after 35 µs delay)
Time 155+100 = 255µs: S finishes sending (last bit on wire to B)
Time 255+20 = 275µs: Last bit arrives at B → DONE
```

**Step 3:** Total time
$$T_{total} = 2 \times T_{trans} + 2 \times T_{prop} + T_{switch}$$
$$= 2(100) + 2(20) + 35 = 200 + 40 + 35 = \mathbf{275\,\mu s}$$

✅ **Answer (a): 275 µs**

---

### Part (b) — Two 5000-bit packets sent back-to-back

**Step 1:** Transmission time per smaller packet
$$T_{trans} = \frac{5000}{10^8} = 50\,\mu s$$

**Step 2:** Trace both packets

```
Time 0:    A starts sending packet 1
Time 50µs: A finishes packet 1; immediately starts packet 2
Time 70µs: Packet 1's last bit arrives at S (50 + 20)
Time 100µs: A finishes packet 2 (50 + 50)
Time 105µs: Packet 1 ready to retransmit (70 + 35)

Time 105µs: S starts sending packet 1 to B
Time 120µs: Packet 2's last bit arrives at S (100 + 20)
Time 155µs: S finishes sending packet 1 (105 + 50)
Time 155µs: Packet 2 ready to retransmit (120 + 35)
Time 155µs: S immediately starts sending packet 2  ← lucky timing!
Time 175µs: Packet 1's last bit arrives at B (155 + 20)
Time 205µs: S finishes sending packet 2 (155 + 50)
Time 225µs: Packet 2's last bit arrives at B → DONE
```

**Step 3:** Total time
$$T_{total} = \mathbf{225\,\mu s}$$

✅ **Answer (b): 225 µs**

---

### 🔑 Key insight — Why Splitting Helps (Pipelining!)

| | Single packet | Two packets |
|---|---|---|
| Total time | 275 µs | 225 µs |
| Speedup | — | **50 µs faster** |

**Why?** With two packets, **packet 2 is being transmitted on link A→S while packet 1 is being processed/forwarded by S**. This is called **pipelining**.

```
Link A→S:  [PKT1 sending][PKT2 sending][idle      ]
Switch S:  [    receiving PKT1   ][rcv PKT2][35µs][send PKT1][send PKT2]
Link S→B:  [idle                            ][PKT1 sending][PKT2 sending]
                                              ↑
                                    overlap = speedup!
```

This is **the fundamental reason packet switching beats circuit switching for bursty data** — multiple packets can be in different parts of the network simultaneously.

### General rule
- **Bigger packets** → less pipelining benefit, but less per-packet overhead (headers)
- **Smaller packets** → more pipelining benefit, but more total header overhead
- **Sweet spot**: depends on link speed, distance, and number of hops (Ethernet uses 1500-byte max for a reason)

---

## 📝 Summary of Key Formulas

```
┌──────────────────────────────────────────────────────────────┐
│  HEADER OVERHEAD (Q1)                                         │
│  Fraction = nh / (M + nh)                                     │
├──────────────────────────────────────────────────────────────┤
│  TRANSMISSION DELAY                                           │
│  T_trans = L / R   (packet bits / bandwidth)                  │
│  Does NOT depend on distance                                  │
├──────────────────────────────────────────────────────────────┤
│  PROPAGATION DELAY                                            │
│  T_prop = d / s    (distance / signal speed)                  │
│  Does NOT depend on packet size or bandwidth                  │
├──────────────────────────────────────────────────────────────┤
│  CIRCUIT SWITCHED TOTAL                                       │
│  T = T_setup + L/R_effective                                  │
│  R_effective = R_link / N_TDM_slots                           │
├──────────────────────────────────────────────────────────────┤
│  STORE-AND-FORWARD (N links)                                  │
│  T = N·(L/R) + N·T_prop + (N-1)·T_switch                      │
│  Splitting into smaller packets enables pipelining            │
└──────────────────────────────────────────────────────────────┘
```

---

## 📋 OSI Layer Quick Reference (for Q3, Q4, Q5, Q6, Q7, Q8)

| Layer | PDU | Address | Key Job | Vulnerabilities |
|---|---|---|---|---|
| 7 Application | Data | URL/email | User services | Auth, app exploits |
| 6 Presentation | Data | — | Encrypt, format | Weak crypto |
| 5 Session | Data | — | Dialog, sync | Session hijack |
| 4 Transport | **Segment** | Port (16b) | Process-to-process | TCP attacks |
| 3 Network | **Packet** | IP (32b) | Routing | Route spoofing |
| 2 Data Link | **Frame** | MAC (48b) | Hop-to-hop | MAC spoofing |
| 1 Physical | **Bits** | — | Bit transmission | **Physical theft, taps** |

---

# 🔵 TUTORIAL 3 PROBLEMS BEGIN HERE

> Tutorial 3 builds on the same delay/switching concepts but adds: congestion control, multi-link path delays, cut-through switching, and Synchronous TDM framing.

---

## 📚 More Concepts You'll Need

### Concept E — Packet vs Circuit Switching for Steady-Rate Apps

- **Steady-rate, long-duration** apps (e.g., voice calls, video streams generating N×k bits every k seconds) want **predictable bandwidth**
- **Circuit switching** = perfect fit (reserved bandwidth, no jitter)
- **Packet switching** = better for bursty traffic (statistical multiplexing wins when sources are silent half the time)

### Concept F — Congestion Control Necessity

If the **sum of all source rates ≤ link capacity**, the network can carry it all. **No congestion control needed.** But if sources can burst above capacity, you need control to prevent buffer overflow / packet loss.

### Concept G — Store-and-Forward Across Multiple Links

For a packet of size L crossing N links with bandwidths R₁, R₂, ..., R_N and propagation delays p₁, p₂, ..., p_N:

$$T_{total} = \sum_{i=1}^{N}\left(\frac{L}{R_i} + p_i\right)$$

Every switch must fully receive before forwarding (store-and-forward).

### Concept H — Cut-Through Switching

Instead of waiting for the whole packet, the switch **starts forwarding after reading just the header** (e.g., the first 200 bits which include the destination address).

This is much faster but riskier (can't check FCS / drop bad frames before forwarding).

### Concept I — Synchronous TDM Framing

In Synchronous TDM, the multiplexer takes one chunk of bits from each input line in **strict round-robin** order, every frame. Sometimes a **framing bit** is added at the start of each frame.

```
Input lines:                Output (one frame):
A: aaa aaa aaa     →
B: bbb bbb bbb     →   [Frame bit] [chunk A][chunk B][chunk C]
C: ccc ccc ccc     →
```

If line A is silent, its slot **is still sent (as zeros or junk)** — that's the rigidity of synchronous TDM.

---

## Q11. Steady-Rate App: Packet vs Circuit Switching + Congestion Control

> An application transmits at a steady rate: every k time units it generates an N-bit unit of data (k small, fixed). When started, it runs for a long period.
>
> (a) Packet-switched OR circuit-switched? Why?
> (b) If packet-switched, and total source rates < link capacity, is congestion control needed?

### Part (a) — Which network type?

**Answer: Circuit-switched is more appropriate.**

**Reasoning:**
- The app generates data at a **constant, predictable rate** (N bits every k seconds)
- It runs for a **long period** → the cost of circuit setup (which is one-time) is amortized over a long session
- Circuit switching gives **guaranteed, dedicated bandwidth** = no jitter, no delay variation, no packet loss from congestion
- Packet switching would work but offers no advantage here (no bursty silence to multiplex over) and adds variable queuing delay

This is exactly why the **traditional telephone network** uses circuit switching — voice is a steady-rate, long-duration app.

### Part (b) — Is congestion control needed?

**Answer: No, congestion control is NOT needed in this scenario.**

**Reasoning:**
- The total demand (sum of all app rates) is **less than every link's capacity**
- Buffers will never overflow → no packet loss from congestion
- Congestion control adds complexity and feedback overhead, but here it would do nothing useful
- The network behaves predictably even without it

⚠️ **Caveat:** This only holds if the assumption stays true. If a new app joins or one app spikes, you'd need control to prevent collapse. Real networks have congestion control because they can't guarantee aggregate demand stays below capacity.

✅ **Answers:** (a) Circuit-switched (steady predictable rate, long duration); (b) No congestion control needed (aggregate rate ≤ capacity)

---

## Q12. Multi-Link Store-and-Forward Path

> Network path: 3 links, 2 store-and-forward switches. Each switch has 1.5 MB memory.
>
> Link speeds & propagation delays:
> ```
> Source ──10 Gbps,40ms──► [Switch 1.5MB] ──2 Gbps,5ms──► [Switch 1.5MB] ──2 Gbps,10ms──► Dest
> ```
>
> (a) Path empty initially — total time to transmit a 500 KB packet from first bit injected to last bit received?
> (b) Same path, but **someone just finished injecting another 500 KB packet on link 1**. Total time for YOUR 500 KB packet to be received?

### Setup

- Packet size = 500 KB = 500 × 1024 × 8 = **4,096,000 bits** (≈ 4 × 10⁶ for clean math, let me use exact)

Actually let me use 500 × 1000 × 8 = 4,000,000 bits for cleaner numbers (textbooks often use SI for "KB" in these problems).

I'll do **both** and note which:

**Exact (KiB):** L = 4,096,000 bits
**Decimal (kB):** L = 4,000,000 bits

I'll use **decimal (4,000,000 bits)** since most textbooks intend this for switching problems.

### Part (a) — Empty path

**Step 1:** Transmission time on each link (L = 4 × 10⁶ bits)

| Link | Bandwidth | T_trans = L/R |
|---|---|---|
| 1 (10 Gbps) | 10⁹ × 10 = 10¹⁰ bps | 4×10⁶ / 10¹⁰ = **0.4 ms** |
| 2 (2 Gbps)  | 2 × 10⁹ bps | 4×10⁶ / 2×10⁹ = **2 ms** |
| 3 (2 Gbps)  | 2 × 10⁹ bps | 4×10⁶ / 2×10⁹ = **2 ms** |

**Step 2:** Sum transmission delays + propagation delays

$$T_{total} = (T_{trans,1} + p_1) + (T_{trans,2} + p_2) + (T_{trans,3} + p_3)$$

$$= (0.4 + 40) + (2 + 5) + (2 + 10)$$

$$= 40.4 + 7 + 12 = \mathbf{59.4\,ms}$$

✅ **Answer (a): 59.4 ms**

### Part (b) — Another packet just finished injecting on link 1

If someone just finished injecting another 500 KB packet on link 1, that packet is **ahead of yours** in the pipeline. Your packet must wait for it to clear each switch's buffer (queue) before being processed.

This means **your packet experiences an extra queuing delay of one transmission time on each link** where it's stuck behind the other packet.

The other packet's full path delay = 59.4 ms (same calculation as part a)

But your packet only has to wait for the other packet to **clear each switch buffer**, not wait for it to fully reach the destination. Specifically:

- On link 1: another packet was *just finished injecting* (its last bit just left source), so it's already on the wire. Your packet starts being transmitted right after → no extra delay on link 1's transmission, but you're behind it
- At switch 1: the other packet must fully arrive + be transmitted on link 2 before your packet can be transmitted on link 2 → extra delay
- Same at switch 2

**Cleaner approach: just compute when YOUR packet's last bit arrives at the destination, accounting for the other packet ahead.**

Timeline (other packet starts at t = -0.4 ms so its "finished injecting on link 1" at t = 0):

```
Other packet:
  t = -0.4 ms: started injecting on link 1
  t =  0     : finished injecting on link 1
  t =  40 ms : last bit arrives at switch 1 (0 + 40 prop)
  t =  42 ms : finished transmitting on link 2 (40 + 2 trans)
  t =  47 ms : last bit arrives at switch 2 (42 + 5 prop)
  t =  49 ms : finished transmitting on link 3 (47 + 2 trans)
  t =  59 ms : last bit arrives at destination (49 + 10 prop)

Your packet:
  t =  0     : start injecting (link 1 just freed)
  t =  0.4 ms: finished injecting on link 1
  t =  40.4 ms: last bit arrives at switch 1
  
  But switch 1 is still busy sending the OTHER packet on link 2 until t = 42 ms
  So your packet must wait at switch 1 until t = 42 ms.
  
  t =  42 ms: switch 1 starts transmitting your packet on link 2
  t =  44 ms: finished transmitting on link 2
  t =  49 ms: last bit arrives at switch 2 (44 + 5 prop)
  
  But switch 2 is still sending the other packet until t = 49 ms.
  Your packet arrives exactly when other finishes → no wait!
  
  t =  49 ms: switch 2 starts transmitting your packet on link 3
  t =  51 ms: finished transmitting on link 3
  t =  61 ms: last bit arrives at destination (51 + 10 prop)
```

✅ **Answer (b): ≈ 61 ms** (only ~1.6 ms slower than empty path, because pipelining lets your packet "follow" the other one through the network)

### 🔑 Insight
Even with another packet ahead, **pipelining** keeps the extra delay small. The two packets end up moving in lockstep through the slower links.

---

## Q13. Latency for Different Switching Strategies

> 10 Mbps Ethernet, packet = 5000 bits, propagation per link = 10 µs, switch begins retransmitting immediately after fully receiving (store-and-forward).
>
> (a) Single store-and-forward switch in path
> (b) Same as (a), but **three switches**
> (c) Same as (b), but the switch uses **cut-through** — starts retransmitting after the first **200 bits** are received

### Setup

- R = 10 Mbps = 10⁷ bps
- L = 5000 bits → T_trans per link = 5000 / 10⁷ = **500 µs**
- Per-link prop = 10 µs
- For (c), switch can start retransmitting after just 200 bits received → switch delay = 200/10⁷ = **20 µs** (instead of 500 µs)

### Part (a) — One store-and-forward switch (2 links)

```
Source ──link 1──► [SW] ──link 2──► Dest
```

$$T_{total} = 2 \times T_{trans} + 2 \times T_{prop}$$
$$= 2(500) + 2(10) = 1000 + 20 = \mathbf{1020\,\mu s}$$

✅ **Answer (a): 1020 µs**

### Part (b) — Three store-and-forward switches (4 links)

```
Source ──L1──► [SW1] ──L2──► [SW2] ──L3──► [SW3] ──L4──► Dest
```

$$T_{total} = 4 \times T_{trans} + 4 \times T_{prop}$$
$$= 4(500) + 4(10) = 2000 + 40 = \mathbf{2040\,\mu s}$$

✅ **Answer (b): 2040 µs**

### Part (c) — Three cut-through switches (4 links)

In cut-through, each switch only delays by **the time to receive the first 200 bits** (not the whole packet).

- Source still transmits full packet: 500 µs on link 1
- Each switch delays by 20 µs (time to receive first 200 bits) before it can start forwarding
- Last bit must still propagate across each link

The packet is **streamed continuously** through all switches in cut-through mode. The total time is:

$$T_{total} = T_{trans,source} + (\text{sum of all prop delays}) + (\text{per-switch cut-through delay} \times \text{num switches})$$

$$= 500 + 4(10) + 3(20) = 500 + 40 + 60 = \mathbf{600\,\mu s}$$

✅ **Answer (c): 600 µs**

### 🔑 Key insight
| Configuration | Total | Speedup vs (b) |
|---|---|---|
| (a) 1 store-and-forward switch | 1020 µs | — |
| (b) 3 store-and-forward switches | 2040 µs | — (baseline) |
| (c) 3 cut-through switches | 600 µs | **3.4× faster** |

**Cut-through is dramatically faster** when packets are large and switches are many. Tradeoff: can't drop bad frames at switch level (they propagate to the destination).

---

## Q14. Synchronous TDM — Output Bit Sequence

> Multiplexer for Synchronous TDM. Frame = **3 time slots**, each slot holds **3 bits**. Each frame starts with a **framing bit alternating 0/1**.
>
> Inputs (300 kbps each):
> - Line A: `101110111101`
> - Line B: `111111100000`
> - Line C: `101000001111`
>
> What's the bit sequence on the outgoing link?

### Concept refresher
Each frame: `[framing bit][3 bits from A][3 bits from B][3 bits from C]`

Frames alternate framing bits: 0, 1, 0, 1, ...

Each input has 12 bits → split into 3-bit chunks → 4 chunks per line → **4 frames total**.

### Step 1 — Split each input into 3-bit chunks

| Line | Chunk 1 | Chunk 2 | Chunk 3 | Chunk 4 |
|---|---|---|---|---|
| A | 101 | 110 | 111 | 101 |
| B | 111 | 111 | 100 | 000 |
| C | 101 | 000 | 001 | 111 |

### Step 2 — Build each frame

Format: **[fbit][A chunk][B chunk][C chunk]**

| Frame | Framing bit | A | B | C | Frame output |
|---|---|---|---|---|---|
| 1 | 0 | 101 | 111 | 101 | `0 101 111 101` |
| 2 | 1 | 110 | 111 | 000 | `1 110 111 000` |
| 3 | 0 | 111 | 100 | 001 | `0 111 100 001` |
| 4 | 1 | 101 | 000 | 111 | `1 101 000 111` |

### Step 3 — Concatenate

```
Frame 1: 0 101 111 101
Frame 2: 1 110 111 000
Frame 3: 0 111 100 001
Frame 4: 1 101 000 111

Outgoing link bit sequence:
0101111101 1110111000 0111100001 1101000111
```

✅ **Final answer:**
```
0101111101  1110111000  0111100001  1101000111
```

(Spaces added for readability — actual output is one continuous bit stream of 40 bits.)

### 🔑 Key insight
- Each frame = 1 + 3·n bits (1 framing bit + n input lines × bits per slot)
- Frames repeat at fixed rate regardless of whether lines have data
- If line C had nothing to send, its 3-slot would still be transmitted (as zeros or junk) → bandwidth wasted
- **Statistical TDM** fixes this by only sending slots that have data, but adds addressing overhead per chunk

---

## 📊 Combined Summary of All Tutorial 2 + 3 Problems

| Q | Topic | Key formula / concept |
|---|---|---|
| 1 | Header overhead | nh / (M + nh) |
| 2 | ACK strategies | Per-packet vs end-of-file tradeoff |
| 3 | OSI layers | Network layer holds destination IP |
| 4 | Encapsulation | Segment → Packet → Frame order |
| 5 | OSI layers | Application layer = resource check |
| 6 | OSI security | Physical layer = physical theft |
| 7 | PDU naming | Data → Segment → Packet → Frame → Bits |
| 8 | Troubleshooting | Default gateway fail = Layer 1 |
| 9 | Circuit switch + TDM | Effective BW = Link BW / N slots |
| 10 | Store-and-forward | Pipelining saves time when split |
| 11 | Steady-rate apps | Circuit switching wins; no congestion control if rate < cap |
| 12 | Multi-link path | T = Σ(L/Rᵢ + pᵢ) |
| 13 | Cut-through switching | Switch delay = (header bits)/R, not L/R |
| 14 | Synchronous TDM | Frame = framing bit + slots in round-robin |

---

If anything is unclear (especially the pipelining in Q10/Q12 or the cut-through math in Q13), ask me to redraw it with a different example.
