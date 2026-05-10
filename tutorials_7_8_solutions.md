# Tutorials 7 & 8 — Wireless (802.11/MACA), Switches & ARP

> **Note on Tutorial 8:** Tutorial 8's questions (switch self-learning with hosts A–H, broadcast/collision domain counts, VLAN-tagged ARP across two switches, and three-subnet ARP from Kurose Fig 5.33) are the same set as Q6–Q11 below. Both tutorials are solved together in this file.
>
> **Topics:** 802.11 DCF timeline, NAV, RTS/CTS, hidden/exposed terminals, switch self-learning, broadcast/collision domains, VLANs, ARP across LANs
> **Reference guides:**
> - [multiple_access_protocols_guide.md](multiple_access_protocols_guide.md) (CSMA/CA, MACA, hidden terminal)
> - [mac_arp_ethernet_switches_vlans_guide.md](mac_arp_ethernet_switches_vlans_guide.md) (switches, ARP, domains, VLANs)

---

## 📚 Concepts You Need First

### Concept A — 802.11 DCF Frame Exchange Timeline

A complete data frame exchange in 802.11 DCF with RTS/CTS:

```
DIFS → Backoff → RTS → SIFS → CTS → SIFS → DATA → SIFS → ACK
```

- **DIFS** (Distributed IFS): wait time before initiating new transmission
- **Backoff**: random number of slots after DIFS to avoid simultaneous transmissions
- **SIFS** (Short IFS): smaller gap used between RTS/CTS/DATA/ACK so nobody else grabs the channel mid-exchange
- All frames carry a **Duration field** → other stations set their **NAV** (do-not-transmit timer)

### Concept B — MAC Layer Throughput Formula

$$\text{MAC throughput} = \frac{\text{Useful data bits per exchange}}{\text{Total time per exchange}}$$

Useful = MAC payload bits. Total time = sum of all timing components in the exchange.

### Concept C — NAV (Network Allocation Vector)
A timer each station maintains. When a station hears RTS or CTS, it reads the Duration field and sets NAV to "stay quiet for this long."
- **Hear CTS** → you're near the receiver → must stay quiet
- **Hear RTS only (no CTS)** → you're near sender but not receiver → safe to transmit

### Concept D — Switch Self-Learning Algorithm
1. Frame arrives on port P with source MAC = X → switch records `(X, P)`
2. Look up destination MAC D:
   - **Found** → forward only to that port (selective forwarding)
   - **Not found** → flood to all ports except incoming

### Concept E — Broadcast vs Collision Domains
| Device | Breaks collision domain? | Breaks broadcast domain? |
|---|---|---|
| Hub | NO | NO |
| Switch | **YES** (each port = own collision domain) | NO |
| Router | YES | **YES** |

### Concept F — Cross-LAN ARP & IP Routing
- IP address: **stays end-to-end**
- MAC address: **changes hop-by-hop** at each router
- ARP only resolves IPs **on the same LAN** — to reach a remote IP, you ARP the **default gateway's IP**

---

# 🧮 PROBLEMS

---

## Q1. 802.11 DCF MAC Throughput

> Wireless LAN parameters:
> - Physical data rate = **58 Mbps**
> - MAC payload = **1562 bytes**
> - MAC header = **22 bytes**
> - ACK frame = **18 bytes**
> - RTS = **14 bytes**
> - CTS = **10 bytes**
> - DIFS = **26 µs**
> - SIFS = **12 µs**
>
> Two stations exchange one data frame using DCF + RTS/CTS.
> (a) Draw timeline diagram
> (b) Calculate total time
> (c) Calculate MAC throughput

### Part (a) — Timeline Diagram

```
Sender:  [DIFS][Backoff][RTS]──[SIFS]──────────[DATA]──────────────────[SIFS]
                                                                            
Receiver:                  [SIFS][CTS]──────────[SIFS][ACK]
                                                            
Other:                                  [── NAV (stay quiet) ──]

Time →
```

Full sequence: **DIFS → RTS → SIFS → CTS → SIFS → DATA → SIFS → ACK**

(Backoff is variable — usually ignored unless given. Assume backoff = 0 here for the calculation.)

### Part (b) — Total Time Calculation

**Step 1:** Compute transmission time of each frame at 58 Mbps.

| Frame | Bits | T = bits / 58 Mbps |
|---|---|---|
| RTS (14 B) | 112 | 112 / 58×10⁶ = **1.93 µs** |
| CTS (10 B) | 80 | 80 / 58×10⁶ = **1.38 µs** |
| DATA (header 22 B + payload 1562 B = 1584 B) | 12,672 | 12,672 / 58×10⁶ = **218.48 µs** |
| ACK (18 B) | 144 | 144 / 58×10⁶ = **2.48 µs** |

**Step 2:** Sum all components

$$T_{total} = \text{DIFS} + T_{RTS} + \text{SIFS} + T_{CTS} + \text{SIFS} + T_{DATA} + \text{SIFS} + T_{ACK}$$

$$= 26 + 1.93 + 12 + 1.38 + 12 + 218.48 + 12 + 2.48$$

$$= \mathbf{286.27\,\mu s}$$

✅ **Answer (b): ≈ 286.27 µs**

### Part (c) — MAC Throughput

Useful bits delivered = MAC payload = 1562 × 8 = **12,496 bits**

$$\text{Throughput} = \frac{12{,}496 \text{ bits}}{286.27 \times 10^{-6} \text{ s}} = 43.65 \times 10^6 \text{ bps} \approx \mathbf{43.65 \text{ Mbps}}$$

✅ **Answer (c): ≈ 43.65 Mbps**

### 🔑 Insight
Even at 58 Mbps physical rate, MAC throughput is only **~75%** because of:
- Per-frame overhead (RTS, CTS, ACK)
- Inter-frame spaces (DIFS, 3× SIFS)
- MAC headers

This is why "advertised speed" of WiFi is always more than what you actually get.

> **DIFS (Distributed Inter-Frame Space):** the longer wait time a station observes before *starting* a new transmission — a low-priority gap.
> **SIFS (Short Inter-Frame Space):** the shorter gap used *between* tightly-coupled frames in one exchange (RTS↔CTS, DATA↔ACK). Because SIFS < DIFS, an ACK always wins the channel before any new frame can grab it.
> **MAC payload:** the user data carried inside one MAC frame, *excluding* MAC headers/trailers.
> **Physical data rate:** the raw bit rate of the radio link before any MAC overhead is subtracted.

---

## Q2. Hidden Terminal & NAV — Which Station is Closer to A?

> Diagram (re-described):
> - A: sends RTS → DATA
> - B: sends CTS → ACK (receiver of A's data)
> - C: hears CTS, sets NAV
> - D: hears... NAV from somewhere

> Of stations C and D, which one is closer to A? Why?

### Solution

**C is closer to A.**

### Reasoning

Look at the timing pattern:
- C's NAV is set **earlier** in the timeline (it spans the whole exchange after CTS)
- D's NAV is set **later** (spans only after the DATA frame transmission begins)

This means:
- **C heard the CTS from B** → C is in B's range. But also C might be in A's range (depending on the diagram)
- **D heard the DATA from A** → D is in A's range, but D didn't hear B's CTS (D is too far from B)

But the question is "closer to A":
- D heard A's DATA → D is in A's range
- C heard B's CTS but maybe not A's RTS

Looking again at the actual diagram positioning: **D's NAV starts when DATA begins** = D heard DATA from A directly. **C's NAV starts when CTS begins** = C heard CTS from B, not from A.

So:
- **D is closer to A** (heard A's transmissions directly)
- **C is closer to B** (heard B's CTS, possibly not A)

✅ **Answer: D is closer to A** — because D's NAV is set after hearing A's DATA frame, meaning D is within A's transmission range. C is closer to B because it set its NAV after hearing B's CTS.

### 🔑 Key insight
RTS/CTS is brilliant because it lets stations infer their relative position based on which control frame they hear first. C is "hidden from A" → can't hear A's RTS → relies on B's CTS to know when to stay quiet.

> **RTS (Request To Send) / CTS (Clear To Send):** tiny control frames exchanged before data so that nodes near the *receiver* (which may not be able to hear the sender) learn to stay silent.
> **NAV (Network Allocation Vector):** a virtual carrier-sense timer each station maintains. When a station hears RTS or CTS, it reads the Duration field and waits that long before attempting to transmit, even without sensing the medium.
> **Hidden terminal:** a node whose transmissions can't be heard by some sender but *do* collide at the sender's intended receiver.

---

## Q3. Hidden Terminal Problem in 4-node Topology

> Wireless laptops A, B, C, D in a line. B is in range of A & C; C in range of B & D. A & D are NOT in each other's range. Using RTS/CTS-based MAC.
>
> (a) If C sends B an RTS, why does A know NOT to transmit?
> (b) If B is sending data to C, why does D know NOT to transmit?

### Part (a) — Why A defers when C sends RTS to B

```
Range diagram:
   ●───●───●───●
   A   B   C   D
   ◄───►   ◄───►   (A↔B and C↔D in range)
       ◄───►       (B↔C in range)
```

**Sequence:**
1. C → RTS → B (C wants to send to B)
2. A is in B's range, so A hears... wait, A wouldn't hear C's RTS directly (A and C aren't in range).

But A WILL hear **B's CTS** when B replies "OK, go ahead":
1. C → RTS → B
2. B → CTS → A (CTS is broadcast in B's range, including A)
3. A hears the CTS → reads the Duration field → sets NAV → stays silent

✅ **Answer (a):** A defers because A hears the **CTS from B** (in response to C's RTS). The CTS reaches everyone in B's range, including A. The Duration field tells A how long to stay quiet.

### Part (b) — Why D defers when B sends data to C

**Sequence:**
1. B → RTS → C
2. C → CTS → B (CTS is broadcast in C's range, **including D**)
3. D hears C's CTS → reads Duration → sets NAV → stays silent during B's data transmission

✅ **Answer (b):** D defers because D hears **C's CTS** (in C's range). The CTS warns D about the upcoming data transmission's duration, even though D can't hear B itself.

### 🔑 Insight
RTS/CTS solves the hidden terminal problem because **the CTS is broadcast in the receiver's range** — it warns everyone near the receiver, even if they can't hear the sender.

---

## Q4. Wireless Concurrent Transmission Possibilities (5-node)

> 5 stations: A, B, C, D, E. Connectivity:
> - **A** can talk to all (B, C, D, E)
> - **B** can talk to A, C, E
> - **C** can talk to A, B, D, E
> - **D** can talk to A, C, E
> - **E** can talk to A, C, D, B
>
> (a) When A is sending to B, what other communications are possible?
> (b) When B is sending to A, what other communications are possible?
> (c) When B is sending to C, what other communications are possible?

### Approach
For another transmission X→Y to happen concurrently:
- **X must not be in current receiver's range** (else X must defer)
- **Y must not be in current sender's range** (else Y can't receive)

### Part (a) — A sends to B

A is sending → A's signal reaches all others (B, C, D, E).
B is receiving → anything also reaching B will collide.

For another pair X→Y:
- X must not be in A's range (impossible — A reaches everyone)
- AND Y must not be in B's range (B's range = A, C, E)

So Y can be **D only** (D is not in B's range).
But X can't be A (A is busy), and X can't be in A's range — impossible.

Hmm, since A reaches everyone, NO other station can transmit (they'd all hear A and defer).

✅ **Answer (a): No other communications possible.** A's transmission reaches all stations, so all defer.

### Part (b) — B sends to A

B is sending → reaches A, C, E (B's range).
A is receiving → can't have another sender reaching A. A's range = everyone, so any sender to A would interfere.

For another pair X→Y:
- X must not be in A's range → impossible (A reaches all)
- OR more precisely, no two transmissions can both have A in range

But D is NOT in B's range. So D could potentially transmit to someone:
- D → C? C is in B's range → C is hearing B → C would get a collision. **No.**
- D → E? E is in B's range → E hears B → collision at E. **No.**
- D → A? A is busy receiving from B. **No.**

✅ **Answer (b): No other communications possible.** Same reason — A is the receiver and A reaches everyone, so any other transmission would interfere with A's reception.

### Part (c) — B sends to C

B → C: B's signal reaches A, C, E. C's signal range = A, B, D, E.

For X → Y concurrently:
- X not in C's range → X can be **only D** (D is not in C's range? Actually D IS in C's range per the spec — let me re-check)

Re-reading: "C can talk to A, B, D, E" → so D **IS** in C's range. Hmm.

Let me reconsider — for X → Y:
- X must not interfere with C (so X can't be in C's range OR X's signal doesn't reach C)
- Y must not be in B's range (else B's transmission collides at Y)

B's range = A, C, E. So Y can be: D (the only one not in B's range).

X must not be in C's range. C's range = A, B, D, E. So X must NOT be A, B, D, or E. Only candidate is... none of the original stations qualify.

So even D → ? doesn't work because D is in C's range.

✅ **Answer (c): No other communications possible** in this dense topology.

### 🔑 Insight
In wireless, dense connectivity = limited concurrency. RTS/CTS doesn't help when every station hears every other.

---

## Q5. MACA — Concurrent Transmissions in 6-Node Line

> 6 stations A through F in a straight line, communicating via MACA. Is it possible for two transmissions to happen simultaneously?

### Setup (assuming each station only reaches its immediate neighbors)
```
A ─── B ─── C ─── D ─── E ─── F
```

If radio range = 1 hop (only adjacent), then:
- **A → B** uses A's transmission and B's reception
- For a concurrent transmission, we need another sender-receiver pair such that:
  - The sender doesn't interfere with B's reception
  - B's transmission (the CTS) doesn't interfere with the other receiver

**Try A→B and E→F simultaneously:**
- A sends RTS to B → B sends CTS (heard by A and C only)
- E sends RTS to F → F sends CTS (heard by E and... no other in this 1-hop model)
- C hears B's CTS → sets NAV → stays silent
- D hears nothing → could potentially transmit

But E's CTS isn't heard by C or D (out of range). So:
- A→B exchange: C is silenced
- E→F exchange: D would be aware only if D heard E or F's transmission. D doesn't hear E or F (assuming 1-hop). But D might want to transmit to C or E — D hearing nothing might transmit and collide.

But the question is just **"is it possible for two transmissions to take place simultaneously?"**

✅ **Answer: YES, two transmissions can happen simultaneously.**

**Example:** **A → B** and **E → F** can happen at the same time.

### Reasoning
- A's signal reaches only B (and possibly C if range = 2 hops)
- E's signal reaches only F (and possibly D)
- A and E are far enough apart that their signals don't collide at either receiver (B or F)
- B's CTS reaches A, C; F's CTS reaches E, D
- Stations C and D are warned by NAV → defer
- A↔B and E↔F exchanges run concurrently without interference ✓

### 🔑 Insight
**MACA enables spatial reuse** — the same channel can carry multiple transmissions in different parts of the network simultaneously, as long as senders/receivers are far enough apart. This is a huge advantage over pure CSMA.

---

## Q6. Switch Self-Learning — 5 Packet Trace

> Topology (described):
> - **Switch 1** connects: Host A (port 2), Host C (port 3), Host D (port 0), and Switch 2 (port 1)
> - **Switch 2** connects: Switch 1 (port 2), Host E (port 3), Switch 3 (port 0), Host F (port 1)
> - **Switch 3** connects: Switch 2 (port 0), Host G (port 1), Host H (port 2), Host B (port 3)
>
> All switching tables initially empty. Trace tables for Switch 1, Switch 2, Switch 3 after each packet:
>
> (a) D → E
> (b) H → E
> (c) F → C
> (d) H → F
> (e) G → E

### Self-Learning Rules
- On receiving frame from port P with source MAC X: record `(X, P)` in table
- Look up destination MAC: if found, forward to that port; if not, flood (all ports except incoming)

### Initial State (empty tables)
| Switch 1 | Switch 2 | Switch 3 |
|---|---|---|
| (empty) | (empty) | (empty) |

---

### Step (a) — Host D sends to Host E

**Path:** D (Switch1, port 0) → ? → E (Switch2, port 3)

**Switch 1:**
- Frame arrives port 0 with source D → learn `(D, 0)`
- Lookup E → not found → flood to ports 1, 2, 3

**Switch 2:**
- Frame arrives on port 2 (from Switch 1) with source D → learn `(D, 2)`
- Lookup E → not found → flood to ports 0, 1, 3

**Switch 3:**
- Frame arrives on port 0 (from Switch 2) with source D → learn `(D, 0)`
- Lookup E → not found → flood to ports 1, 2, 3

**Tables after (a):**

| Switch 1 | Switch 2 | Switch 3 |
|---|---|---|
| D → 0 | D → 2 | D → 0 |

---

### Step (b) — Host H sends to Host E

**Path:** H (Switch3, port 2) → Switch2 → E (Switch2, port 3)

**Switch 3:**
- Frame on port 2, source H → learn `(H, 2)`
- Lookup E → not found → flood to ports 0, 1, 3

**Switch 2:**
- Frame on port 0 (from Switch 3), source H → learn `(H, 0)`
- Lookup E → not found → flood to ports 1, 2, 3

**Switch 1:**
- Frame on port 1 (from Switch 2), source H → learn `(H, 1)`
- Lookup E → not found → flood to ports 0, 2, 3

**Tables after (b):**

| Switch 1 | Switch 2 | Switch 3 |
|---|---|---|
| D → 0, H → 1 | D → 2, H → 0 | D → 0, H → 2 |

---

### Step (c) — Host F sends to Host C

**Path:** F (Switch2, port 1) → Switch1 → C (Switch1, port 3)

**Switch 2:**
- Frame on port 1, source F → learn `(F, 1)`
- Lookup C → not found → flood to ports 0, 2, 3

**Switch 1:**
- Frame on port 1 (from Switch 2), source F → learn `(F, 1)`
- Lookup C → not found → flood to ports 0, 2, 3

**Switch 3:**
- Frame on port 0 (from Switch 2), source F → learn `(F, 0)`
- Lookup C → not found → flood to ports 1, 2, 3

**Tables after (c):**

| Switch 1 | Switch 2 | Switch 3 |
|---|---|---|
| D → 0, H → 1, F → 1 | D → 2, H → 0, F → 1 | D → 0, H → 2, F → 0 |

---

### Step (d) — Host H sends to Host F

**Path:** H (Switch3, port 2) → Switch2 → F (Switch2, port 1)

**Switch 3:**
- Frame on port 2, source H → already known (H, 2) ✓ no change
- Lookup F → found on port 0 → forward only to port 0 (selective forwarding! No flood)

**Switch 2:**
- Frame on port 0 (from Switch 3), source H → already known (H, 0) ✓ no change
- Lookup F → found on port 1 → forward only to port 1 (no flood, no Switch 1 traffic)

**Switch 1:** ❌ Doesn't see this frame!

**Tables after (d):**

| Switch 1 | Switch 2 | Switch 3 |
|---|---|---|
| D → 0, H → 1, F → 1 | D → 2, H → 0, F → 1 | D → 0, H → 2, F → 0 |

(No changes — frames went through Switch3 → Switch2 → F directly because tables already had needed info.)

---

### Step (e) — Host G sends to Host E

**Path:** G (Switch3, port 1) → Switch2 → E (Switch2, port 3)

**Switch 3:**
- Frame on port 1, source G → learn `(G, 1)`
- Lookup E → not found → flood to ports 0, 2, 3

**Switch 2:**
- Frame on port 0 (from Switch 3), source G → learn `(G, 0)`
- Lookup E → not found → flood to ports 1, 2, 3

**Switch 1:**
- Frame on port 1 (from Switch 2), source G → learn `(G, 1)`
- Lookup E → not found → flood to ports 0, 2, 3

**Final tables after (e):**

| Switch 1 | Switch 2 | Switch 3 |
|---|---|---|
| D → 0 | D → 2 | D → 0 |
| H → 1 | H → 0 | H → 2 |
| F → 1 | F → 1 | F → 0 |
| G → 1 | G → 0 | G → 1 |

---

## Q7. Domains in Switch + Hub Topology (2 switches, 6 hubs)

> Topology: 2 switches connected to each other, each switch connects to 3 hubs, each hub has multiple PCs.

### Counting

**Collision domains:**
- Each **hub** = one collision domain (all PCs on a hub share collisions)
- 6 hubs × 1 = **6 collision domains**
- Plus the link between the two switches = 1 collision domain
- = **7 collision domains** (but if you count only the host-side ones, it's 6)

**Broadcast domains:**
- Switches do NOT break broadcast domains
- All PCs share one broadcast network
- = **1 broadcast domain**

✅ **Answer: 1 broadcast domain, 6 (or 7) collision domains**

---

## Q8. Domains in Router + Switch Topology (Sales / Production)

> Topology: 1 router (e0, e1) with 2 switches — Sales (port e0) and Production (port e1).

### Counting

**Broadcast domains:** Router separates them → **2 broadcast domains** (Sales and Production)

**Collision domains:** Each switch port = its own collision domain.
- Sales switch: count of ports in use = ~5 (4 PCs + 1 uplink to router)
- Production switch: same ~5
- Add the e0 and e1 router-to-switch links

Roughly: **8–10 collision domains** (depends on exact PC count). If each switch has 4 PCs:
- Sales: 4 PCs × 1 collision domain each + 1 (uplink) = 5
- Production: same = 5
- **= 10 collision domains**

✅ **Answer: 2 broadcast domains, ≈10 collision domains** (verify with exact PC count from your slide diagram)

---

## Q9. Domains with VLANs

> 1 router + 2 switches connecting Human Resources and Management departments. Default VLAN only.

### Counting (default VLAN = 1 broadcast domain per switch network)

Without VLAN separation, both switches are in the same broadcast domain unless connected through router.

- If switches are connected through router → **2 broadcast domains** (HR, Management)
- If switches are connected directly → **1 broadcast domain**

For collision domains: each switch port = 1 collision domain.

**Typical answer: 2 broadcast domains** (because router e0/e1 separates them), and **collision domains = number of active switch ports**.

✅ **Answer: 2 broadcast domains, collision domains = sum of all switch ports in use**

---

## Q10. ARP Across 2 VLAN Switches

> Computer at port 8 of Switch1 wants to send ARP request to computer at port 2 of Switch2 — but doesn't have its MAC.
>
> (a) Draw ARP request packet at output of port 8 (Switch1), port 16 of Switch1, and port 3 of Switch2
> (b) Draw ARP response at port 3 of Switch2 and port 1 of Switch2

### Setup
- VLAN switching uses **802.1Q tagging** between switches
- Computer at Switch1 port 8 = "Mac1Switch1"
- Computer at Switch2 port 2 = "Mac2Switch2"
- Ports 2, 3, 5 on Switch2 belong to **EE VLAN**
- Trunk port between switches carries tagged frames

### Part (a) — ARP Request Packets

**At output of port 8 (Switch1) — leaving the source PC:**
```
┌──────────────┬──────────────┬──────┐
│ Dst: FF:FF...│ Src: Mac1Sw1 │ ARP  │  (broadcast, untagged)
└──────────────┴──────────────┴──────┘
```

**At port 16 of Switch1 (trunk port to Switch2) — VLAN tag added:**
```
┌──────────────┬──────────────┬──────┬──────┐
│ Dst: FF:FF...│ Src: Mac1Sw1 │ TAG  │ ARP  │  (broadcast + 802.1Q tag for EE VLAN)
└──────────────┴──────────────┴──────┴──────┘
```

**At port 3 of Switch2 — VLAN tag stripped before forwarding to host:**
```
┌──────────────┬──────────────┬──────┐
│ Dst: FF:FF...│ Src: Mac1Sw1 │ ARP  │  (untagged again, broadcast on EE VLAN ports)
└──────────────┴──────────────┴──────┘
```

### Part (b) — ARP Response Packets

The destination computer (Mac2Switch2) replies with **unicast** ARP.

**At port 3 of Switch2 — entering switch:**
```
┌──────────────┬──────────────┬──────┐
│ Dst: Mac1Sw1 │ Src: Mac2Sw2 │ ARP  │  (untagged, unicast)
└──────────────┴──────────────┴──────┘
```

**At port 1 of Switch2 (trunk to Switch1) — VLAN tag added:**
```
┌──────────────┬──────────────┬──────┬──────┐
│ Dst: Mac1Sw1 │ Src: Mac2Sw2 │ TAG  │ ARP  │
└──────────────┴──────────────┴──────┴──────┘
```

✅ **Key:** ARP request is broadcast; ARP reply is unicast. VLAN tag is added on trunk links between switches and stripped before delivering to hosts.

> **VLAN (Virtual LAN):** a software-defined broadcast domain — one physical switch can host several VLANs that behave like separate switches.
> **Trunk port:** a switch port configured to carry frames from multiple VLANs at once. Frames on a trunk are tagged so the receiving switch knows which VLAN each frame belongs to.
> **Access port:** a switch port belonging to exactly one VLAN. Frames here are *untagged* (the host doesn't know about VLANs).
> **802.1Q tag:** the 4-byte field inserted between the source-MAC and EtherType fields that carries the VLAN ID (12 bits → up to 4096 VLANs) and a 3-bit priority.

---

## Q11. Three Subnets, Two Routers — Full ARP Trace (Kurose Fig 5.33)

> Three LANs (Subnet 1, 2, 3) interconnected by 2 routers.
> - Subnet 1 hosts: A, B (192.168.1.xxx)
> - Subnet 2: D + router interfaces (192.168.2.xxx)
> - Subnet 3: E, F (192.168.3.xxx)
> - C is also somewhere connected via router

### Part (a) — Assign IP Addresses

Each interface needs an IP in its subnet:

| Device | Interface | IP Address |
|---|---|---|
| Host A | NIC | 192.168.1.10 |
| Host B | NIC | 192.168.1.11 |
| Router 1 | Subnet 1 side | 192.168.1.1 (default gateway for Subnet 1) |
| Router 1 | Subnet 2 side | 192.168.2.1 |
| Host D | NIC | 192.168.2.10 |
| Router 2 | Subnet 2 side | 192.168.2.2 |
| Router 2 | Subnet 3 side | 192.168.3.1 (default gateway for Subnet 3) |
| Host E | NIC | 192.168.3.10 |
| Host F | NIC | 192.168.3.11 |
| Host C | NIC | (whichever subnet — say 192.168.3.12 if on Subnet 3) |

### Part (b) — Assign MAC Addresses

Each adapter has a unique MAC:

| Device | MAC |
|---|---|
| Host A | AA:AA:AA:AA:AA:AA |
| Host B | BB:BB:BB:BB:BB:BB |
| Router1 (Subnet 1 side) | R1A:R1A:R1A:R1A:R1A:R1A |
| Router1 (Subnet 2 side) | R1B:R1B:R1B:R1B:R1B:R1B |
| Host D | DD:DD:DD:DD:DD:DD |
| Router2 (Subnet 2 side) | R2A:R2A:R2A:R2A:R2A:R2A |
| Router2 (Subnet 3 side) | R2B:R2B:R2B:R2B:R2B:R2B |
| Host E | EE:EE:EE:EE:EE:EE |
| Host F | FF:FF:FF:FF:FF:FF |

### Part (c) — IP Datagram from E to B (ARP tables empty)

E (Subnet 3) wants to send to B (Subnet 1) — different subnets, so packet goes through both routers.

**Step 1:** E checks ARP table for default gateway (Router2's Subnet-3 interface, IP 192.168.3.1). Empty → must ARP.

**Step 2:** E broadcasts **ARP request**: "Who has 192.168.3.1?"
- Frame: dst = FF:FF:FF:FF:FF:FF, src = EE:EE:EE...
- Reaches everyone on Subnet 3 (E, F, C, Router2)

**Step 3:** Router2's Subnet-3 interface replies: ARP reply (unicast)
- Frame: dst = EE:EE..., src = R2B:R2B...
- Says "192.168.3.1 → R2B"

**Step 4:** E now caches (192.168.3.1 → R2B). Builds frame:
- IP src = E's IP (192.168.3.10), IP dst = B's IP (192.168.1.11)
- Frame: dst MAC = R2B, src MAC = EE
- Sends out

**Step 5:** Router2 receives, looks at IP dst (192.168.1.11). Routing table says: send to Router1 via Subnet 2. Router2 needs Router1's MAC on Subnet 2.

If Router2's ARP table has Router1's MAC → forward immediately. If not → ARP for it.

**Step 6:** Router2 ARPs for 192.168.2.1 (Router1's Subnet-2 IP) on Subnet 2. Router1 replies. Router2 builds new frame:
- IP src = E (unchanged), IP dst = B (unchanged)
- Frame: dst MAC = R1B, src MAC = R2A
- Sends out on Subnet 2

**Step 7:** Router1 receives, looks at IP dst (192.168.1.11). Routing says: send to Subnet 1.

**Step 8:** Router1 ARPs for B's MAC (192.168.1.11) on Subnet 1 (if not cached). B replies.

**Step 9:** Router1 builds final frame:
- IP src = E (unchanged), IP dst = B (unchanged)
- Frame: dst MAC = BB, src MAC = R1A
- Sends to B on Subnet 1

**Step 10:** B receives the frame. Done!

### Part (d) — Same as (c) but ARP table empty in sending host only (others up to date)

The trace is **almost identical** but only Step 2-3 is needed (E's ARP for Router2).

Routers 1 and 2 already have their ARP tables filled, so:
- Step 6 skipped (Router2 already knows Router1's MAC)
- Step 8 skipped (Router1 already knows B's MAC)

So total: 1 ARP exchange (E to Router2), then the data frame is forwarded all the way through.

✅ **Key takeaways:**
- **IP src/dst stay the same** end-to-end (E → B)
- **MAC src/dst change at every router hop**
- ARP is needed at every hop where the destination MAC isn't cached
- ARP only resolves IPs **on the same subnet**

> **ARP cache / ARP table:** a small per-host table mapping known IP addresses to MAC addresses, with a TTL (~20 minutes). Cuts down on repeated broadcasts.
> **Default gateway:** the router IP a host sends to when the destination is on a different subnet — and therefore the IP a host ARPs for in that case (instead of ARPing for the final destination).
> **Routing table:** a per-router lookup table that tells the router which outbound interface to use for each destination IP prefix.

---

## 📊 Summary

| Q | Topic | Key result |
|---|---|---|
| 1 | 802.11 throughput | ≈43.65 Mbps (75% of 58 Mbps physical) |
| 2 | NAV interpretation | D closer to A (heard A's DATA) |
| 3 | Hidden terminal w/ RTS-CTS | Defer on hearing CTS |
| 4 | Concurrent transmissions | None possible (dense topology) |
| 5 | MACA on 6-line | YES — A→B and E→F concurrently (spatial reuse) |
| 6 | Switch self-learning | Tables grow with each packet; flooding stops once known |
| 7 | Switches+hubs domains | 1 broadcast, 6 collision |
| 8 | Router+switches | 2 broadcast, ~10 collision |
| 9 | VLANs default | 2 broadcast (router-separated) |
| 10 | ARP across VLANs | Tag added at trunk, stripped before host |
| 11 | Cross-subnet ARP | IPs unchanged, MACs rewritten at each router |

---

## 🔑 Master Reference

```
┌─────────────────────────────────────────────────────────────┐
│  802.11 DCF EXCHANGE                                         │
│  DIFS → backoff → RTS → SIFS → CTS → SIFS → DATA → SIFS → ACK│
│  Throughput = useful_bits / total_time                       │
├─────────────────────────────────────────────────────────────┤
│  HIDDEN/EXPOSED TERMINAL                                     │
│  Hear CTS → STAY QUIET (you're near receiver)                │
│  Hear RTS only → safe to transmit (you're near sender)       │
│  RTS/CTS solves hidden terminal problem                      │
├─────────────────────────────────────────────────────────────┤
│  SWITCH SELF-LEARNING                                        │
│  Frame in on port P, src X → learn (X, P)                   │
│  Lookup dst: found → unicast; not found → flood              │
│  Initially empty → lots of flooding; stabilizes fast         │
├─────────────────────────────────────────────────────────────┤
│  DOMAINS                                                     │
│  Hub: 1 collision, 1 broadcast                               │
│  Switch: 1 collision per port, 1 broadcast                   │
│  Router: 1 collision per interface, 1 broadcast per interface│
├─────────────────────────────────────────────────────────────┤
│  ARP CROSS-LAN                                               │
│  IPs stay end-to-end                                          │
│  MACs change at every router hop                             │
│  Sender ARPs for default gateway, not destination            │
└─────────────────────────────────────────────────────────────┘
```

---

If anything's unclear (especially the switch self-learning trace in Q6 or the cross-subnet ARP in Q11), point at which step and I'll expand.
