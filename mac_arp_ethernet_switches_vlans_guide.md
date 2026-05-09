# MAC, ARP, Ethernet, Switches & VLANs — Full Study Guide

> **How to use:** Each section names the slide(s) it covers. Open your PPT side-by-side. ASCII diagrams render best in monospace.

---

## 0. The Big Picture — Why This Chapter Exists

You already know that computers talk over a shared channel using MAC protocols (CSMA/CD etc.). But two questions remain:

1. **Who is the message for?** → We need addresses.
2. **How do those addresses connect to IP addresses (which is what apps actually use)?** → That's ARP.
3. **How do real-world Ethernet networks actually look?** → Switches, frames, VLANs.

So this chapter is all about **addressing + the real Ethernet ecosystem**.

```
   Application (uses IP)
         │
         ▼   "I want to send to 137.196.7.78"
   ┌─────────────────────────────┐
   │   Network Layer (IP)        │   needs MAC address to actually deliver
   └─────────────────────────────┘
         │
         ▼   ARP figures out the MAC for that IP
   ┌─────────────────────────────┐
   │   Link Layer (Ethernet)     │   sends frame using MAC addresses
   └─────────────────────────────┘
         │
         ▼
   [Wire / Switch / Other devices]
```

---

## 1. MAC Addresses (📑 Slide 1: "MAC addresses and ARP")

### Two kinds of address — don't confuse them!

| | IP Address | MAC Address |
|---|---|---|
| Length | 32 bits | 48 bits |
| Lives at | Network layer | Link layer |
| Job | Routing across the internet | Delivering to a specific NIC on a LAN |
| Portable? | No (changes when you move network) | Yes (burnt into the card) |
| Analogy | Postal address (changes if you move house) | Social Security Number (always yours) |

### What a MAC address looks like (📑 Slide 5: "Addressing")

48 bits → 12 hex digits, written with colons or dashes:

```
06 : 01 : 02 : 01 : 2C : 4B
└──┘ └──┘ └──┘ └──┘ └──┘ └──┘
 1 byte each (8 bits) → total 6 bytes = 48 bits
```

### Where MACs come from (📑 Slide 3: "LAN addresses (more)")

- **IEEE** assigns the first 3 bytes (the OUI = Organizationally Unique Identifier) to each manufacturer
- The manufacturer assigns the last 3 bytes uniquely
- This guarantees every NIC on Earth has a unique MAC — globally unique
- The MAC is **burnt into ROM** on the network card at the factory

### MAC vs IP — the analogy that helps

- **MAC** = Social Security Number → identifies *you* no matter where you live
- **IP** = Postal address → identifies *where you currently are* on the internet

Move your laptop to a new WiFi network → new IP, **same MAC**.

### Unicast vs Broadcast vs Multicast (📑 Slide 6)

The **first bit of the first byte** of the MAC determines the type:

| First bit | Type | Meaning |
|---|---|---|
| 0 | Unicast | One specific recipient |
| 1 | Multicast | A group of recipients |
| All 1s (FF:FF:FF:FF:FF:FF) | Broadcast | Everyone on the LAN |

---

## 2. ARP — Address Resolution Protocol (📑 Slides 4–9)

### The Problem ARP Solves

Your computer has the IP address of who it wants to talk to (e.g., from DNS). But to actually send a frame on Ethernet, you need the **MAC address** — and you don't know it.

> **ARP's job:** Given an IP address, find the corresponding MAC address (on the same LAN).

### ARP Table (📑 Slide 4)

Every node keeps a small table mapping IP → MAC:

```
┌─────────────────┬───────────────────┬──────┐
│   IP Address    │   MAC Address     │ TTL  │
├─────────────────┼───────────────────┼──────┤
│ 137.196.7.78    │ 1A-2F-BB-76-09-AD │ 20m  │
│ 137.196.7.23    │ 71-65-F7-2B-08-53 │ 15m  │
└─────────────────┴───────────────────┴──────┘
```

- **TTL** = Time To Live (how long this entry is valid, typically 20 minutes)
- If you don't have the entry → run ARP to get it
- After TTL expires → entry is removed; ask again next time

### ARP Protocol — How It Works (📑 Slides 5–6)

Suppose A wants to send to B (B's IP = 137.196.7.78). A doesn't know B's MAC.

```
Step 1: A broadcasts an ARP REQUEST to ALL nodes on the LAN
        Destination MAC = FF:FF:FF:FF:FF:FF (broadcast)
        Payload: "Who has IP 137.196.7.78? Tell me your MAC."

   A ──REQ──► [LAN]──► everyone (B, C, D, ...)

Step 2: ONLY B replies (its IP matches)
        B sends a UNICAST ARP REPLY directly to A
        Payload: "I'm 137.196.7.78. My MAC is 1A-2F-BB-76-09-AD."

   B ──REPLY──► A   (just to A, not broadcast)

Step 3: A caches the result in its ARP table for ~20 minutes
        Now A can send the actual frame to B.
```

### Why ARP is "plug-and-play"

- Nodes build ARP tables **automatically** — no admin needed
- New device plugs in → others learn about it the first time they need to talk
- Compare with manually configuring routes everywhere: ARP is much nicer

### ARP Packet Structure (📑 Slide 8)

```
┌─────────────┬─────────────┬─────────┬───────┬──────────────────┐
│ Hardware    │ Protocol    │ HW len  │ Op    │ Sender HW addr   │
│ type        │ type        │ Plen    │       │ Sender IP addr   │
│ (Ethernet=1)│ (IPv4=0x800)│ (6, 4)  │(1=req,│ Target HW addr   │
│             │             │         │ 2=rep)│ Target IP addr   │
└─────────────┴─────────────┴─────────┴───────┴──────────────────┘
```

The whole ARP packet is wrapped in an Ethernet frame with **EtherType = 0x0806** (so the receiver knows "this is ARP, not regular IP data").

---

## 3. Routing to Another LAN (📑 Slides 10–13: "Addressing: routing to another LAN")

This is the **most common confusion point** of the whole chapter — slow down here.

### Setup

```
    LAN 1                        LAN 2
   ┌──────┐                     ┌──────┐
   │  A   │                     │  B   │
   └──┬───┘                     └──┬───┘
      │                            │
      │     ┌──────────┐           │
      └─────┤ Router R ├───────────┘
            └──────────┘
        (R has 2 interfaces:
         R1 on LAN1, R2 on LAN2)
```

A wants to send an IP datagram to B. They are on **different LANs**, so a router is in between.

### The KEY insight

- **IP addresses** in the datagram stay the same end-to-end (always A → B)
- **MAC addresses** in the frame change at every hop

Why? Because MAC is *local* — it only matters within one LAN. As soon as the datagram crosses the router, a new frame is built with new MAC addresses for the next LAN.

### Step-by-step trace

#### Step 1: A creates IP datagram
```
[ IP src = A,  IP dst = B,  payload ]
```

#### Step 2: A creates link-layer (Ethernet) frame for LAN 1
A sees that B is on a different network. A's routing table says "to reach LAN 2, send via router R." So A puts **R's MAC** as the destination, **not B's MAC**.

```
LAN 1 frame:
┌───────────────┬───────────────┬─────────────────────────────────┐
│ MAC dst = R1  │ MAC src = A   │ IP datagram (src=A, dst=B)      │
└───────────────┴───────────────┴─────────────────────────────────┘
            ↑
   destination is the ROUTER, not B
```

#### Step 3: R receives the frame, strips link layer, looks at IP
R sees: "IP destination is B, which is on LAN 2 (my other interface)." R needs to forward the IP datagram to B.

#### Step 4: R creates a NEW frame for LAN 2
Now the destination MAC is **B's MAC**, and the source MAC is **R2's MAC** (R's interface on LAN 2).

```
LAN 2 frame:
┌───────────────┬───────────────┬─────────────────────────────────┐
│ MAC dst = B   │ MAC src = R2  │ IP datagram (src=A, dst=B)      │
└───────────────┴───────────────┴─────────────────────────────────┘
```

#### Step 5: B receives the frame
B's MAC matched → it processes the frame, hands the IP datagram up to its network layer.

### The two-line summary

> **MAC changes hop-by-hop. IP stays end-to-end.**

If you remember only one thing from this chapter, make it that.

---

## 4. What is Ethernet? (📑 Slides 14–15)

- Ethernet = the most widely used LAN technology, invented in 1970s at Xerox
- A system to connect computers → form a local network → using protocols to control simultaneous transmission (the MAC stuff from last chapter)
- Standardized as **IEEE 802.3**

### IEEE 802 architecture (📑 Slide 16: "IEEE Ethernet")

The link layer is split into **two sublayers**:

```
┌──────────────────────────────────────────────────────┐
│  LLC (Logical Link Control) — IEEE 802.2             │
│  • Multiplexing: lets multiple network protocols     │
│    (IP, IPX, AppleTalk) share one link               │
│  • Implemented in SOFTWARE                           │
├──────────────────────────────────────────────────────┤
│  MAC (Medium Access Control) — IEEE 802.3            │
│  • Framing, MAC addressing                           │
│  • Medium access control (CSMA/CD, token, etc.)      │
│  • Implemented in HARDWARE (NIC)                     │
└──────────────────────────────────────────────────────┘
```

### Ethernet Physical Topologies (📑 Slide 17)

Two main shapes the wiring can take:

```
BUS topology (old, before mid-90s):       STAR topology (today):
                                         
   ───┬───┬───┬───┬───                          [Hub or Switch]
      │   │   │   │                            ╱   │   │   ╲
     PC  PC  PC  PC                          PC   PC   PC   PC
     
  one shared coaxial cable               every PC has its own cable
  (any collision affects all)            to a central device
```

Today: 100% star topology.

---

## 5. Broadcast & Collision Domains (📑 Slides 18–20)

These two terms are **always confused** — get them straight now.

### Collision Domain

A region of the network where **if two devices transmit at the same time, their signals will collide**.

```
Collision domain example (HUB connects everyone):

       [Hub]
      ╱  │  ╲
     A   B   C
     
If A and C transmit simultaneously → collision.
A, B, C are all in the SAME collision domain.
```

A **switch** breaks collision domains — each port is its own collision domain.

### Broadcast Domain

A region of the network where **a broadcast frame will reach every device**.

```
Broadcast domain: bounded by ROUTERS.
A broadcast (FF:FF:FF:FF:FF:FF) goes to everyone in the LAN
but does NOT cross routers.
```

A **router** breaks broadcast domains.

### Quick rules to memorize

| Device | Breaks collision domain? | Breaks broadcast domain? |
|---|---|---|
| Hub | NO (just repeats signal) | NO |
| Switch | **YES** (each port = own collision domain) | NO |
| Router | YES | **YES** |

### Worked Example (📑 Slide 20 question)

> The diagram has a router connecting 2 hubs (Hub-A with PCs A, B; Hub-B with PC C, plus a third hub somewhere). How many broadcast and collision domains?

**Answer pattern:**
- **Broadcast domains** = number of router-separated networks → here, 2
- **Collision domains** = count each hub-network as one collision domain (and each switch port if any). With the typical layout (2 hubs + a small extra) → 3 collision domains.

So the answer for the slide diagram is typically: **(B) 2 broadcast domains and 3 collision domains.**

---

## 6. Normal Ethernet Operation (📑 Slide 21)

How a frame travels:

```
Sender                                     Receiver
   │                                          │
   │ Address mismatch                         │
   │ (frame not for me) → discard             │
   │ ◄──────────────────────                  │
   │                                          │
   │ Transmitted packet                       │ Address match
   │ (one collision per LAN broadcast frame)  │ → process
   │ ──────────────────────────────────────►  │
   │                                          │
```

Every NIC sees every frame on the wire (in a hub/bus). The NIC's hardware checks the destination MAC and:
- **Match (or broadcast)** → pass to OS
- **No match** → silently drop

---

## 7. Ethernet Generations (📑 Slides 22–23)

Ethernet has evolved while keeping the same frame format:

```
Standard Ethernet → Fast Ethernet → Gigabit Ethernet → 10-Gigabit Ethernet
   10 Mbps           100 Mbps          1 Gbps              10 Gbps
```

### Standard Ethernet (📑 Slide 23)
- 10 Mbps (the original)
- **Connectionless**: no handshake before sending
- **Unreliable**: no acknowledgments at the link layer (upper layers handle retransmission if needed)
- Frames may be dropped silently
- Uses **CSMA/CD** for medium access

---

## 8. Ethernet Frame Format (📑 Slides 24–27)

This is the actual structure of bytes on the wire:

```
┌──────────┬──────┬──────┬──────┬──────┬───────────────┬──────┐
│ Preamble │ Dest │ Src  │ Type │ Len  │  Data + Pad   │ CRC  │
│ (7 bytes)│ MAC  │ MAC  │      │      │               │      │
│          │ 6B   │ 6B   │ 2B   │      │  46–1500 B    │ 4 B  │
└──────────┴──────┴──────┴──────┴──────┴───────────────┴──────┘
+ SFD (1 byte) inside preamble
```

### Field-by-field

| Field | Size | Purpose |
|---|---|---|
| **Preamble** | 7 bytes | Pattern `10101010` × 7 — synchronizes receiver clock to sender |
| **SFD** (Start Frame Delimiter) | 1 byte | `10101011` — "frame is starting now!" |
| **Destination MAC** | 6 bytes | Where the frame is going |
| **Source MAC** | 6 bytes | Where the frame came from |
| **Type / Length** | 2 bytes | If ≥ 0x0600 → it's the EtherType (0x0800=IP, 0x0806=ARP). Else it's length. |
| **Data + Padding** | 46–1500 bytes | Payload. Must be ≥ 46 bytes (pad if shorter) so total frame meets CSMA/CD min size |
| **CRC** | 4 bytes | Cyclic Redundancy Check. Receiver recomputes; if mismatch → drop frame |

### Why the 46-byte minimum?
Because total frame must be ≥ 64 bytes (the CSMA/CD min frame size we calculated last chapter):
- 6 (dst) + 6 (src) + 2 (type) + 46 (data) + 4 (CRC) = **64 bytes** ✓

### Maximum frame size: 1518 bytes (or 1500 byte payload)

---

## 9. Categories of Standard Ethernet (📑 Slide 30)

Standard 10 Mbps Ethernet has 4 wiring variants:

| Name | Cable | Topology |
|---|---|---|
| **10Base5** | Thick coaxial | Bus |
| **10Base2** | Thin coaxial | Bus |
| **10Base-T** | Twisted pair (UTP) | Star |
| **10Base-F** | Fiber optic | Star |

The naming convention:
- **10** = speed in Mbps
- **Base** = baseband signaling (vs broadband)
- **5/2/T/F** = max segment length in 100m (so 5=500m, 2=200m) OR cable type

---

## 10. Ethernet Switch (📑 Slides 32–35)

A switch is the modern replacement for a hub. It's a smart link-layer device.

### What a switch does

1. **Stores** every incoming frame
2. **Examines** the destination MAC
3. **Selectively forwards** the frame to ONLY the port leading to that MAC (not all ports like a hub)
4. Uses CSMA/CD per segment if needed

### Why switches are awesome

- **Transparent** — hosts don't even know switches exist
- **Plug-and-play, self-learning** — no configuration needed
- **No collisions** between hosts on different ports → much higher throughput
- Each port is its own collision domain → hosts can transmit simultaneously

### Multiple Simultaneous Transmissions (📑 Slide 35)

Because each port is independent, the switch can support many parallel conversations at once:

```
        [Switch]
       ╱  │  │  ╲
      A   B  C   D
      
A → B and C → D can happen at the SAME TIME (no collision).
A switch with N ports can support up to N/2 simultaneous transmissions.
```

---

## 11. Switch Forwarding Table & Self-Learning (📑 Slides 36–40)

How does the switch know which port leads to which MAC?

### The Switch Table

```
┌──────────────────┬───────────┬──────┐
│   MAC Address    │ Interface │ TTL  │
├──────────────────┼───────────┼──────┤
│ A's MAC          │     1     │ 60   │
│ B's MAC          │     1     │ 60   │
│ C's MAC          │     2     │ 60   │
└──────────────────┴───────────┴──────┘
```

Like an ARP table, but mapping **MAC → physical port** (not IP → MAC).

### Self-Learning Algorithm

The switch starts with an **empty table**. It learns by watching:

```
When a frame arrives on port P with source MAC = X:
   → "Aha! X is reachable via port P."
   → Add (X, P) to table.
```

### Frame Filtering / Forwarding Logic

```
When frame arrives at switch with destination MAC = D:
   1. Record (source MAC, incoming port) in table  ← learning
   2. Look up D in table:
      ┌─ FOUND on port P:
      │   ├─ if P == incoming port → drop (it would loop back)
      │   └─ else → forward only to P    ← selective forwarding
      └─ NOT FOUND:
          → flood (forward on all ports except incoming)  ← like a hub, just for unknowns
```

### Worked Example: Self-Learning (📑 Slide 40)

```
Switch starts empty.
Frame: A → A'  (i.e., source A, destination A')
       arrives on port 1.

Step 1: Switch records (A, port 1).
Step 2: Looks up A' → not in table → FLOOD on ports 2, 3, 4
        (every port except port 1).
Step 3: A' (let's say on port 4) eventually replies.
        Frame: A' → A arrives on port 4.
Step 4: Switch records (A', port 4).
Step 5: Looks up A → found on port 1 → forward only to port 1.

Now the switch has learned both. Future A↔A' frames go directly between ports 1 and 4.
```

---

## 12. Interconnecting Switches (📑 Slides 41–42)

You can chain switches together:

```
    A     B           E     F
     ╲   ╱             ╲   ╱
     [S1] ─── [S2] ─── [S3]
       │       │
       C       D       G    H
                       │   ╱
                      [S4]
```

**Self-learning still works** — each switch independently builds its table by snooping on source MACs of frames passing through.

### Example query

> Sending from A to G — how does S₁ know to forward toward G?

**Answer:** Same as single-switch case! S₁'s table eventually contains "G is reachable via the port leading to S₂." It learned this when G previously sent a frame somewhere and that frame passed through S₁.

---

## 13. Switches vs Routers (📑 Slide 43)

| | Switch | Router |
|---|---|---|
| Layer | Link (Layer 2) | Network (Layer 3) |
| Uses | MAC addresses | IP addresses |
| Forwarding decision | Self-learning, MAC table | Routing protocols, IP table |
| Plug-and-play | YES | NO (needs config) |
| Breaks collision domain | YES | YES |
| Breaks broadcast domain | NO | YES |

Both are store-and-forward devices.

---

## 14. VLANs — Virtual LANs (📑 Slides 44–49)

### Why VLANs exist (the motivation)

Imagine a corporate switched LAN:

```
     [Big Switch]
    ╱╱  ││  ╲╲
  CS  EE  Sci  HR  ...
```

Problems with one giant flat LAN:
1. **No traffic isolation** — broadcasts from HR reach Computer Science (wasted bandwidth)
2. **Inefficient use of switches** — small departments could share, but you'd have to physically rewire
3. **Security concerns** — anyone can sniff anyone's broadcast traffic

### What is a VLAN?

A **VLAN (Virtual LAN)** = a switch (or set of switches) configured by software so that **one physical switch behaves like multiple separate switches**.

Each VLAN = its own broadcast domain.

```
Physical: ONE switch, 8 ports

Logical (after VLAN config):
   ┌──────VLAN 10 (CS dept)──────┐    ┌──────VLAN 20 (EE dept)──────┐
   │  Port 1   Port 2   Port 3   │    │  Port 5   Port 6   Port 7   │
   └─────────────────────────────┘    └─────────────────────────────┘

Ports 1–3 act as if they're on a separate switch from ports 5–7.
Broadcasts from Port 1 do NOT reach Port 5.
```

### Port-based VLAN (📑 Slide 47)

The simplest kind: assign each switch port to a VLAN by configuration.

- **Traffic isolation** — VLAN 10's frames never reach VLAN 20
- **Dynamic membership** — admin can move a port to a different VLAN by software command (no physical rewiring!)
- **Forwarding between VLANs** = needs a router (because they're separate broadcast domains)

### VLANs Spanning Multiple Switches (📑 Slide 48)

What if VLAN 10 has members on Switch1 AND Switch2? You connect the switches with a special **trunk port** that carries traffic for *all* VLANs.

```
   Switch 1                     Switch 2
   ┌──────────┐                 ┌──────────┐
   │ V10  V10 │                 │  V10 V20 │
   │  │    │  │ ───trunk───►    │   │   │  │
   │  P1   P2 │                 │   P3  P4 │
   └──────────┘                 └──────────┘

The trunk link carries frames from BOTH VLAN 10 and VLAN 20.
How does Switch 2 know which VLAN a frame belongs to?
→ Use a TAGGED frame (IEEE 802.1Q).
```

### IEEE 802.1Q Frame Format (📑 Slide 49)

A normal Ethernet frame has no VLAN info. To carry VLAN ID across trunk links, we **insert a 4-byte tag** between source MAC and Type field:

```
Normal Ethernet frame:
[Preamble | Dst MAC | Src MAC | Type | Data | CRC ]

802.1Q tagged frame:
[Preamble | Dst MAC | Src MAC | TAG (4B) | Type | Data | CRC ]
                                  ↑
                                  └─ contains VLAN ID (12 bits = up to 4096 VLANs)
                                     + priority (3 bits)
                                     + format indicator (1 bit)
```

### How tagging works in practice

```
1. PC on VLAN 10 sends a frame to Switch 1.
   (Frame has NO tag — PCs don't know about VLANs.)

2. Switch 1 sees frame entered on VLAN-10 port.
   Before sending out the trunk, Switch 1 INSERTS the VLAN tag.

3. Frame travels across trunk with tag = "VLAN 10".

4. Switch 2 reads the tag → knows this belongs to VLAN 10
   → forwards only to VLAN-10 ports on Switch 2.

5. Before sending to the actual PC, Switch 2 STRIPS the tag.
   (PC sees a normal untagged frame.)
```

So tagging is **invisible to end hosts** — only switches deal with tags, and only on trunk links.

### Big picture: VLANs let you...

- Separate broadcast domains by software (no rewiring)
- Move a user to a "different LAN" by just changing port config
- Span "logical LANs" across multiple physical switches
- Use one trunk link to carry traffic for many VLANs

---

## 15. NUMERICAL & CONCEPTUAL QUESTIONS — Worked

### 🧮 Q1: How many domains?

> Diagram: Router R connects to Hub H1 (PCs A, B) and Hub H2 (PC C). How many broadcast domains? Collision domains?

**Step 1 — Broadcast domains:**
Routers split broadcast domains. R has 2 sides → **2 broadcast domains**.

**Step 2 — Collision domains:**
Each hub = 1 collision domain (everything on a hub shares it).
- H1 = 1 collision domain (A, B share it)
- H2 = 1 collision domain (C alone, but still 1)
- Plus the router-to-hub link counts as part of the hub's domain (since hubs don't separate them).

**Answer: 2 broadcast domains, 2 collision domains** (or 3 depending on whether the diagram has an extra hub).

The slide-style answer "2 broadcast / 3 collision" usually assumes a third hub or a switch-port-as-domain.

### 🧮 Q2: ARP Walkthrough

> A (IP=10.0.0.1, MAC=AA) wants to send to B (IP=10.0.0.2). Empty ARP table. Trace the messages.

```
1. A checks ARP table for 10.0.0.2 → not found.

2. A broadcasts ARP request:
   Eth dst = FF:FF:FF:FF:FF:FF
   Eth src = AA
   Payload: "Who has 10.0.0.2? Tell 10.0.0.1 (AA)"

3. Every node receives. Only B matches.

4. B sends ARP reply (UNICAST, not broadcast):
   Eth dst = AA
   Eth src = BB
   Payload: "10.0.0.2 is at BB"

5. A caches: 10.0.0.2 → BB (TTL 20 min).

6. A sends data frame:
   Eth dst = BB
   Eth src = AA
   Payload: actual IP datagram
```

### 🧮 Q3: Cross-LAN Routing

> A (10.0.0.1, MAC=AA, on LAN1) sends to B (20.0.0.1, MAC=BB, on LAN2). Router R has interfaces R1 (10.0.0.254, MAC=R1M on LAN1) and R2 (20.0.0.254, MAC=R2M on LAN2). Trace the MAC and IP fields at each step.

```
On LAN 1 (frame from A to R):
   IP src = 10.0.0.1     IP dst = 20.0.0.1
   MAC src = AA          MAC dst = R1M

On LAN 2 (frame from R to B):
   IP src = 10.0.0.1     IP dst = 20.0.0.1   ← SAME IPs
   MAC src = R2M         MAC dst = BB         ← NEW MACs
```

**Notice:** IP addresses unchanged end-to-end. MAC addresses rewritten at the router.

### 🧮 Q4: Switch Self-Learning

> Switch with 4 ports. Empty table. Sequence: (1) A→C, (2) C→A, (3) B→C, (4) D→B. Show the table after each step. A on port 1, B on port 2, C on port 3, D on port 4.

| Step | Frame | Action | Table after |
|---|---|---|---|
| 1 | A→C | Learn (A, port 1). C unknown → flood ports 2,3,4 | (A,1) |
| 2 | C→A | Learn (C, port 3). A known on port 1 → forward to port 1 only | (A,1) (C,3) |
| 3 | B→C | Learn (B, port 2). C known on port 3 → forward to port 3 only | (A,1) (B,2) (C,3) |
| 4 | D→B | Learn (D, port 4). B known on port 2 → forward to port 2 only | (A,1) (B,2) (C,3) (D,4) |

After step 4, fully learned. No more flooding needed.

---

## 16. EXAM CHEAT SHEET

```
┌─────────────────────────────────────────────────────────────┐
│  ADDRESSES                                                   │
│  • IP = 32 bits, network layer, portable, like postal addr  │
│  • MAC = 48 bits, link layer, burnt-in, like SSN            │
│  • Broadcast MAC = FF:FF:FF:FF:FF:FF                         │
├─────────────────────────────────────────────────────────────┤
│  ARP                                                         │
│  • Maps IP → MAC on same LAN                                 │
│  • REQUEST = broadcast; REPLY = unicast                      │
│  • Caches result with TTL ~20 min                            │
│  • EtherType for ARP = 0x0806                                │
├─────────────────────────────────────────────────────────────┤
│  CROSS-LAN ROUTING                                           │
│  • IP src/dst stay end-to-end                                │
│  • MAC src/dst rewritten at every router hop                 │
├─────────────────────────────────────────────────────────────┤
│  ETHERNET FRAME                                              │
│  • Preamble (7) + SFD (1) + Dst (6) + Src (6) +             │
│    Type (2) + Data (46–1500) + CRC (4)                       │
│  • Min frame = 64 B, Max = 1518 B                            │
│  • EtherType: IP=0x0800, ARP=0x0806                          │
├─────────────────────────────────────────────────────────────┤
│  DOMAINS                                                     │
│  • Hub: 1 collision dom, 1 broadcast dom                     │
│  • Switch: each port = own collision dom; 1 broadcast dom    │
│  • Router: each interface = own collision + broadcast dom    │
├─────────────────────────────────────────────────────────────┤
│  SWITCH                                                      │
│  • Self-learning: source MAC + incoming port → table         │
│  • Forward: known dst → 1 port; unknown dst → flood          │
│  • N-port switch supports up to N/2 simultaneous flows       │
├─────────────────────────────────────────────────────────────┤
│  VLAN                                                        │
│  • One physical switch → many virtual switches (by config)   │
│  • Each VLAN = own broadcast domain                          │
│  • Trunk port carries multiple VLANs                         │
│  • 802.1Q tag: 4 bytes, contains VLAN ID (12 bits = 4096)    │
│  • Tag inserted by switch on trunk; stripped before host     │
└─────────────────────────────────────────────────────────────┘
```

---

## 17. Study Plan

1. **Today:** Sections 0–3. Especially MAC vs IP and the cross-LAN routing trace — that's the conceptual backbone.
2. **Tomorrow:** Sections 4–9 (Ethernet basics, frame format, domains).
3. **Day 3:** Sections 10–14 (switches, self-learning, VLANs). Do the worked examples in section 15 by hand.
4. **Day 4 (exam day):** Cheat sheet only.

If anything is fuzzy, ask:
- "Walk me through the cross-LAN routing again with different IPs"
- "Make me 3 self-learning switch problems"
- "Explain 802.1Q tagging step by step"
- "Quiz me on broadcast vs collision domains"
