# Tutorials 5 & 6 — ALOHA, CSMA/CD Combined Solutions

> **Topics:** ALOHA throughput, backoff time, CSMA/CD minimum frame size, contention slots, collision timing
> **Reference guide:** [multiple_access_protocols_guide.md](multiple_access_protocols_guide.md)
>
> **Tutorial 5** = Q1–Q6 (ALOHA backoff, throughput, CSMA/CD min frame)
> **Tutorial 6** = Q7–Q12 (CSMA/CD efficiency, contention slots, collision timing with backoff)

---

## 📚 Concepts You Need First

### Concept A — Binary Exponential Backoff
After the **K-th collision**, a station waits a random number of **slot times** chosen uniformly from `{0, 1, 2, ..., 2^K − 1}`.

$$T_B = R \times T_{slot}, \quad R \in \{0, 1, ..., 2^K - 1\}$$

For ALOHA: slot time = $2 \times T_p$ (round-trip propagation).
For CSMA/CD Ethernet: slot = 51.2 µs (standard).

### Concept B — Pure ALOHA Throughput
$$S = G \cdot e^{-2G}, \quad S_{max} = \frac{1}{2e} \approx 0.184 \text{ at } G = 0.5$$

### Concept C — Slotted ALOHA Throughput
$$S = G \cdot e^{-G}, \quad S_{max} = \frac{1}{e} \approx 0.368 \text{ at } G = 1$$

For both: throughput in **frames per second** = $S / T_{fr}$.

### Concept D — CSMA/CD Minimum Frame Size
For collision detection to work, the sender must still be transmitting when the collision signal returns:

$$T_{fr} \geq 2 \cdot T_p \quad \Rightarrow \quad L_{min} = 2 \cdot T_p \cdot R$$

### Concept E — Contention Slot
A "contention slot" in CSMA/CD = the **round-trip propagation time** = $2 \cdot T_p$. This is the worst-case time to detect a collision.

> **Jam signal:** a 48-bit garbage pattern an Ethernet station sends right after detecting a collision, to make sure every other station also sees the collision.
> **Repeater:** a Layer-1 device that regenerates a weak signal so cables can run longer; it adds a small processing delay (typically ~10 µs) but does NOT separate collision domains.
> **Slot time:** the unit of waiting in binary exponential backoff. For Ethernet, slot time = 51.2 µs = 512 bit-times at 10 Mbps.

### Concept F — CSMA/CD Efficiency
$$E = \frac{1}{1 + 5B}$$
where $B = t/D$ (avg propagation / avg transmission delay)

(Note: textbook often uses **5B** in the denominator; some use **6.44a** — check which formula your course wants. This tutorial uses 5B per the question.)

---

# 🧮 PROBLEMS

---

## Q1. (Tut 5) ALOHA Backoff Times

> ALOHA network, stations max **600 km** apart, signal speed = $3 \times 10^8$ m/s. Find possible backoff times $T_B$ for different attempt numbers K.

### Step 1 — Slot time = round-trip propagation
$$T_p = \frac{600 \times 1000}{3 \times 10^8} = 2 \times 10^{-3} \text{ s} = 2 \text{ ms}$$

$$T_{slot} = 2 \times T_p = 4 \text{ ms}$$

### Step 2 — Backoff for each K

After K collisions, R is randomly chosen from {0, 1, …, 2^K − 1}, then $T_B = R \times 4$ ms.

| K (attempt) | Range of R | Possible R values | Possible $T_B$ (ms) |
|---|---|---|---|
| 1 | 0 to 1 | 0, 1 | 0, 4 |
| 2 | 0 to 3 | 0, 1, 2, 3 | 0, 4, 8, 12 |
| 3 | 0 to 7 | 0, 1, …, 7 | 0, 4, 8, 12, 16, 20, 24, 28 |
| 4 | 0 to 15 | 0–15 | 0, 4, 8, …, 60 |

✅ **Pattern:** After K-th collision, $T_B \in \{0, 4, 8, ..., (2^K - 1) \times 4\}$ ms.

### 🔑 Insight
Backoff range **doubles** each time → spreads stations apart in time → reduces collision probability for retries.

---

## Q2. (Tut 5) Pure ALOHA — Collision-Free Requirement

> Pure ALOHA, **200-bit frames**, channel = **200 kbps**. What's required to make this frame collision-free?

### Frame transmission time
$$T_{fr} = \frac{200 \text{ bits}}{200{,}000 \text{ bps}} = 1 \text{ ms}$$

### Vulnerable time (Pure ALOHA) = $2 \times T_{fr}$ = **2 ms**

### Collision-free requirement
For NO collision, **no other station** can start transmitting within the 2-ms vulnerable window around our frame.

✅ **Answer:** No other station may transmit a frame during the **2 ms** vulnerable window. Equivalently, the channel must be **silent for at least 2 ms** (one frame-time before AND one frame-time after our frame starts).

---

## Q3. (Tut 5) Pure ALOHA Throughput at Different Loads

> Pure ALOHA, 200-bit frames, 200 kbps. Find throughput when system produces:
> (a) 1000 fps, (b) 500 fps, (c) 250 fps

### Setup
- $T_{fr}$ = 1 ms → max channel rate = **1000 frames/sec**
- G = (frames offered per sec) / (max frames per sec) = (offered fps) / 1000

### Apply $S = G \cdot e^{-2G}$ then convert back to fps

| Case | Offered fps | G | $S = Ge^{-2G}$ | Throughput (fps) = S × 1000 |
|---|---|---|---|---|
| (a) | 1000 | 1.0 | 1 × e⁻² = 0.135 | **135 fps** |
| (b) | 500 | 0.5 | 0.5 × e⁻¹ = 0.184 | **184 fps** |
| (c) | 250 | 0.25 | 0.25 × e⁻⁰·⁵ = 0.152 | **152 fps** |

### 🔑 Insight
Maximum at G = 0.5 (= 500 fps offered) → **184 fps actually delivered** = 18.4% of channel.

✅ **Answers:** (a) 135 fps, (b) **184 fps (max!)**, (c) 152 fps

---

## Q4. (Tut 5) Slotted ALOHA Throughput

> Same setup. Find throughput in fps when offered:
> (a) 1000 fps, (b) 500 fps, (c) 250 fps

### Apply $S = G \cdot e^{-G}$

| Case | Offered fps | G | $S = Ge^{-G}$ | Throughput (fps) |
|---|---|---|---|---|
| (a) | 1000 | 1.0 | 1 × e⁻¹ = 0.368 | **368 fps (max!)** |
| (b) | 500 | 0.5 | 0.5 × e⁻⁰·⁵ = 0.303 | **303 fps** |
| (c) | 250 | 0.25 | 0.25 × e⁻⁰·²⁵ = 0.195 | **195 fps** |

### 🔑 Insight
Slotted ALOHA peaks at G = 1 (= 1000 fps offered) → **368 fps** = 36.8% efficiency = **2× better than Pure ALOHA**.

> **G (offered load):** average number of frames *attempted* per frame-time, including retransmissions.
> **S (throughput):** average number of frames *successfully delivered* per frame-time.
> **fps:** frames per second.

✅ **Answers:** (a) **368 fps (max!)**, (b) 303 fps, (c) 195 fps

---

## Q5. (Tut 5) CSMA/CD Minimum Frame Size — 10 Mbps

> CSMA/CD, BW = **10 Mbps**, max prop time = **25.6 µs**. Find min frame size.

### Apply $L_{min} = 2 \cdot T_p \cdot R$

$$L_{min} = 2 \times 25.6 \times 10^{-6} \times 10^7 = 512 \text{ bits} = \mathbf{64 \text{ bytes}}$$

✅ **Answer: 512 bits = 64 bytes** (this is the standard Ethernet minimum frame size!)

---

## Q6. (Tut 5) CSMA/CD Minimum Frame Size — 1 Mbps, 1 ms

> CSMA/CD, BW = **1 Mbps**, max prop time = **1 ms**. Find min frame size.

### Apply formula
$$L_{min} = 2 \times 10^{-3} \times 10^6 = 2000 \text{ bits} = \mathbf{250 \text{ bytes}}$$

✅ **Answer: 2000 bits = 250 bytes**

### Comparison with Q5
- Same network type (CSMA/CD), but **slower BW + longer cable** → **bigger min frame needed** (2000 vs 512 bits)
- This is why long-distance links use other protocols, not CSMA/CD — frames would have to be huge.

---

# 🔵 TUTORIAL 6 PROBLEMS BEGIN HERE

---

## Q7. (Tut 6) CSMA/CD Throughput at Different Speeds

> $B = t/D$, $E = 1/(1 + 5B)$. Given C = 100 Mbps, L = 2000 bits, d = 1 mile = 1609.34 m, signal at speed of light in cable.
>
> (a) Find t and D
> (b) Find B and E
> (c) Repeat with C = 1 Gbps
> (d) Does E increase or decrease with C? Why?

### Speed of light in cable ≈ $2 \times 10^8$ m/s (typical assumption)

### Part (a) — t and D for 100 Mbps

**Propagation delay t:**
$$t = \frac{1609.34}{2 \times 10^8} = 8.05 \times 10^{-6} \text{ s} = \mathbf{8.05\,\mu s}$$

**Transmission delay D:**
$$D = \frac{2000}{10^8} = 2 \times 10^{-5} \text{ s} = \mathbf{20\,\mu s}$$

### Part (b) — B and E for 100 Mbps

$$B = \frac{t}{D} = \frac{8.05}{20} = 0.4025$$

$$E = \frac{1}{1 + 5 \times 0.4025} = \frac{1}{1 + 2.0125} = \frac{1}{3.0125} \approx \mathbf{0.332 \text{ (33.2\%)}}$$

### Part (c) — Repeat with C = 1 Gbps

**t stays the same** (depends on distance, not BW): t = **8.05 µs**

**D changes:**
$$D = \frac{2000}{10^9} = 2 \times 10^{-6} \text{ s} = \mathbf{2\,\mu s}$$

**B and E:**
$$B = \frac{8.05}{2} = 4.025$$

$$E = \frac{1}{1 + 5 \times 4.025} = \frac{1}{1 + 20.125} = \frac{1}{21.125} \approx \mathbf{0.047 \text{ (4.7\%)}}$$

### Part (d) — Effect of higher C on efficiency

✅ **Answer: E DECREASES dramatically as C increases.**

**Why?**
- Higher BW → smaller transmission time D
- But propagation t is fixed by physics (distance)
- So B = t/D grows → denominator (1 + 5B) grows → efficiency drops

**Intuition:** With faster links, you finish sending frames so quickly that the **wasted time waiting for collision detection (= 2t)** becomes proportionally larger. CSMA/CD is fundamentally bad at gigabit speeds. This is why **Gigabit Ethernet uses full-duplex switched links** instead of CSMA/CD.

---

## Q8. (Tut 6) Length of Contention Slot in CSMA/CD

A contention slot = round-trip propagation time = $2 \cdot T_p$.

Speed of light in vacuum = $3 \times 10^8$ m/s.

### (e) 2 km cable, signal = 82% of c

$$\text{speed} = 0.82 \times 3 \times 10^8 = 2.46 \times 10^8 \text{ m/s}$$
$$T_p = \frac{2000}{2.46 \times 10^8} = 8.13\,\mu s$$
$$T_{slot} = 2 \times 8.13 = \mathbf{16.26\,\mu s}$$

### (f) 40 km multimode fiber, signal = 65% of c

$$\text{speed} = 0.65 \times 3 \times 10^8 = 1.95 \times 10^8 \text{ m/s}$$
$$T_p = \frac{40{,}000}{1.95 \times 10^8} = 205.13\,\mu s$$
$$T_{slot} = 2 \times 205.13 = \mathbf{410.26\,\mu s}$$

### (g) 40 km copper cable, signal = 92% of c

$$\text{speed} = 0.92 \times 3 \times 10^8 = 2.76 \times 10^8 \text{ m/s}$$
$$T_p = \frac{40{,}000}{2.76 \times 10^8} = 144.93\,\mu s$$
$$T_{slot} = 2 \times 144.93 = \mathbf{289.86\,\mu s}$$

✅ **Answers:** (e) **16.26 µs**, (f) **410.26 µs**, (g) **289.86 µs**

### 🔑 Insight
Longer cable + slower medium → longer contention slot → more time wasted per collision detection. Long fibre runs are 25× worse than short copper.

---

## Q9. (Tut 6) Collision Timing with Repeaters & Backoff

> Two nodes A & B on opposite ends of a **1200 m** cable. Each has one frame of **1500 bits**. Both transmit at t=0. **4 repeaters** between A and B, each adding **10 µs** processing delay. Rate = **100 Mbps**. CSMA/CD with backoff slots = **135 µs** (this is the round-trip/contention slot time given). After collision, **A draws K=0**, **B draws K=1**. Ignore jam + 96-bit time delay. Signal speed = $2 \times 10^8$ m/s.
>
> (a) One-way propagation delay (with repeater delays)?
> (b) When is A's packet completely delivered at B?
> (c) When is B's packet completely delivered at A?

### Part (a) — One-way propagation delay

**Pure cable propagation:**
$$T_{cable} = \frac{1200}{2 \times 10^8} = 6 \times 10^{-6} \text{ s} = 6\,\mu s$$

**Add 4 repeater delays:**
$$T_{rep} = 4 \times 10 = 40\,\mu s$$

$$T_p = 6 + 40 = \mathbf{46\,\mu s} = 4.6 \times 10^{-5} \text{ s}$$

### Setup for (b) and (c)

- Frame = 1500 bits, R = 100 Mbps → $T_{trans} = 1500/10^8 = 15\,\mu s$
- Round-trip = 2 × 46 = 92 µs
- Slot time given = 135 µs
- After K=0, A waits **0 slots** (= 0 µs)
- After K=1, B waits **1 slot** (= 135 µs)

### Timeline of collision

```
t = 0:        A and B both start transmitting
t = 46 µs:    A's signal reaches B; B detects collision and stops
              B's signal reaches A; A detects collision and stops
              (Both detect collision at same moment t = 46 µs)
              Ignoring jam (per question):
t = 46 µs:    A starts backoff (0 slots) → ready immediately
t = 46 µs:    B starts backoff (1 slot = 135 µs) → ready at t = 181 µs
```

### Part (b) — A's packet delivered at B

A retransmits at **t = 46 µs** (no backoff wait):

```
t = 46 µs:    A starts retransmitting
t = 46 + 15 = 61 µs:   A finishes transmitting (last bit on wire)
t = 61 + 46 = 107 µs:  Last bit arrives at B
```

✅ **Answer (b): t = 107 µs = 1.07 × 10⁻⁴ s**

### Part (c) — B's packet delivered at A

B retransmits at **t = 181 µs** (after 1-slot backoff):

But wait — at t = 181 µs, is the channel free? A finished transmitting at t = 61 µs, and A's last bit reaches B at t = 107 µs. So channel is fully clear at B's location from t = 107 µs onwards. B at t = 181 µs senses idle → transmits.

```
t = 181 µs:   B starts transmitting
t = 181 + 15 = 196 µs:  B finishes
t = 196 + 46 = 242 µs:  Last bit arrives at A
```

✅ **Answer (c): t = 242 µs = 2.42 × 10⁻⁴ s**

---

## Q10. (Tut 6) CSMA/CD Min Frame at 1 Gbps

> CSMA/CD, **1 Gbps**, **1 km** cable, no repeaters, signal = **200,000 km/s**. Find min frame size.

### Setup
- $T_p = 1 \text{ km} / 200{,}000 \text{ km/s} = 5 \times 10^{-6} \text{ s} = 5\,\mu s$
- $L_{min} = 2 \cdot T_p \cdot R = 2 \times 5 \times 10^{-6} \times 10^9$

$$L_{min} = \mathbf{10{,}000 \text{ bits} = 1250 \text{ bytes}}$$

✅ **Answer: 10,000 bits** (way bigger than 64-byte Ethernet — that's why Gigabit doesn't really use CSMA/CD)

---

## Q11. (Tut 6) Collision with Jam Signal — 10 Mbps Ethernet

> Nodes A & B on same **10 Mbps** Ethernet. Prop delay = **225 bit-times**. Both send at t=0, collide. After collision, they finish jam signal. Jam = **48 bits**.
>
> When do they finish transmitting jam?

### Setup
- 1 bit-time at 10 Mbps = 0.1 µs
- Prop delay = 225 bit-times = 22.5 µs

### Timeline

```
t = 0:         A and B start transmitting
t = 225 bt:    A's signal reaches B (and vice versa)
               → both detect collision
t = 225 bt:    Both start jamming
t = 225 + 48 = 273 bt:  Both finish jam signal
```

✅ **Answer: t = 273 bit-times = 27.3 µs**

---

## Q12. (Tut 6) Collision + Backoff + Delivery Time

> A & B opposite ends of cable, propagation delay = **12.5 ms**. Both transmit at t=0. Frames collide. After 1st collision, **A draws K=0**, **B draws K=1**. Ignore jam. BW = 10 Mbps, packet = 1000 bits.
>
> When is A's packet completely delivered at B?

### Setup
- T_p = 12.5 ms = 12,500 µs
- Slot time = 2 × T_p = 25 ms = 25,000 µs
- T_trans of 1000-bit packet = 1000/10⁷ = **100 µs = 0.1 ms**

### Backoff
- A: K=0 → wait 0 slots = 0 µs
- B: K=1 → wait 1 slot = **25 ms** = 25,000 µs

### Timeline (in ms)

```
t = 0:        A and B both start transmitting
t = 12.5 ms:  Both detect collision
              (Ignoring jam per question)
t = 12.5 ms:  A's backoff = 0 → A ready to retransmit immediately
              B's backoff = 25 ms → B ready at t = 12.5 + 25 = 37.5 ms

A retransmits:
t = 12.5 ms:  A starts sending packet
t = 12.5 + 0.1 = 12.6 ms:  A finishes transmitting (last bit on wire)
t = 12.6 + 12.5 = 25.1 ms:  Last bit arrives at B
```

✅ **Answer: A's packet completely delivered at B at t = 25.1 ms**

(B's backoff (37.5 ms) is well after A's packet arrives, so B doesn't interfere.)

---

## 📊 Summary

| Q | Topic | Key result |
|---|---|---|
| 1 | ALOHA backoff range | After K collisions: T_B ∈ {0, 4, 8, ..., (2^K−1)×4} ms |
| 2 | Pure ALOHA vulnerable time | 2 ms silence required |
| 3 | Pure ALOHA throughput | Max 184 fps at G=0.5 |
| 4 | Slotted ALOHA throughput | Max 368 fps at G=1 (2× Pure) |
| 5 | CSMA/CD min frame (10 Mbps, 25.6 µs) | 512 bits = 64 bytes |
| 6 | CSMA/CD min frame (1 Mbps, 1 ms) | 2000 bits = 250 bytes |
| 7 | CSMA/CD efficiency vs speed | E drops sharply as C grows |
| 8 | Contention slot lengths | Depends on cable + medium |
| 9 | Collision timing w/ repeaters | A done at 107 µs, B done at 242 µs |
| 10 | Min frame 1 Gbps | 10,000 bits = 1250 bytes |
| 11 | Jam signal end | t = 273 bit-times |
| 12 | A delivered after backoff | t = 25.1 ms |

---

## 🔑 Master Formula Sheet

```
┌─────────────────────────────────────────────────────────────┐
│  ALOHA THROUGHPUT                                            │
│  Pure:    S = G·e^(-2G);    max = 0.184 at G=0.5             │
│  Slotted: S = G·e^(-G);     max = 0.368 at G=1               │
│  Throughput in fps = S × (1 / T_fr)                          │
├─────────────────────────────────────────────────────────────┤
│  BACKOFF (Binary Exponential)                                │
│  After K collisions: R uniform in {0, 1, ..., 2^K - 1}      │
│  T_B = R × slot_time                                         │
│  ALOHA slot = 2·T_p (round-trip)                             │
├─────────────────────────────────────────────────────────────┤
│  CSMA/CD MIN FRAME                                           │
│  L_min = 2 · T_p · R                                         │
│  Ethernet (10 Mbps, 25.6 µs) = 512 bits = 64 bytes           │
├─────────────────────────────────────────────────────────────┤
│  CSMA/CD EFFICIENCY                                          │
│  E = 1 / (1 + 5B), where B = t/D = T_p/T_trans               │
│  Higher BW → smaller D → bigger B → lower E                  │
├─────────────────────────────────────────────────────────────┤
│  CONTENTION SLOT                                             │
│  Slot = 2·T_p (worst-case collision detection time)          │
│  Depends on cable length + signal speed in medium            │
├─────────────────────────────────────────────────────────────┤
│  COLLISION TIMELINE PATTERN                                  │
│  t = 0: both start                                           │
│  t = T_p: both detect collision                              │
│  t = T_p + jam: jam ends                                     │
│  After backoff: each retransmits → trace each timeline       │
└─────────────────────────────────────────────────────────────┘
```

---

## ⚠️ Common Exam Pitfalls

1. **Slot time for ALOHA = 2·T_p**, not T_fr (often confused)
2. **CSMA/CD min frame**: don't forget to MULTIPLY by bandwidth
3. **Backoff R = 0 means NO wait** (not 1 slot wait)
4. **When tracing collisions**, both stations detect at t = T_p (not 2T_p)
5. **Pure ALOHA vulnerable = 2T_fr; CSMA vulnerable = T_p** (different formulas!)
6. **Higher bandwidth ≠ better CSMA/CD efficiency** — it actually hurts because B = T_p/T_trans grows

---

You now have all 5 tutorials covered. Solve them by hand once without looking, then verify against these solutions.
