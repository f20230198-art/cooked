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

**Wireless problem:** can't detect collisions while transmitting (your own signal drowns out everything else). So we *avoid* instead of *detect*.

### Three Avoidance Tools
1. **IFS** (Inter-Frame Space) — fixed wait after sensing idle
2. **Contention Window** — random wait, doubles each retry (binary exponential)
3. **ACK** — receiver confirms; no ACK = assume collision

### Diagram

```
Sender:  ─[sense idle]─[wait IFS]─[wait random slots]─[SEND]─[wait ACK]─
                                       ↑
                          contention window (random)
```

---

## 8. DCF — Distributed Coordination Function (📑 Slides 41–42: "DCF Basic mode")

WiFi MAC layer. Uses CSMA/CA + ACKs. No collision detection (wireless), so adds delays as a "priority" scheme.

```
Sender:  ─[DIFS]─[Backoff]─[───DATA───]──────────[ACK?]─
Receiver: ─────────────────────[receives]─[SIFS]─[ACK]─
Others:  ─────────────────────────────────[NAV: stay quiet]─
```

- **SIFS** (Short IFS) — used for ACK (highest priority)
- **DIFS** (Distributed IFS) — used before sending data (lower priority)

---

## 9. Hidden & Exposed Terminal Problems (📑 Slides 44–45)

Wireless-only weirdness. Range is limited, so two nodes might not hear each other but their signals collide at a third.

### Hidden Terminal Problem

```
       A's range            C's range
      ◯─────◯─────◯       ◯─────◯─────◯
      │    A's    │       │    C's    │
      │   reach   │       │   reach   │
      └───────────┘       └───────────┘

   A ────────── B ────────── C
              (both can hear B,
               but A and C can't hear each other)

A → B (transmitting)
C senses idle (can't hear A) → also sends to B
   ↓
COLLISION at B! (C is "hidden" from A)
```

### Exposed Terminal Problem

```
   A ────── B ────── C ────── D

B → A (transmitting)
C wants to send to D (D can't hear B at all → would be safe)
But C senses B's signal → defers unnecessarily
   ↓
WASTED OPPORTUNITY (C is "exposed" to B)
```

---

## 10. MACA / RTS-CTS Solution (📑 Slides 46–49)

Tiny handshake before data:

```
Sender ──RTS (Request To Send)──→ Receiver
              [I want to send X bytes for Y duration]

Receiver ──CTS (Clear To Send)──→ Sender + everyone nearby
              [OK, everyone hearing this: shut up for Y duration]

Sender ──────DATA──────→ Receiver
Receiver ───ACK────→ Sender
```

Everyone who hears the CTS sets their **NAV** (Network Allocation Vector) timer = "do not transmit until time X."

### How it solves both problems

- **Hidden terminal:** C hears CTS from B → C stays quiet during A→B transmission ✓
- **Exposed terminal:** C only hears RTS from B (not CTS from A on the other side) → C knows it's safe to send ✓

---

## 11. Controlled Access Protocols (📑 Slides 50–55)

### Reservation (📑 Slide 50)

```
| Reservation mini-frame | Data Slot 1 | Data Slot 2 | ... |
| 1 0 1 0 1 (5 bits)     | Stn1 sends  | Stn3 sends  | ... |
  ↑   ↑   ↑
  Stn1 Stn3 Stn5 want to send (set their bit)
```

Stations book a slot first, then transmit conflict-free.

### Polling (📑 Slides 51–52)

```
   Primary
  ┌────────┐
  │ Master │ ──poll──→ Secondary 1: "Got data?" → reply
  └────────┘ ──poll──→ Secondary 2: "Got data?" → reply
              ──poll──→ Secondary 3: "Got data?" → reply
```

- **Select**: primary wants to send TO a secondary
- **Poll**: primary asks IF a secondary wants to send
- ⚠ Primary = single point of failure

### Token Passing (📑 Slides 53–55)

```
        ┌─────[Token]─────┐
        ↓                 │
       Stn1              Stn4
        │                 ↑
        ↓                 │
       Stn2 ──────────→ Stn3
```

Whoever holds the token may transmit. Then passes it on. Logical ring.

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
