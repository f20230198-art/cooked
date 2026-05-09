# Multiple Access Protocols — Full Study Guide

> **How to use this:** Each section says which slide(s) it covers. Keep your PPT open beside this file. Diagrams are in ASCII — read them in monospace.

---

## 0. The Big Picture (📑 Slide 1: "Multiple Access Mechanisms")

One shared channel (wire or radio), many computers. Only one can talk at a time, otherwise → **collision**.

```
            Multiple Access Protocols
                     |
     ┌───────────────┼────────────────┐
     |               |                |
 Random Access   Controlled Access  Channelization
 (talk anytime,  (take turns in     (split channel
  fix collisions  organized way)     itself)
  later)
     |               |                |
 ALOHA           Reservation       FDMA (frequency)
 Slotted ALOHA   Polling           TDMA (time)
 CSMA            Token Passing     CDMA (code)
 CSMA/CD
 CSMA/CA
```

Your slides focus mostly on **Random Access** + a bit of Controlled Access at the end.

---

## 1. Random Access — Core Idea (📑 Slides 2–3: "Random Access Protocols")

- No master. Every node decides on its own when to send.
- Two senders at the same moment → **collision** → garbage → retransmit.
- The whole game = "minimize collisions" + "recover from them."

**The progression (each protocol fixes the previous one's flaw):**

```
Pure ALOHA  →  Slotted ALOHA  →  CSMA  →  CSMA/CD  →  CSMA/CA
(talk        (talk only         (listen   (listen      (wireless:
 anytime)     at slot start)     first)    while        avoid
                                           talking      collisions)
                                           too)
```

---

## 2. Key Ideas of Random Access (📑 Slides 4–5: "key ideas of random access")

- **Stations** — independent, equal-priority nodes
- **Channel** — single, shared, everyone hears everyone
- **Time** — continuous (Pure ALOHA, CSMA) OR sliced into equal **slots** (Slotted ALOHA)
- **Carrier Sense** — listen before talking
- **Collision Detection** — listen *while* talking, stop the moment overlap is detected
- **Randomness** — after a collision, wait a *random* time before retrying (otherwise both retry simultaneously and collide again)

---

## 3. Pure ALOHA (📑 Slides 6–9: "ALOHA Network", "Frames in Pure ALOHA", "ALOHA Protocol")

**Rule:** Whenever you have data → just send. If no ACK comes → assume collision → wait random time → resend.

### Frames in Pure ALOHA — diagram

```
Station 1: ──[Frame1.1]──────────[Frame1.2]──────────────→ time
Station 2: ────────[Frame2.1]──────────[Frame2.2]────────→
Station 3: ──[Frame3.1]──────[Frame3.2]──────────────────→
Station 4: ────[Frame4.1]──────────[Frame4.2]────────────→
                  ↑↑                  ↑↑
            COLLISION!           COLLISION!
        (frames overlap in time, both destroyed)
```

### Vulnerable Time (📑 Slide 10: "ALOHA: Vulnerable Time")

A frame collides if **any other** transmission starts within one frame-time *before* OR one frame-time *after* it begins.

```
       ← T_fr →    ← T_fr →
    ──────────|──────────|──────── time
              ↑
        my frame starts here
   ◄═════ vulnerable window ═════►
        2 × T_fr total
```

**Vulnerable time = 2 · T_fr**

### Throughput (📑 Slide 11: "ALOHA: Throughput")

$$S = G \cdot e^{-2G}$$

- G = average frames *attempted* per frame-time
- S = average frames *successfully delivered*
- **Max throughput = 18.4% at G = 0.5** (memorize)

---

## 4. Slotted ALOHA (📑 Slides 12–14: "Slotted ALOHA", "Vulnerable Time", "Throughput")

**Fix:** force everyone to start *only* at slot boundaries. Time = quantized.

### Diagram

```
slot:    | 1 | 2 | 3 | 4 | 5 | 6 |
S1:      |F11|   |   |XXX|   |F12|     ← X = collision
S2:      |   |F21|   |XXX|F22|   |
S3:      |F31|   |F32|   |   |   |
                  ↑       ↑
              slot 4 has 2 senders → COLLISION
```

Frames either align perfectly (no overlap) or fully collide — no partial overlaps.

### Vulnerable Time
**Cut in half: T_fr** (only the current slot is dangerous, not the previous one)

### Throughput
$$S = G \cdot e^{-G}$$

**Max throughput = 36.8% at G = 1**

### 🎯 MEMORIZE — Pure vs Slotted (📑 Slide 15)

| | Max Throughput | At G = |
|---|---|---|
| Pure ALOHA | 18.4% | 0.5 |
| Slotted ALOHA | 36.8% | 1 |

Slotted is **2× better**. Still wastes ~63% of channel.

---

## 5. CSMA — Carrier Sense Multiple Access (📑 Slides 16–17: "CSMA")

**Big idea:** "Listen before talking." Drastically reduces collisions.

But collisions still happen because of **propagation delay**:

```
Station A ──────────────── Station B
 (sends at t=0)          (senses idle at t=0,
                          A's signal hasn't arrived yet,
                          starts sending at t=0.5×T_p)
 t=T_p: A's signal reaches B → COLLISION at B
```

### Assumptions (📑 Slide 18)
- Constant frame length
- No errors except collisions
- Every station hears every transmission
- Propagation delay << transmission time

### Vulnerable Time (📑 Slide 20: "CSMA: Vulnerable Time")

```
A sends ──┐
          ▼
   ─────[ Frame ]─────────── time
   ◄T_p► (only this small window is dangerous)
```

**Vulnerable time = T_p (propagation time)** — way smaller than ALOHA's 2·T_fr.

### Persistence Methods (📑 Slides 21–24)

What to do when channel is idle/busy?

#### (a) 1-Persistent (📑 Slide 22)
- Busy → keep listening
- Becomes idle → send **immediately** (probability 1)
- ⚠ If many stations are waiting, all jump → guaranteed collision
- Used in classic Ethernet

```
Busy.....Busy.....Busy.....IDLE!
                              ↓
                     Send immediately
```

#### (b) Non-Persistent (📑 Slide 23)
- Busy → wait *random* time, then sense again
- Idle → send immediately
- Fewer collisions, but wastes idle time

```
Busy → wait random → sense → if idle, send; if busy, wait random again
```

#### (c) P-Persistent (📑 Slides 24–25)
- For slotted channels
- Idle → send with probability *p*; with probability (1-p) wait one slot
- Best of both worlds

```
IDLE slot:
  flip biased coin (prob p)
  ├─ heads → send
  └─ tails → wait one slot, retry
```

---

## 6. CSMA/CD — Collision Detection (📑 Slides 27–32: "CSMA/CD Protocol", "Network Size Restriction")

**Problem with plain CSMA:** if collision happens, both senders waste the *entire* frame time before noticing.

**Fix:** *listen while you talk*. Detect collision instantly → STOP, send a **jam signal**, wait **random backoff**.

### Procedure (📑 Slide 28)
```
1. Sense channel
2. If idle → start transmitting
3. While transmitting, KEEP LISTENING
4. Collision? → STOP + send jam (48 bits) + binary exponential backoff
5. Go to step 1
```

### Collision Detection Window — diagram (📑 Slide 29)

```
A ──[start sending]──────────[detects collision]──── stop
              \              /
               \            /  ← signals collide
                \          /     somewhere in middle
                 \        /
B ──────────[start sending]──[detects collision]──── stop

Time for A to know: up to 2 × T_p (round trip)
```

### 🎯 Minimum Frame Size (📑 Slides 31–32) — VERY IMPORTANT

For CSMA/CD to work, A must still be transmitting when collision news returns:

$$T_{fr} \geq 2 \cdot T_p$$

So:
$$\boxed{\text{Min frame size (bits)} = 2 \cdot T_p \cdot \text{Bandwidth}}$$

This is why Ethernet has a **64-byte (512-bit)** minimum frame.

### Energy Levels Diagram (📑 Slide 35)

```
Energy
  │       ┌──────[Collision spike]──────┐
  │       │                              │
  │  ┌────┘                              └────┐
  │  │ frame transmitting           idle       frame
  └──┴───────────────────────────────────────────────→ time
```

Collision = abnormally high energy → station knows.

### Efficiency Formula (📑 Slides 36–38)

$$\text{Efficiency} = \frac{1}{1 + 6.44 \cdot a}$$

where $a = T_p / T_{fr}$.

- Small `a` (short cable, big frames) → high efficiency
- Big `a` (long cable, small frames) → low efficiency

---

## 7. CSMA/CA — Collision Avoidance (📑 Slide 40: "CSMA/CA")

### Why a new protocol for wireless?

In wired Ethernet, **CSMA/CD works** because a station can listen *while* it transmits and instantly detect that two signals are colliding (the voltage on the wire spikes).

In **wireless**, this doesn't work. Two big reasons:

1. **Your own signal is millions of times stronger than any incoming signal at your antenna.** It's like trying to hear someone whisper while you're shouting into a megaphone — you can't. So you can't detect collisions while transmitting.
2. **A collision happens at the receiver, not at the sender.** Two transmissions might overlap perfectly fine at the sender's location but collide at some other node.

So wireless can't use "Collision Detection." Instead, it tries to **Avoid** collisions in the first place — that's what the **CA** stands for.

### How does CSMA/CA avoid collisions?

It uses **three tools together**:

#### Tool 1 — IFS (Inter-Frame Space)
After sensing the channel idle, **don't send immediately**. Wait a small fixed time first.

Why? Because a faraway transmission might still be on its way to you. Waiting gives it time to arrive. Different message types use different IFS lengths to create *priority*:
- **SIFS** (Short IFS) — highest priority, used for ACKs
- **DIFS** (Distributed IFS) — lower priority, used before regular data

So an ACK always wins over a fresh data frame because SIFS < DIFS.

#### Tool 2 — Contention Window (random backoff)
After the IFS, still don't send! Pick a **random number of time slots** between 0 and CW (contention window size), then wait that many slots, sensing all the time.

Why? If two stations were both waiting on the same busy channel, they'd both finish IFS at the same instant. If they both transmitted right then → guaranteed collision. The random wait spreads them out.

If a collision still happens, **double CW** (binary exponential backoff: 7 → 15 → 31 → 63 → ...).

#### Tool 3 — ACK + Timeout
The receiver must send an ACK back. If the sender doesn't get an ACK within a timeout → assume collision → retry.

This is needed because the sender can't *see* the collision (Tool 1's reason). The only proof that a frame got through is an ACK.

### Full CSMA/CA Timing Diagram

```
Sender:    [sense channel]
                │
                ▼
           [channel busy?]──Yes──┐
                │ No              │
                ▼                 │
           [wait DIFS]            │
                │                 │
                ▼                 │
           [pick random N slots]  │
                │                 │
                ▼                 │
           [count down N,         │
            sensing each slot]    │
                │                 │
                ▼                 │
           [N=0? → SEND DATA] ◄───┘ (if channel becomes busy
                │                    again, freeze countdown
                ▼                    and resume after it's free)
           [wait for ACK]
                │
                ▼
           [no ACK? → double CW, retry]
```

**The whole point:** all of this delay machinery exists to *prevent* two stations from ever transmitting at the same instant.

---

## 8. DCF — Distributed Coordination Function (📑 Slides 41–42)

DCF is just the **official name** for the CSMA/CA mode in WiFi (802.11). Same protocol, fancy name. "Distributed" because there's no master station — every WiFi device runs DCF on its own.

### DCF Basic Mode — annotated timing

```
Time → → → → → → → → → → → → → → → → → → → → → → → → → →

Sender:    [DIFS]─[Random Backoff]─[──── DATA ────]──────────[ next... ]
                                                    │
Receiver:  ─────────────────────────────[receives]─[SIFS]─[ACK]
                                                              │
Other      ─────────────────────────────────────────[NAV: be quiet]
stations:                                            for the duration
                                                     announced in DATA header
```

What's happening, step by step:

1. **DIFS** — sender waits the standard inter-frame space after channel becomes idle
2. **Random Backoff** — sender counts down a random number of slots (collision avoidance)
3. **DATA** — sender transmits its frame; the frame header includes a "duration" field telling everyone how long it'll take
4. **Other stations** read that duration field → set their **NAV** timer → stay silent
5. **SIFS** (shorter than DIFS) — receiver waits this brief gap, then…
6. **ACK** — receiver sends ACK. Because SIFS < DIFS, no other station can grab the channel before the ACK goes out
7. After ACK, channel is free; the next round of DIFS + backoff begins

**Key insight:** SIFS is shorter than DIFS so that ACKs always have priority. This guarantees the data/ACK exchange finishes atomically without interruption.

---

## 9. Hidden & Exposed Terminal Problems (📑 Slides 44–45)

These are **only wireless problems**. They happen because radios have limited range — not every node can hear every other node. This breaks the basic CSMA assumption ("if I sense idle, the channel really is idle").

### 9a. Hidden Terminal Problem

**Setup:** Three stations A, B, C in a line. A and C can both reach B, but A and C are too far apart to hear each other.

```
   A's radio range            C's radio range
   ╭─────────────╮            ╭─────────────╮
   │             │            │             │
   │   A ······· B ········ C    │
   │             │            │             │
   ╰─────────────╯            ╰─────────────╯
                ↑
        B is in BOTH ranges,
        but A and C are NOT in each other's range
```

**What goes wrong:**

```
Time 1:  A starts transmitting to B
         A ──────────► B
         (C cannot hear A — A is "hidden" from C)

Time 2:  C wants to transmit to B
         C senses the channel → "Sounds idle to me!"  (because C can't hear A)
         C ──────────► B

Time 3:  At B, A's signal and C's signal arrive together
         → COLLISION at B
         → B receives garbage from both
```

**Why it's called "hidden":** A is a hidden transmitter from C's perspective. C can't sense A's transmission, so C wrongly believes the channel is free.

**Consequence:** carrier sensing alone is not enough in wireless.

### 9b. Exposed Terminal Problem

**Setup:** Four stations A, B, C, D in a line. B and C can hear each other. But A is only in B's range, and D is only in C's range.

```
   A's range    B's range       C's range    D's range
   ╭──────╮    ╭──────╮         ╭──────╮    ╭──────╮
   │  A───┼────┼──B   │         │   C──┼────┼───D  │
   ╰──────╯    ╰──────╯         ╰──────╯    ╰──────╯
                    ↑               ↑
              B and C can hear each other,
              but A cannot hear C, and D cannot hear B
```

**What goes wrong:**

```
Time 1:  B is transmitting to A
         B ──────► A
         (C can hear B, but the data is going leftward to A, not toward C)

Time 2:  C wants to transmit to D (rightward)
         C senses channel → "Busy! B is talking."
         C defers and stays silent.

Time 3:  But wait — D is far from B. D wouldn't have heard B at all.
         C → D would have been a perfectly safe parallel transmission.
         C unnecessarily wasted a transmission opportunity.
```

**Why it's called "exposed":** C is "exposed" to B's transmission (it can hear it), but that exposure shouldn't matter — C's intended receiver D is out of B's range.

**Consequence:** carrier sensing is *too cautious* in wireless — it kills throughput by stopping safe parallel transmissions.

### Summary

| Problem | Carrier sensing's mistake | Effect |
|---|---|---|
| Hidden terminal | Says "idle" when really busy at receiver | Causes collisions |
| Exposed terminal | Says "busy" when transmission would have been safe | Wastes bandwidth |

Both problems show that **simple "listen before talk" doesn't work in wireless**. We need something smarter → MACA / RTS-CTS.

---

## 10. MACA / RTS-CTS Solution (📑 Slides 46–49)

**MACA = Multiple Access with Collision Avoidance.** The trick: instead of sensing the channel, **explicitly negotiate** with the receiver before sending data. The negotiation is a tiny handshake using two control packets.

### The handshake

```
Step 1:  Sender ──RTS──► Receiver
         (RTS = Request To Send. Tiny packet.
          Contains: "I want to send to you. It will take T microseconds.")

Step 2:  Receiver ──CTS──► back to Sender (and everyone in the receiver's range)
         (CTS = Clear To Send. Also tiny.
          Contains: "OK go ahead. Everyone else who hears this: shut up for T microseconds.")

Step 3:  Sender ──── DATA ────► Receiver
         (Now the actual data flows, knowing nobody near the receiver will interfere.)

Step 4:  Receiver ──ACK──► Sender
         ("Got it.")
```

The magic: any node that hears either the RTS or the CTS sets its **NAV** (Network Allocation Vector) — an internal "do not transmit until time X" timer based on the duration field in the packet.

### How RTS/CTS solves the Hidden Terminal Problem

Same A–B–C scenario from earlier:

```
A wants to send to B:
   A ──RTS──► B          (C can't hear this RTS — A is hidden from C)
   B ──CTS──► A and C    (CTS goes to everyone in B's range, INCLUDING C!)
                         (C now sees CTS → sets NAV → stays quiet)
   A ──DATA──► B         (no collision, because C is silent)
```

✅ **Solved!** Even though C can't hear A directly, C hears B's CTS, which warns C off.

### How RTS/CTS solves the Exposed Terminal Problem

Same A–B–C–D scenario:

```
B wants to send to A (leftward):
   B ──RTS──► A and C    (C hears B's RTS — knows B will transmit)
   A ──CTS──► B          (CTS goes to A's range only — C does NOT hear this)
   B ──DATA──► A

C wants to send to D (rightward):
   C heard the RTS from B but NOT a CTS.
   Rule: only defer if you hear a CTS, not an RTS.
   So C is free to proceed:
   C ──RTS──► D
   D ──CTS──► C
   C ──DATA──► D         (parallel transmission — works fine!)
```

✅ **Solved!** Hearing only an RTS (no CTS) means "the receiver is somewhere I can't reach, so my transmission won't interfere there."

### Quick mental model

- **Hear a CTS** = "the receiver is near me. Stay quiet."
- **Hear an RTS but no CTS** = "the sender is near me but the receiver is far away. I can transmit to my own faraway partner safely."

This is why CSMA/CA in WiFi has an *optional* RTS/CTS mode — it's expensive (two extra packets per data exchange) but essential when hidden terminals are common.

---

## 11. Controlled Access Protocols (📑 Slides 50–55)

So far every protocol has been "random access" — stations decide on their own. **Controlled access** is the opposite: there's a coordination scheme so that only one station transmits at a time, by design. No collisions, ever.

Three main flavors:

### 11a. Reservation (📑 Slide 50)

**Idea:** before any data flows, every station declares "I want to send" or "I don't" in a tiny pre-frame. Then the actual data slots are allocated in order, conflict-free.

```
Time goes left to right →

| Reservation mini-frame |  Data slot  |  Data slot  |  Data slot  |
|  1   0   1   0   1     |   Stn1's    |   Stn3's    |   Stn5's    |
|  ↑       ↑       ↑     |    data     |    data     |    data     |
|  Stn1   Stn3    Stn5   |             |             |             |
|  raise their hand      |             |             |             |
```

How it works:
1. Time is divided into "intervals." Each interval starts with a tiny **reservation frame** containing N mini-slots (one per station)
2. To request a turn, a station sets its bit in its mini-slot to 1
3. After the reservation frame, stations whose bits were 1 transmit in order
4. Repeat

✅ Pro: no collisions
❌ Con: even idle stations consume reservation overhead

### 11b. Polling (📑 Slides 51–52)

**Idea:** one station is the boss (**primary**); the rest are workers (**secondaries**). Workers can only act when the boss talks to them.

```
        ┌──────────┐
        │ Primary  │  (the boss / master station)
        └────┬─────┘
             │ asks each one in turn
             │
   ┌─────────┼─────────┐
   ▼         ▼         ▼
 [Sec1]    [Sec2]    [Sec3]
```

Two types of primary→secondary interactions:

- **SELECT** — Primary has data **for** Secondary X. Sends a SEL frame: "Sec X, get ready." Then sends data.
- **POLL** — Primary asks Secondary X: "Got anything to send?"
  - If yes → secondary sends its data
  - If no → secondary sends a NAK; primary moves on to next

**Polling cycle:** Primary polls Sec1, then Sec2, then Sec3, ... loops forever.

✅ Pro: no collisions, simple
❌ Con: primary is a single point of failure; lots of "got anything?" overhead even when nobody has data

### 11c. Token Passing (📑 Slides 53–55)

**Idea:** a special tiny packet called a **token** circulates around the network. **Whoever holds the token has permission to transmit.** No token = no transmission.

Stations are arranged in a **logical ring** (the physical wiring can be anything):

```
       ┌─────► Stn1 ─────┐
       │                 │
       │   Token moves   ▼
      Stn4              Stn2
       ▲                 │
       │                 │
       └───── Stn3 ◄─────┘
```

How it works:

1. The token is created when the network starts
2. The token circulates around the ring continuously
3. When a station receives the token:
   - **Has data?** → keep the token, transmit one (or a few) frames, then release the token to the next station
   - **No data?** → immediately pass the token to the next station
4. Nobody can transmit without the token → no collisions, ever

**Token management challenges (worth knowing):**
- **Lost token** — the only token is destroyed. Nobody can transmit. Solution: monitor station detects no token for too long → generates a new one.
- **Duplicate token** — two tokens circulating. Risk of collisions. Solution: monitor station detects extras → removes them.
- **Priority** — multi-bit token can carry priority info so urgent stations get to transmit first.

✅ Pro: no collisions; fair access (everyone gets the token in turn); great under heavy load
❌ Con: complex to manage tokens; one bad node can break the ring; extra latency at low load (you wait for token even when nobody else wants to transmit)

### Comparison Table

| Method | Coordinator? | Collision-free? | Failure mode |
|---|---|---|---|
| Reservation | shared mini-frame protocol | yes | reservation slot loss desyncs everyone |
| Polling | centralized (primary) | yes | primary failure = whole network down |
| Token Passing | distributed (token) | yes | lost/duplicate token; need monitor station |

---

## 12. ALL FORMULAS — Quick Reference

| Protocol | Vulnerable Time | Throughput Formula | Max Throughput |
|---|---|---|---|
| Pure ALOHA | 2·T_fr | S = G·e^(-2G) | 18.4% (G=0.5) |
| Slotted ALOHA | T_fr | S = G·e^(-G) | 36.8% (G=1) |
| CSMA | T_p | depends on persistence | varies |
| CSMA/CD | T_p | 1/(1 + 6.44·a) | depends on `a` |

**Helper formulas:**
- Propagation time: $T_p = \text{distance} / \text{signal speed}$
- Transmission time: $T_{fr} = \text{frame size} / \text{bandwidth}$
- $a = T_p / T_{fr}$
- **Min CSMA/CD frame: $2 \cdot T_p \cdot \text{Bandwidth}$**

---

# 13. NUMERICAL PROBLEMS — FULLY SOLVED

### 🧮 Problem 1: CSMA/CD Min Frame Size (📑 Slide 39, Q1)

> **1 Gbps**, **1 km** cable, signal speed **200,000 km/s**. Minimum frame size?

**Step 1:** Propagation time
$$T_p = \frac{1 \text{ km}}{200{,}000 \text{ km/s}} = 5\,\mu s$$

**Step 2:** Min transmission time = round-trip
$$T_{fr,\min} = 2 \cdot T_p = 10\,\mu s$$

**Step 3:** Convert to bits
$$\text{Min frame} = 10 \times 10^{-6} \text{ s} \times 10^9 \text{ bps} = 10{,}000 \text{ bits}$$

✅ **Answer: 10,000 bits = 1250 bytes**

---

### 🧮 Problem 2: 10 Mbps Ethernet, 225-bit prop delay (📑 Slide 39, Q2)

> A and B on 10 Mbps Ethernet. Prop delay = 225 bit times. Both start at t=0. A's frame = 1000 bits. Frames collide. Jam = 48 bits.

```
t = 0      A starts sending  ──┐
                                │ 225 bit times
t = 0      B starts sending  ──┤ (signals propagating)
                                │
t = 225    B detects collision  (A's signal reached B)
t = 225    A detects collision  (B's signal reached A)
t = 273    A finishes jam (225 + 48)
```

**Key facts to remember:**
- Both detect collision at the **same time** = T_p = 225 bit times
- Jam takes 48 bits → finishes at t = 273

---

### 🧮 Problem 3: Token Ring Throughput (📑 Slide 56-ish)

> 10 Mbps token ring, 4 stations, 2 km between each, signal speed 2×10⁸ m/s, frame = 1 Kbit.

**Step 1:** One-hop propagation
$$T_p = \frac{2000}{2 \times 10^8} = 10\,\mu s$$

**Step 2:** Full ring latency (4 hops × 2 km = 8 km)
$$T_{ring} = \frac{8000}{2 \times 10^8} = 40\,\mu s$$

**Step 3:** Frame transmission time
$$T_{fr} = \frac{1000}{10 \times 10^6} = 100\,\mu s$$

**Step 4:** Cycle time (each of 4 stations sends one frame, plus token rotation)
$$T_{cycle} = 4 \times T_{fr} + T_{ring} = 400 + 40 = 440\,\mu s$$

**Step 5:** Per-station throughput
$$\text{Throughput} = \frac{1000 \text{ bits}}{440\,\mu s} \approx 2.27 \text{ Mbps}$$

---

### 🧮 Problem 4: 10 Mbps Ethernet with Backoff (📑 Slide 39, Q3)

> 5 nodes; prop delay = 325 bit times; frame = 1000 bits; CSMA/CD with binary exponential backoff. After 1st collision, A picks 0, B picks 1.

| Time (bit times) | Event |
|---|---|
| 0 | A and B start transmitting |
| 325 | Both detect collision |
| 325 + 48 = 373 | Jam signals end |
| 373 | A's backoff = 0 → tries immediately, but A senses busy because B's residue is still on wire — needs to wait until clear. In practice A senses idle just after 373 |
| ~373 → A retransmits 1000 bits | |
| ~1373 | A finishes |
| B's backoff = 1 slot = 1 × 2·T_p = 650 bit times → B tries at t = 1023 → senses busy (A is sending) → defers |

**Result:** A wins, B defers and retries after A finishes.

---

# 14. EXAM CHEAT SHEET (one-pager)

```
┌─────────────────────────────────────────────────────┐
│  ALOHA family                                        │
│  • Pure: vuln=2T_fr, S=Ge^-2G, max=18.4% (G=0.5)    │
│  • Slotted: vuln=T_fr, S=Ge^-G, max=36.8% (G=1)     │
├─────────────────────────────────────────────────────┤
│  CSMA — listen before talking. Vuln = T_p           │
│  • 1-persistent: send immediately (Ethernet)         │
│  • Non-persistent: random wait if busy               │
│  • P-persistent: send w/ prob p in each idle slot   │
├─────────────────────────────────────────────────────┤
│  CSMA/CD — listen WHILE talking                      │
│  • Min frame = 2·T_p·Bandwidth                       │
│  • Efficiency = 1/(1 + 6.44a), a = T_p/T_fr         │
│  • Jam signal = 48 bits                              │
│  • Ethernet min frame = 64 bytes = 512 bits          │
├─────────────────────────────────────────────────────┤
│  CSMA/CA (wireless)                                  │
│  • Can't detect collisions → IFS + window + ACK     │
│  • Hidden terminal → use RTS/CTS                     │
│  • NAV = "stay quiet until time X"                   │
├─────────────────────────────────────────────────────┤
│  Controlled Access                                   │
│  • Reservation: book a slot in mini-frame            │
│  • Polling: primary asks each secondary              │
│  • Token: hold token = right to send                 │
├─────────────────────────────────────────────────────┤
│  MAGIC NUMBERS                                       │
│  • Pure ALOHA max:      1/(2e) ≈ 18.4%              │
│  • Slotted ALOHA max:   1/e    ≈ 36.8%              │
│  • CSMA/CD factor:      6.44                         │
│  • Ethernet jam:        48 bits                      │
│  • Ethernet min frame:  512 bits (64 bytes)         │
└─────────────────────────────────────────────────────┘
```

---

# 15. Study Plan (4 days)

1. **Day 1 (today):** Read sections 0–4 with PPT open. Just *follow the story*.
2. **Day 2:** Read 5–11. Draw the diagrams yourself by hand (huge memory boost).
3. **Day 3:** Solve the 4 numericals in section 13 *without looking*. Then check.
4. **Day 4 (exam day):** Just glance at section 14. Done.

**If anything is fuzzy, ask me to:**
- "Expand backoff with a worked example"
- "Make 3 more practice numericals"
- "Draw the timing diagram for CSMA/CA + RTS-CTS"
- "Quiz me on this"
