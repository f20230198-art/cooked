# Introduction, OSI Model & Performance — Full Study Guide (Chapter 1)

> **How to use:** Each section names the slide(s) it covers. Open your PPT side-by-side. ASCII diagrams render best in monospace. The numerical problems at the end are exam gold — practice them.

---

## 0. The Big Picture — What This Chapter Is About

This is the foundation chapter. It answers four big questions:

1. **What is a network?** (basics, topologies, types)
2. **How is the Internet structured?** (edge, core, access)
3. **How do we organize all the protocols?** (OSI model — 7 layers)
4. **How do we measure network performance?** (bandwidth, throughput, delay, latency)

```
   Foundation chapter
        │
   ┌────┼─────────┬───────────┬──────────────┐
   ▼    ▼         ▼           ▼              ▼
Basics Topologies Internet  OSI Model    Performance
       (Bus,Star,             (7 layers)   (delay, BW,
        Ring,Mesh)                          throughput)
```

---

## 1. Data Communications Basics (📑 Slide 4)

### Key terms

- **Telecommunication** — communication at a distance
- **Data** — information in any agreed-upon form
- **Data communication** — exchange of data between two devices via some transmission medium

### The 5 components of data communication

```
   [Sender] ──── Message ────► [Receiver]
       │                            │
   uses Protocol           uses Protocol
   (rules)                 (must be the same!)
       │                            │
       └──── Transmission Medium ───┘
              (wire, fiber, air)
```

| Component | What it is |
|---|---|
| Sender | Device that originates the message |
| Receiver | Device that gets the message |
| Message | The actual data |
| Medium | Physical path (cable, wireless) |
| Protocol | Rules both sides follow |

---

## 2. What Is a Network? (📑 Slide 5)

A **network** = set of devices (called **nodes**) connected by **communication links**.

A **node** = anything that can send/receive data: computer, printer, phone, router, etc.

### Network Criteria (4 things every good network must have)

1. **Fault Tolerance** — keeps working even when parts fail
2. **Scalability** — can grow without redesign
3. **QoS (Quality of Service)** — guarantees about speed, delay, reliability
4. **Security** — protection from unauthorized access

---

## 3. Why We Need Computer Networks (📑 Slides 6–10)

5 main reasons:

| Use | Example |
|---|---|
| **File sharing** | Shared drive, file server |
| **Hardware sharing** | One printer for many users |
| **User communication** | Email, chat, video calls |
| **Application sharing** | Client/server apps (banking, ERP) |
| **Network gaming** | Multi-player games |
| **VoIP** | Internet calls (replaces traditional phone/PSTN) |

---

## 4. Data Flow Direction (📑 Slide 12)

Three modes of communication between two devices:

```
(a) SIMPLEX — one direction only

   [Sender] ──────────► [Receiver]
   (e.g., Mainframe → Monitor, keyboard → CPU)

(b) HALF-DUPLEX — both directions, but one at a time

   [Station A] ──────────► [Station B]   (time 1)
   [Station A] ◄────────── [Station B]   (time 2)
   (e.g., walkie-talkies)

(c) FULL-DUPLEX — both directions simultaneously

   [Station A] ◄────────► [Station B]
   (e.g., phone call, modern Ethernet)
```

---

## 5. Types of Connections (📑 Slide 13)

```
(a) POINT-TO-POINT — dedicated link between exactly two devices
    [Station] ──────── [Station]

(b) MULTIPOINT — one link shared by multiple devices
    [Mainframe] ──┬───┬───┬─────
                  │   │   │
                [Stn][Stn][Stn]
```

---

## 6. Network Topologies (📑 Slides 14–27)

**Topology** = the physical arrangement of cables, computers, and devices.

```
              Topology
                 │
   ┌─────┬───────┼───────┬──────┐
   ▼     ▼       ▼       ▼      ▼
 Mesh   Star    Bus    Ring   Hybrid
```

### 6a. Bus Topology (📑 Slides 15–17)

```
Cable end ─[Tap]──[Tap]──[Tap]── Cable end
            │      │      │
          [Stn]  [Stn]  [Stn]
```

**Single cable**, all nodes tap into it. Terminators at both ends prevent signal bounce.

✅ **Pros:** Cheap, easy to install, good for small networks. One node failing doesn't kill others.
❌ **Cons:** Hard to fault-isolate, congestion-prone, no security, cable break = whole network down.

Used with **Ethernet (CSMA/CD)** in old setups.

### 6b. Star Topology (📑 Slides 18–20)

```
              [Hub or Switch]
              ╱   │   │   ╲
            [Stn][Stn][Stn][Stn]
```

Every node connects to a **central hub/switch**.

✅ **Pros:** Easy to manage, scalable, secure, one cable failure = only one node affected. **Most popular today.**
❌ **Cons:** Hub is single point of failure. Needs more cable than bus.

### 6c. Ring Topology (📑 Slides 21–24)

```
        [Stn]──────[Stn]
         │           │
        [Stn]      [Stn]
         │           │
        [Stn]──────[Stn]
```

Each node connects to two neighbors → forms a circle. Data flows in **one direction**. Uses **token passing** for access.

✅ **Pros:** Easy fault location, great for long distances and high traffic, reliable.
❌ **Cons:** Expensive, single point of failure (one break = everyone affected), less common today.

### 6d. Mesh Topology (📑 Slides 25–26)

```
        [Stn]
       ╱│╲ ╲
      ╱ │ ╲ ╲
   [Stn]─┼──[Stn]
     │ ╳ │ ╳ │
   [Stn]─┴──[Stn]
```

**Every node connects to every other node directly.** Maximum redundancy.

For n nodes, you need **n(n−1)/2 cables**.

✅ **Pros:** No traffic congestion, robust (multiple paths), easy fault identification.
❌ **Cons:** Massively expensive (lots of cable + ports), only used for backbone networks.

### 6e. Hybrid Topology (📑 Slide 27)

A combination — e.g., a **star backbone** connecting multiple **bus networks**.

```
         [Hub]
        ╱  │  ╲
       ╱   │   ╲
   ──Bus── ──Bus── ──Bus──
   |  |  |  |  |  |  |  |
  Stn Stn Stn Stn Stn Stn
```

---

## 7. Categories of Networks (📑 Slide 28)

Classified by **size/distance**:

| Type | Range | Example |
|---|---|---|
| **LAN** (Local Area Network) | Building, room | Office WiFi, school lab |
| **MAN** (Metropolitan Area Network) | City, campus | Cable TV network |
| **WAN** (Wide Area Network) | Country, world | The Internet |

---

## 8. Network Devices (📑 Slide 30)

The 4 essential building blocks:

1. **End Points** — PCs, servers, printers, phones (the things using the network)
2. **Interconnections** — NICs, cables, connectors (the physical glue)
3. **Switches** — connect endpoints to form a LAN
4. **Routers** — connect multiple LANs to form internetworks; choose best path between LANs and WANs

---

## 9. What is the Internet? (📑 Slides 31–35)

### Two ways to define it

#### "Nuts and bolts" view
The Internet = **a network of networks**, made of:
- **Hosts / End-systems** — millions of devices (PCs, phones, servers) running network apps
- **Communication links** — fiber, copper, radio, satellite (with different bandwidths)
- **Routers** — forward packets through the network
- **Protocols** — TCP, IP, HTTP, FTP, PPP — control sending/receiving

#### "Service" view
The Internet = a **communication infrastructure** enabling:
- Distributed apps (WWW, email, games, e-commerce)
- Communication services: **connectionless** OR **connection-oriented**

### What's a protocol?

> **Protocol** = format + order of messages sent and received between network entities + actions taken on transmission/receipt.

Human analogy: "Hi → Hi → Got the time? → 2:00" is a protocol. So is "TCP connection request → TCP reply → GET /file → response."

---

## 10. Network Structure (📑 Slides 38–49)

The Internet has 3 zones:

```
   ┌─────────────────────────────────────┐
   │           Network EDGE              │
   │  (your PC, phone, server)           │
   └────────────┬────────────────────────┘
                │
   ┌────────────▼────────────────────────┐
   │         ACCESS NETWORKS             │
   │  (DSL, cable, WiFi, cellular)       │
   └────────────┬────────────────────────┘
                │
   ┌────────────▼────────────────────────┐
   │           Network CORE              │
   │  (routers, "network of networks")   │
   └─────────────────────────────────────┘
```

### 10a. Network Edge (📑 Slide 39)

End systems run application programs at the "edge" of the network. Two interaction models:

- **Client/Server** — client requests, server provides (e.g., browser ↔ web server)
- **Peer-to-peer** — symmetric (e.g., teleconferencing, BitTorrent)

### 10b. Connection-Oriented vs Connectionless (📑 Slides 40–41)

| | TCP (Connection-Oriented) | UDP (Connectionless) |
|---|---|---|
| Setup | Handshake first | Just send |
| Reliable? | Yes (ACKs + retransmit) | No |
| Flow control? | Yes | No |
| Congestion control? | Yes | No |
| Used by | HTTP, FTP, SMTP, Telnet | Streaming media, VoIP, gaming |

### 10c. Network Core (📑 Slides 42–46)

The mesh of interconnected routers. **The fundamental question:** how is data transferred?

Two approaches:

#### Circuit Switching (📑 Slides 43–44)
- Reserve end-to-end resources for the "call"
- Dedicated bandwidth (no sharing)
- Guaranteed performance
- Requires call setup time
- Used in: traditional telephone network
- Bandwidth divided into pieces by **FDM** (frequency), **TDM** (time), or **CDM** (code)

```
FDM (Frequency Division):           TDM (Time Division):
┌──────────────┐                    ┌─┬─┬─┬─┬─┬─┬─┬─┐
│ 4 KHz user 1 │                    │1│2│3│4│1│2│3│4│
├──────────────┤   } link           └─┴─┴─┴─┴─┴─┴─┴─┘
│ 4 KHz user 2 │                     ↑
├──────────────┤              all "2" slots dedicated
│ 4 KHz user 3 │              to user 2's call
└──────────────┘
```

#### Packet Switching (📑 Slides 45–46)
- Data split into **packets**
- Packets share network resources (statistical multiplexing)
- Each packet uses full link bandwidth when sent
- No reservation — resources used as needed
- **Store and forward** — packets move one hop at a time
- **Congestion** — packets queue if demand exceeds capacity

```
       [Router]
       ┌──────┐
A ─┐   │ ████ │ ← queue of packets
B ─┼──►│ ████ │ ──► statistical multiplexing on shared link
       └──────┘
```

> **Internet uses packet switching.** Telephone networks use(d) circuit switching.

---

## 11. Access Networks (📑 Slides 49–52)

How end systems connect to edge routers:

- **Residential** — Dialup (56 Kbps), DSL (1–8 Mbps), Cable
- **Institutional** — Ethernet LAN (10/100/1000 Mbps)
- **Wireless** — WiFi, cellular (4G/5G)

Two questions to ask about any access network:
- What's the **bandwidth** (bits per second)?
- Is it **shared** (cable, Ethernet) or **dedicated** (DSL)?

---

## 12. The OSI Model (📑 Slides 53–58)

### History
- ISO released **OSI (Open Systems Interconnection)** in 1984
- Goal: solve compatibility problems as networks grew
- Divides networking into **7 layers** to reduce complexity

### The 7 Layers — MEMORIZE THE ORDER

```
┌─────────────────┐
│ 7. Application  │ ← Network processes to apps (HTTP, FTP, SMTP)
├─────────────────┤
│ 6. Presentation │ ← Data formatting, encryption, compression
├─────────────────┤
│ 5. Session      │ ← Inter-host communication, dialog control
├─────────────────┤
│ 4. Transport    │ ← End-to-end connections (TCP, UDP)
├─────────────────┤
│ 3. Network      │ ← Addressing & best-path routing (IP)
├─────────────────┤
│ 2. Data Link    │ ← Access to media (frames, MAC addr)
├─────────────────┤
│ 1. Physical     │ ← Binary transmission on the wire
└─────────────────┘
```

### Mnemonic to memorize (top→bottom)
**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing
(Application, Presentation, Session, Transport, Network, Data link, Physical)

Or bottom→top: **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

### Layer split

- **Upper 4 layers (5,6,7)** — application-oriented services
- **Lower 4 layers (1,2,3,4)** — flow of data through the network

### How layers communicate (📑 Slide 58)

**Each layer talks to the same layer on the other device** (peer-to-peer). But the *actual data* travels down through the local layers, across the wire, and back up.

```
Sender                                       Receiver
─────────                                    ─────────
App  ◄·············peer protocol············► App
 │                                              ▲
 ▼                                              │
Pres ◄·············peer protocol············► Pres
 │                                              ▲
 ▼                                              │
... (down through Phy)                       ... (up from Phy)
 │                                              ▲
 ▼                                              │
Phy  ──────── physical wire ─────────────►   Phy
```

---

## 13. Layer Functions Summary Table (📑 Slide 59 — VERY IMPORTANT)

| Layer | PDU | Function | Protocols |
|---|---|---|---|
| 7. Application | Data | Human-machine interface | HTTP, FTP, SMTP, DNS, TELNET, DHCP |
| 6. Presentation | Data | Encryption, format translation | SSL, TLS |
| 5. Session | Data | Establish/maintain/terminate sessions | PPTP, SIP, NetBIOS |
| 4. Transport | **Segment** | Segmentation, ACK, sequencing | TCP, UDP |
| 3. Network | **Packet** | Addressing & routing | IPv4, IPv6, ICMP |
| 2. Data Link | **Frame** | Error detection, flow control | Ethernet, PPP, ATM |
| 1. Physical | **Bits** | Bit transmission on a carrier | USB, Bluetooth |

> **PDU (Protocol Data Unit)** = the name for the data block at each layer. **Memorize these names: Bits → Frame → Packet → Segment → Data**

---

## 14. Encapsulation (📑 Slide 60)

As data goes DOWN the layers at the sender, each layer adds a **header** (and Layer 2 also adds a **trailer**). At the receiver, each layer strips its own header going UP.

```
SENDER                                  RECEIVER
─────────                               ─────────
[H7|D7]  ←Application                   [H7|D7]
[H6|D6 ]                                [H6|D6 ]
[H5|D5  ]                               [H5|D5  ]
[H4|D4   ]                              [H4|D4   ]
[H3|D3    ]                             [H3|D3    ]
[H2|D2     |T2]                         [H2|D2     |T2]
[01010101010101]──────► wire ─────────► [01010101010101]
```

Each header carries the info that layer needs (e.g., source/dest port at L4, source/dest IP at L3, source/dest MAC at L2).

---

## 15. Each Layer in Detail

### 15a. Application Layer (Layer 7) — 📑 Slides 61–62
- The interface to user applications
- Examples: FTP, HTTP, SMTP, SNMP, NFS, Telnet, DNS, DHCP
- "Allows access to network resources"

### 15b. Presentation Layer (Layer 6) — 📑 Slides 63–64
- **Format**, **syntax**, **semantics** of data
- Translates between different data formats
- Handles **encryption/decryption** and **compression**
- "Translate, encrypt, compress data"

### 15c. Session Layer (Layer 5) — 📑 Slides 65–66
- Establishes, maintains, terminates **sessions** (logical connections)
- **Dialog control**: full-duplex or half-duplex coordination
- **Synchronization & checkpointing** (so you can resume after a crash)
- Handles login/password validation

### 15d. Transport Layer (Layer 4) — 📑 Slides 67–69

**Critical layer.** Provides **process-to-process** delivery (vs hop-to-hop or host-to-host).

Responsibilities:
- **Segmentation & reassembly** — break large messages into segments, rebuild at receiver
- **Connection control** — connectionless (UDP) or connection-oriented (TCP)
- **Flow control** — don't overwhelm receiver
- **Error control** — end-to-end (vs Data Link's hop-to-hop)
- **Multiplexing** — multiple processes share one channel
- **Service-point addressing** — uses **port numbers**

```
HOST A (multiple processes)              HOST B
[Browser][Email][Game]                  [Web srv][Mail srv]
   │       │      │                         ▲        ▲
   └───┬───┴──────┘                         └────┬───┘
       │ port numbers identify processes         │
       ▼                                         │
   [Transport]──────── across network ─────────►[Transport]
```

### 15e. Network Layer (Layer 3) — 📑 Slides 71–73
- **Routing** packets through the network
- **Logical (IP) addressing**
- **Internetworking** — connecting different networks
- Fragmentation of packets to fit different media
- **Device: Router**

> **Source-to-destination delivery** (across multiple hops/networks)

```
[End A]──[Router B]──[Router E]──[End F]
       hop      hop      hop
       └── source-to-destination delivery ──┘
```

### 15f. Data Link Layer (Layer 2) — 📑 Slides 74–76
- **Framing** — wraps packets into frames with header + trailer
- **Physical (MAC) addressing**
- **Error detection & correction** (per hop)
- **Flow control** (per hop)
- **Access control** (CSMA/CD, etc.)
- **Devices: Bridge, Switch (multi-port bridge)**

> **Hop-to-hop delivery** (one link at a time)

### 15g. Physical Layer (Layer 1) — 📑 Slides 77–78
- Converts logical 1s and 0s into **electrical/optical/radio signals**
- Defines: cables, voltages, data rates, encoding, modulation
- **Devices: Repeater, Hub (multi-port repeater)**
- Handles transmission medium, line configuration, topology, transmission mode

> **Bit-by-bit delivery on one link**

---

## 16. Critical Difference: Where Each Layer's Job Happens (📑 Slide 68)

This trips students up. Memorize:

| Layer | Scope | Where errors checked |
|---|---|---|
| Data Link (L2) | **Hop-to-hop** | At every router/switch |
| Network (L3) | Host-to-host | End systems |
| Transport (L4) | **Process-to-process** | Only at end systems |

So:
- **Data link error control** → checks each hop (frame integrity)
- **Transport error control** → checks only end-to-end (segment integrity)

---

## 17. TCP/IP Protocol Suite (📑 Slides 81–82)

The actual stack used in the real Internet (slightly different from OSI):

```
OSI Model           TCP/IP (5-layer view)
─────────           ─────────────────────
7. Application  ┐
6. Presentation │   Application
5. Session      ┘    (HTTP, FTP, SMTP, DNS, ...)
4. Transport         Transport
                      (TCP, UDP, SCTP)
3. Network           Network
                      (IP, ICMP, ARP)
2. Data Link    ┐    Data Link
1. Physical     ┘    Physical
```

The OSI model is a **reference**; TCP/IP is what's actually implemented.

---

## 18. Types of Addresses (📑 Slides 83–93)

There are **four** kinds of address — one for each major layer:

| Address Type | Layer | Example | Used By |
|---|---|---|---|
| **Physical (MAC)** | Data Link (2) | `07:01:02:01:2C:4B` (48 bits) | NIC, switches |
| **Logical (IP)** | Network (3) | `192.168.1.1` (32 bits) | Routers |
| **Port** | Transport (4) | `80` (16 bits) | TCP/UDP, processes |
| **Specific** | Application (7) | `user@example.com`, URLs | Apps |

### Key fact (📑 Slide 92)

> **Physical addresses change hop-by-hop. Logical and port addresses stay the same end-to-end.**

This is the same insight as Chapter 3 (MAC vs IP routing across LANs).

### MAC address structure

```
   ┌─────────────────────────────────────┐
   │  MAC address (48 bits)              │
   ├──────────────┬──────────────────────┤
   │ OUI (24 bits)│ Vendor-assigned (24) │
   │ (manufacturer│ (unique per device)  │
   │ ID by IEEE)  │                      │
   └──────────────┴──────────────────────┘
```

### Port Number Facts
- 16-bit number (0 – 65535)
- Ports 0–1024 are **reserved** for system services (HTTP=80, HTTPS=443, FTP=21, SMTP=25)

---

## 19. Network Performance — Key Metrics (📑 Slides 94–103)

The most-tested topic in this chapter. Understand each metric clearly.

### 19a. Bandwidth vs Throughput (📑 Slide 94)

| | Definition | Analogy |
|---|---|---|
| **Bandwidth** | Maximum bits per second the link CAN carry | Width of a highway |
| **Throughput** | Actual bits per second delivered | Number of cars actually flowing |

**Throughput is always ≤ bandwidth** (often much less due to congestion, overhead, errors).

### 19b. Latency / Delay (📑 Slide 101)

**Latency = total time from sending bit to receiving it.**

$$\boxed{\text{Latency} = T_{prop} + T_{trans} + T_{queue} + T_{proc}}$$

Where:
- $T_{prop}$ = Propagation delay (physical travel)
- $T_{trans}$ = Transmission delay (pushing bits onto wire)
- $T_{queue}$ = Queuing delay (waiting in router buffer)
- $T_{proc}$ = Processing delay (router decisions)

### 19c. The Four Delay Types — Defined

#### 1. Transmission Delay (📑 Slide 99)

Time to **push all bits onto the link**.

$$\boxed{T_{trans} = \frac{N \text{ (number of bits)}}{S \text{ (bandwidth)}}}$$

**Depends on:** packet size + link bandwidth. **NOT** on distance.

#### 2. Propagation Delay (📑 Slide 100)

Time for a bit to **physically travel** from sender to receiver.

$$\boxed{T_{prop} = \frac{d \text{ (distance)}}{s \text{ (signal speed)}}}$$

**Depends on:** distance + medium. **NOT** on bandwidth or packet size.

Typical signal speeds:
- Vacuum: $3 \times 10^8$ m/s (speed of light)
- Cable: $2.3 \times 10^8$ m/s
- Fiber: $2.0 \times 10^8$ m/s

#### 3. Processing Delay (📑 Slide 102)

Time for a router/switch to:
- Read header
- Look up routing table
- Verify checksums

Usually small (microseconds), but adds up over many hops.

#### 4. Queuing Delay (📑 Slide 103)

Time a packet **waits in a router's queue** before being transmitted.

Highly variable — depends on **traffic load**:
- Light traffic → ~0
- Heavy traffic → can be huge (or packets dropped)

### Visual: where each delay happens

```
Source ─[T_trans]─► [Wire: T_prop] ─► Router ─[T_proc + T_queue]─► [Wire: T_prop] ─► Dest
        push bits     bits travel       routing decision           travel again
```

### Latency = Bandwidth tradeoff insight

- **Short message + high bandwidth** → propagation delay dominates (e.g., short email over fiber)
- **Long message + low bandwidth** → transmission delay dominates (e.g., big file over slow link)

---

# 20. NUMERICAL PROBLEMS — FULLY SOLVED

This is the highest-yield exam material. Master these patterns.

---

### 🧮 Problem 1: Throughput from Frame Rate (📑 Slide 95)

> Network bandwidth = 10 Mbps. Passes 12,000 frames/min, each frame = 10,000 bits. Find throughput.

**Step 1:** Convert to bits/second
$$\text{Throughput} = \frac{12{,}000 \text{ frames} \times 10{,}000 \text{ bits/frame}}{60 \text{ seconds}}$$

**Step 2:** Compute
$$= \frac{1.2 \times 10^8}{60} = 2 \times 10^6 \text{ bps} = 2 \text{ Mbps}$$

✅ **Answer: 2 Mbps** (only 1/5 of bandwidth — typical real-world inefficiency)

---

### 🧮 Problem 2: Propagation Time Across Atlantic (📑 Slide 104)

> Distance = 12,000 km, signal speed = 2.4 × 10⁸ m/s. Find propagation time.

**Step 1:** Convert distance to meters
$$d = 12{,}000 \text{ km} = 12{,}000 \times 1000 \text{ m} = 1.2 \times 10^7 \text{ m}$$

**Step 2:** Apply formula
$$T_{prop} = \frac{d}{s} = \frac{1.2 \times 10^7}{2.4 \times 10^8} = 0.05 \text{ s} = 50 \text{ ms}$$

✅ **Answer: 50 ms**

---

### 🧮 Problem 3: Short message over fast link (📑 Slide 106)

> 2500-byte email, BW = 1 Gbps, distance = 12,000 km, speed = 2.4 × 10⁸ m/s. Find both delays.

**Propagation time** (same as Problem 2):
$$T_{prop} = 50 \text{ ms}$$

**Transmission time:**
$$T_{trans} = \frac{2500 \times 8 \text{ bits}}{10^9 \text{ bps}} = \frac{20{,}000}{10^9} = 0.020 \text{ ms} = 20\,\mu s$$

✅ **Answer:** Propagation = **50 ms**, Transmission = **0.020 ms**

📌 **Key insight:** Propagation **dominates** here. Transmission is negligible. (Short message + huge bandwidth.)

---

### 🧮 Problem 4: Long message over slow link (📑 Slide 108)

> 5,000,000-byte image, BW = 1 Mbps, distance = 12,000 km. Find both delays.

**Propagation time** (same):
$$T_{prop} = 50 \text{ ms}$$

**Transmission time:**
$$T_{trans} = \frac{5{,}000{,}000 \times 8}{10^6} = \frac{4 \times 10^7}{10^6} = 40 \text{ s}$$

✅ **Answer:** Propagation = **50 ms**, Transmission = **40 s**

📌 **Key insight:** Transmission **dominates**. Propagation is negligible. (Long message + slow bandwidth.)

> **Compare with Problem 3:** Same distance, same propagation. But the dominant delay flips depending on message size and bandwidth.

---

### 🧮 Problem 5: Circuit-Switched File Transfer (📑 Slides 110–111)

> 640,000-bit file, all links 1.536 Mbps, TDM with 24 slots/sec, 500 ms to set up the circuit.

**Step 1:** Each circuit's bandwidth (one TDM slot)
$$\text{Bandwidth per slot} = \frac{1.536 \text{ Mbps}}{24} = 64 \text{ Kbps}$$

**Step 2:** Transmission time
$$T_{trans} = \frac{640{,}000 \text{ bits}}{64{,}000 \text{ bps}} = 10 \text{ s}$$

**Step 3:** Add setup time
$$\text{Total} = 500 \text{ ms} + 10 \text{ s} = 10.5 \text{ s}$$

✅ **Answer: 10.5 seconds**

📌 **Key insight:** In circuit switching, transmission time is **independent of the number of links** (because the circuit reserves resources end-to-end).

---

### 🧮 Problem 6: Packet Switching with Continuous Send (📑 Slides 112–113, part a)

> 1.5-MB file, RTT = 80 ms, packet size = 1 KB, 2×RTT handshake first, BW = 10 Mbps, packets sent **continuously**.

**Step 1:** Initial handshake time
$$T_{handshake} = 2 \times 80 = 160 \text{ ms}$$

**Step 2:** Network delay = propagation + transmission
- Propagation = RTT/2 = **40 ms**
- File size in bits = $1.5 \times 1{,}048{,}576 \times 8 = 12{,}582{,}912$ bits
- Transmission = $\frac{12{,}582{,}912}{10^7} = 1.258 \text{ s} \approx 1260 \text{ ms}$

$$T_{network} = 40 + 1260 = 1300 \text{ ms}$$

**Step 3:** Total
$$T_{total} = 160 + 1300 = 1460 \text{ ms} = \mathbf{1.46 \text{ s}}$$

✅ **Answer: 1.46 s**

---

### 🧮 Problem 7: Packet Switching with RTT-per-packet (📑 Slides 112–113, part b)

> Same file (1.5 MB, 1-KB packets), RTT = 80 ms, BW = **100 Mbps**, but **wait one RTT after each packet**.

**Step 1:** Number of packets
$$N = \frac{1.5 \times 1{,}048{,}576}{1024} = 1536 \text{ packets}$$

**Step 2:** Network delay (transmission at 100 Mbps + propagation)
- Transmission = $\frac{12{,}582{,}912}{10^8} = 0.126 \text{ s} = 126 \text{ ms}$
- Propagation = 40 ms
- Network delay = 166 ms

**Step 3:** Add (N−1) RTT waits
$$T_{waits} = (1536 - 1) \times 80 = 1535 \times 80 = 122{,}800 \text{ ms}$$

**Step 4:** Total
$$T_{total} = 160 + 166 + 122{,}800 = 123{,}126 \text{ ms} \approx \mathbf{123.1 \text{ s}}$$

✅ **Answer: ~123 s**

📌 **Lesson:** Even with 10× the bandwidth, the per-packet RTT wait makes it **84× SLOWER** than continuous send. **Stop-and-wait protocols are killers.**

---

### 🧮 Problem 8: Another File Transfer (📑 Slides 115–116)

> 1000-KB file, RTT = 100 ms, packet = 1 KB, 2×RTT handshake, BW = 1.5 Mbps.

**Part (a) — Continuous send:**

- Handshake = 200 ms
- File bits = 1000 × 1024 × 8 = 8,192,000 bits
- Transmission = 8,192,000 / 1,500,000 ≈ 5460 ms ≈ 5500 ms (slide rounds)
- Propagation = 50 ms

$$T_{total} \approx 200 + 5500 + 200 = 5700 \text{ ms} = \mathbf{5.7 \text{ s}}$$

**Part (b) — One RTT after each packet:**

- Number of packets = 1000
- Extra waits = 999 × 100 = 99,900 ms ≈ 100,000 ms
- $T_{total}$ = 200 + 5700 + (1000−1) × 100 = **105.8 s**

✅ **Answers:** (a) 5.7 s, (b) 105.8 s

---

### 🧮 Problem 9: Transmission Delay Quick Calculation

> Send 1024 bits over Fast Ethernet (100 Mbps). Find transmission delay.

$$T_{trans} = \frac{1024}{10^8} = 10.24 \times 10^{-6} \text{ s} = \mathbf{10.24\,\mu s}$$

(Slide says 1.024 µs, but the math gives 10.24 µs — be careful with the 100 Mbps = 10⁸ conversion.)

---

# 21. EXAM CHEAT SHEET

```
┌─────────────────────────────────────────────────────────────┐
│  NETWORK BASICS                                              │
│  • Network = nodes + links                                   │
│  • Criteria: Fault tolerance, Scalability, QoS, Security     │
│  • Data flow: Simplex, Half-duplex, Full-duplex              │
│  • Connections: Point-to-point, Multipoint                   │
├─────────────────────────────────────────────────────────────┤
│  TOPOLOGIES                                                  │
│  • Bus: single cable, terminators, cheap, fragile            │
│  • Star: central hub, popular, hub = SPOF                    │
│  • Ring: token passing, one direction, expensive             │
│  • Mesh: full connectivity, n(n−1)/2 links, only backbone    │
├─────────────────────────────────────────────────────────────┤
│  NETWORK CATEGORIES                                          │
│  • LAN (small) → MAN (city) → WAN (Internet)                 │
├─────────────────────────────────────────────────────────────┤
│  INTERNET STRUCTURE                                          │
│  • Edge (hosts) → Access nets → Core (routers)               │
│  • Circuit switching: dedicated, wasteful (telephone)        │
│  • Packet switching: shared, efficient (Internet)            │
├─────────────────────────────────────────────────────────────┤
│  OSI 7 LAYERS (memorize order!)                              │
│  7 Application  | Data    | HTTP, FTP, SMTP                  │
│  6 Presentation | Data    | SSL, TLS, encryption             │
│  5 Session      | Data    | Dialog control                   │
│  4 Transport    | Segment | TCP, UDP                          │
│  3 Network      | Packet  | IP, routing                      │
│  2 Data Link    | Frame   | Ethernet, MAC                    │
│  1 Physical     | Bits    | Cable, signals                   │
│  Mnemonic: All People Seem To Need Data Processing           │
├─────────────────────────────────────────────────────────────┤
│  ADDRESSES (one per layer)                                   │
│  • Physical (MAC, 48 bit, L2)  — changes hop-to-hop          │
│  • Logical (IP, 32 bit, L3)    — stays end-to-end            │
│  • Port (16 bit, L4)           — stays end-to-end            │
│  • Specific (URL, email, L7)                                 │
├─────────────────────────────────────────────────────────────┤
│  PERFORMANCE FORMULAS                                        │
│  • Transmission delay:  T_trans = N / Bandwidth              │
│  • Propagation delay:   T_prop  = distance / speed           │
│  • Total latency = T_prop + T_trans + T_queue + T_proc       │
│                                                              │
│  • Bandwidth ≠ Throughput (throughput ≤ bandwidth)           │
│  • Short msg + fast link → propagation dominates             │
│  • Long msg + slow link → transmission dominates             │
├─────────────────────────────────────────────────────────────┤
│  KEY UNITS                                                   │
│  • 1 KB = 1024 B = 8192 bits                                 │
│  • 1 MB = 1024 × 1024 B                                      │
│  • 1 Mbps = 10⁶ bps;  1 Gbps = 10⁹ bps                      │
│  • Speed in cable ≈ 2 × 10⁸ m/s                             │
└─────────────────────────────────────────────────────────────┘
```

---

# 22. Study Plan

1. **Day 1:** Sections 0–11 (basics, topologies, internet structure). Memorize the 4 topologies' pros/cons.
2. **Day 2:** Sections 12–18 (OSI model + addresses). MEMORIZE the 7 layers in order with their PDUs and protocols.
3. **Day 3:** Section 19 (performance metrics) + solve Problems 1–4 by hand.
4. **Day 4:** Solve Problems 5–9 (the bigger numericals). Glance at cheat sheet.

---

## 23. The "MUST KNOW" Items for Exam

If you only memorize 5 things from this chapter:

1. ⭐ **The 7 OSI layers in order** (with mnemonic)
2. ⭐ **The PDU at each layer** (Bits → Frame → Packet → Segment → Data)
3. ⭐ **Transmission delay = N/B**, **Propagation delay = d/s** (and *what each depends on*)
4. ⭐ **MAC changes hop-by-hop, IP/Port stay end-to-end**
5. ⭐ **Circuit switching vs Packet switching** trade-off

If anything is fuzzy, ask:
- "Walk me through OSI encapsulation step by step"
- "Make me 5 more delay calculation problems"
- "Explain TDM vs FDM with a diagram"
- "Quiz me on which layer does what"
