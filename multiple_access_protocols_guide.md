# Multiple Access Protocols — Full Study Guide

> Goal: by the end of this file you will (a) understand every slide in plain English, (b) know the formulas and when to use them, (c) be able to solve the numerical questions at the end.

---

## 0. The Big Picture (read this first!)

Imagine **one wire / one radio channel** shared by many computers. Only one can talk at a time, otherwise their signals overlap and get destroyed = **collision**.

The "Multiple Access Protocol" = the *rule* that decides **who talks when**.

There are 3 families of rules:

| Family | Idea | Examples |
|---|---|---|
| **Random Access** | Talk whenever you want, deal with collisions afterwards | ALOHA, Slotted ALOHA, CSMA, CSMA/CD, CSMA/CA |
| **Controlled Access** | Take turns in some organized way | Reservation, Polling, Token Passing |
| **Channelization** | Split the channel itself (frequency / time / code) | FDMA, TDMA, CDMA |

Almost all your slides are about **Random Access**, with a bit on Controlled Access at the end.

---

## 1. Random Access — Core Idea

- No master station. Every node decides *for itself* when to send.
- If two send at the same time → **collision** → both frames are garbage → must retransmit.
- The whole game = "minimize collisions" + "recover when they happen."

Four protocols, in order of cleverness (each fixes a problem of the one before):

1. **Pure ALOHA** — talk anytime
2. **Slotted ALOHA** — talk only at the start of a time slot
3. **CSMA** — listen first, then talk
4. **CSMA/CD** — listen first, talk, AND keep listening while talking to detect collisions early
5. **CSMA/CA** — same as CSMA but for *wireless*, where you can't detect collisions, so you try to *avoid* them

---

## 2. Key Ideas of Random Access (the "model")

- **Stations** = independent nodes, no coordination
- **Channel** = one shared channel; everyone hears every transmission
- **Time** = either continuous (Pure ALOHA, CSMA) or sliced into equal **slots** (Slotted ALOHA)
- **Carrier Sense** = listen before talking ("is anyone speaking?")
- **Collision Detection** = listen *while* talking to catch overlap, then stop immediately
- **Randomness** = after a collision, wait a *random* time before retrying (otherwise you'd just keep colliding)

---

## 3. Pure ALOHA

- Developed at Univ of Hawaii (1970s).
- Rule: **whenever you have data, just send it.**
- If no acknowledgment comes back → assume collision → wait random time → resend.

### Vulnerable Time (very important!)
A frame can collide with another frame that started up to **one frame-time** before it OR up to one frame-time after.

**Vulnerable time = 2 × T_fr** (T_fr = time to transmit one frame)

### Throughput Formula
$$S = G \cdot e^{-2G}$$

- G = average number of frames the network *attempts* per frame-time
- S = average number of frames *successfully* delivered per frame-time
- **Maximum throughput = 0.184 (≈ 18.4%)** when G = 1/2

Meaning: at best, only ~18% of channel capacity is useful. Very wasteful.

---

## 4. Slotted ALOHA

Fix Pure ALOHA's wastefulness: **force everyone to start sending only at the beginning of a fixed time slot.**

### Assumptions
- All frames same size
- Time divided into equal slots (= one frame transmission time)
- All nodes synchronized (master clock)
- All nodes can detect collisions
- After collision → retransmit in a *future* random slot

### Vulnerable Time
Cut in half! **Vulnerable time = T_fr** (only one slot, not two)

### Throughput Formula
$$S = G \cdot e^{-G}$$

- **Maximum throughput = 0.368 (≈ 36.8%)** when G = 1

So: Slotted ALOHA is **2× better** than Pure ALOHA. But still wastes ~63% of the channel.

### Pure vs Slotted ALOHA — Memorize this
| | Max throughput | At G = |
|---|---|---|
| Pure ALOHA | 18.4% | 1/2 |
| Slotted ALOHA | 36.8% | 1 |

---

## 5. Carrier Sense Multiple Access (CSMA)

Big idea: **"Listen before talking."** This drastically reduces collisions because you won't start talking when someone else clearly is.

But collisions can *still* happen because of **propagation delay** — when station A starts sending, the signal takes time to reach station B. During that gap, B can sense an idle channel and also start sending. Boom, collision.

### Vulnerable Time of CSMA
**Vulnerable time = propagation time (T_p)**

(Much smaller than ALOHA's 2·T_fr, since usually T_p << T_fr.)

### Persistence Methods (what to do when channel is busy/idle)

These are 3 strategies for *when* to send after sensing:

#### (a) 1-Persistent CSMA
- If channel busy → keep listening
- The instant it becomes idle → send **immediately** (probability = 1)
- Problem: if multiple stations are waiting, they'll all jump at once → guaranteed collision
- Used in Ethernet

#### (b) Non-Persistent CSMA
- If channel busy → wait a *random* amount of time, then sense again
- If idle → send immediately
- Reduces collisions but wastes channel time (random waits even when channel becomes free)

#### (c) P-Persistent CSMA
- Used when channel is **slotted**
- If idle → send with probability *p*; with probability *(1-p)* wait one time slot and try again
- If busy → wait until next slot and start over
- Best of both worlds: fewer collisions AND better channel utilization

---

## 6. CSMA/CD (Collision Detection) — used in classic Ethernet

CSMA wastes time when collisions happen because each colliding station finishes transmitting its whole frame before realizing.

**Fix:** *listen while you talk*. The moment you detect a collision → STOP immediately, send a brief "jam signal," then back off (wait random time using **binary exponential backoff**).

### Procedure
1. Sense channel (CSMA part)
2. If idle, start transmitting
3. While transmitting, keep listening
4. If collision detected → stop, send jam, wait random, go back to step 1

### Minimum Frame Size — VERY IMPORTANT for numericals
For collision detection to work, the sender must still be transmitting when the collision signal gets back to it.

**Required:** Transmission time ≥ 2 × propagation time
$$T_{fr} \geq 2 \cdot T_p$$

This gives a **minimum frame length**:
$$\text{Min frame size (bits)} = 2 \cdot T_p \cdot \text{Bandwidth}$$

This is why Ethernet has a 64-byte minimum frame.

### Efficiency of CSMA/CD
$$\text{Efficiency} = \frac{1}{1 + 6.44 \cdot a}$$
where $a = T_p / T_{fr}$

- Smaller `a` (short cable or large frames) → higher efficiency
- Larger `a` (long cable or small frames) → lower efficiency

Limit case (1 retry expected): efficiency at best ≈ **1/e ≈ 36.8%** (but usually higher in practice with smart formulas).

---

## 7. CSMA/CA (Collision Avoidance) — used in WiFi

In **wireless**, you cannot reliably detect collisions while transmitting (your own transmission drowns out others). So instead of detecting collisions, we **avoid** them.

### Three avoidance tools:
1. **Inter-Frame Space (IFS)** — wait a small fixed time after sensing idle, before starting
2. **Contention Window** — pick a random number of slots to wait, doubles each retry (binary exponential)
3. **Acknowledgments (ACK)** — receiver must confirm; no ACK = assume collision

### DCF (Distributed Coordination Function)
The basic WiFi MAC layer mode. Uses CSMA/CA + ACKs. No collision detection (wireless), so it adds delays as a "priority" scheme.

---

## 8. Hidden & Exposed Terminal Problems (wireless-only!)

Two nodes might not hear each other but their signals collide at a third node. This is unique to wireless.

### Hidden Terminal Problem
- A and C cannot hear each other but both can reach B
- A is talking to B; C senses idle (can't hear A) and also starts sending → **collision at B**

### Exposed Terminal Problem
- B is sending to A. C is in B's range but wants to send to D (who is far from B).
- C senses B and waits — but actually it's safe to send (D wouldn't be affected)
- Result: wasted opportunity

---

## 9. MACA / RTS-CTS — solves Hidden & Exposed Terminal

Before sending data, do a tiny handshake:

1. Sender → **RTS** (Request To Send) — small packet announcing intent + duration
2. Receiver → **CTS** (Clear To Send) — broadcasts back (heard by anyone near receiver)
3. Now everyone near the receiver knows to stay quiet for that duration (using **NAV** = Network Allocation Vector — internal "do not transmit until time X" timer)
4. Sender sends data → receiver sends ACK

This solves:
- **Hidden terminal:** C hears the CTS from B and stays quiet
- **Exposed terminal:** C only hears RTS from B (not the CTS from A), so C knows it can transmit

---

## 10. Controlled Access Protocols

### Reservation
- Time divided into intervals
- Each interval starts with a **reservation mini-frame** with N slots (one per station)
- A station that wants to transmit sets its bit in its slot
- Then stations transmit in order, conflict-free

### Polling
- One **primary** station controls; others are **secondary**
- Primary asks each secondary in turn: "Got anything?" (SEL/POLL function)
- **Select** = primary wants to send to a secondary
- **Poll** = primary asks secondary if it wants to send
- Simple but the primary is a single point of failure

### Token Passing
- Stations form a logical ring
- A special **token** is passed around
- Only the station holding the token may transmit
- After transmitting (or if nothing to send), pass token to next station
- Token management: lost token, duplicate token, priority all need rules

---

## 11. Quick Reference — All Formulas in One Place

| Protocol | Vulnerable Time | Throughput | Max Throughput |
|---|---|---|---|
| Pure ALOHA | 2·T_fr | S = G·e^(-2G) | 18.4% at G=0.5 |
| Slotted ALOHA | T_fr | S = G·e^(-G) | 36.8% at G=1 |
| CSMA | T_p | (depends on persistence) | varies |
| CSMA/CD | T_p | 1 / (1 + 6.44a), a = T_p/T_fr | depends on `a` |

**Key relation for CSMA/CD min frame size:**
$$\text{Min frame size} = 2 \cdot T_p \cdot \text{Bandwidth}$$

**Propagation time:** T_p = distance / signal speed
**Transmission time:** T_fr = frame size / bandwidth

---

# 12. NUMERICAL PROBLEMS — FULLY SOLVED

These are the problems from your slides. Learn the *pattern*, not just the answer.

---

### Problem 1: CSMA/CD Minimum Frame Size

> A CSMA/CD network running at **1 Gbps** over a **1 km** cable, with no repeaters. The signal speed in the cable is **200,000 km/s**. What is the minimum frame size?

**Step 1 — Propagation time T_p:**
$$T_p = \frac{\text{distance}}{\text{speed}} = \frac{1 \text{ km}}{200{,}000 \text{ km/s}} = 5 \times 10^{-6} \text{ s} = 5\,\mu s$$

**Step 2 — Minimum transmission time:**
$$T_{fr,\min} = 2 \cdot T_p = 10\,\mu s = 10 \times 10^{-6} \text{ s}$$

**Step 3 — Minimum frame size in bits:**
$$\text{Min frame} = T_{fr,\min} \times \text{Bandwidth} = 10 \times 10^{-6} \times 10^9 = 10{,}000 \text{ bits}$$

**Answer: 10,000 bits = 1250 bytes**

---

### Problem 2: 10 Mbps Ethernet, two stations far apart

> Stations A and B are on the same 10 Mbps Ethernet. Propagation delay between them is **225 bit times**. Suppose A and B send frames at t = 0. Frames collide. After the collision, A transmits a jam signal. Assume a 48-bit jam signal.
>
> (a) When does B detect the collision?
> (b) When does A detect the collision?
> (c) When does A finish transmitting jam?

**Setup:** Bit time = 1/(10 Mbps) = 0.1 µs. Propagation delay = 225 bit times.

**(a) When does B detect collision?**
A's signal reaches B after 225 bit times. B started at 0, so B sees A's signal arriving at t = 225 bit times. That's when B detects collision.
**Answer: t = 225 bit times**

**(b) When does A detect collision?**
B's signal also takes 225 bit times to reach A. So A detects collision at t = 225 bit times too.
**Answer: t = 225 bit times**

**(c) When does A finish jam?**
A starts jamming at t = 225, jam is 48 bits long → finishes at t = 225 + 48 = **273 bit times**.

---

### Problem 3: CSMA/CA / Token-Ring style timing

> **(From slide)** 10 Mbps token-ring of 4 stations, each separated by 2 km. Signal speed 2×10⁸ m/s. Frame size = 1 Kbit. Compute (a) ring latency, (b) cycle time, (c) effective throughput per station.

**Step 1 — Bit propagation across one segment (2 km):**
$$T_p = \frac{2000 \text{ m}}{2\times10^8 \text{ m/s}} = 10\,\mu s$$

**Step 2 — Total ring latency (4 stations × 2 km = 8 km):**
$$T_{ring} = \frac{8000}{2\times10^8} = 40\,\mu s$$

(If each station also adds, say, 1-bit delay, you'd add 4 × 0.1 µs = 0.4 µs. Use only what slide specifies.)

**Step 3 — Frame transmission time:**
$$T_{fr} = \frac{1000 \text{ bits}}{10 \times 10^6 \text{ bps}} = 100\,\mu s$$

**Step 4 — Cycle time** (one full round of all 4 stations sending one frame each):
Approximate: 4 × T_fr + ring latency = 400 + 40 = **440 µs**

**Step 5 — Effective throughput per station:**
Each station sends 1 Kbit per cycle:
$$\text{Throughput} = \frac{1000 \text{ bits}}{440\,\mu s} \approx 2.27 \text{ Mbps per station}$$

(Exact answer depends on the slide's assumptions — apply the same method.)

---

### Problem 4: 10 Mbps Broadcast Ethernet, propagation 325 bit times

> 5 nodes; propagation delay between any two = 325 bit times; frame size 1000 bits; CSMA/CD with binary exponential backoff. After 1st collision, A wins random number 0, B wins 1.

**Bit time:** 1/10 Mbps = 0.1 µs

| Time (bit times) | Event |
|---|---|
| 0 | A and B begin transmission |
| 325 | A and B detect collision |
| 325 + 48 = 373 | Jam signals end |
| After backoff (A: 0 slots, B: 1 slot, slot = 2·T_p = 650 bit times) | A retransmits immediately; B waits 650 bit times |
| ~ 373 + (small wait) → A transmits 1000-bit frame | |
| 373 + 1000 = 1373 | A finishes transmission |

So A successfully transmits at around bit-time 1373; B's retry happens later because its backoff slot is longer than A's.

(The exact slide solution computes "B retransmits at t = 373 + 650 = 1023 bit times" — and by then A's frame is in the air, so B senses busy and defers.)

---

### Problem 5: Token-Ring with 30 Mbps, 10 Mbps Broadcast comparison

> Suppose nodes A and B are on the same 10 Mbps broadcast channel; propagation delay between A and B is 325 bit times. Suppose CSMA/CD and Ethernet packets are used. Frame size 1500 bits.

Same method as Problem 2/4. Key takeaways:
- Collision detection time = 2 × propagation = 650 bit times
- Frame must be at least 650 bits long for CSMA/CD to work → 1500-bit frame is fine
- After collision: jam (48 bits), then exponential backoff: random {0, 1} slots × 2 × T_p

---

# 13. EXAM CHEAT SHEET (one-page)

**ALOHA family:**
- Pure ALOHA: send anytime; vulnerable = 2T_fr; max throughput = 18.4%
- Slotted ALOHA: send at slot start; vulnerable = T_fr; max throughput = 36.8%

**CSMA family:**
- Listen before talking. Vulnerable time = T_p
- 1-persistent: send immediately when idle (Ethernet)
- Non-persistent: random wait when busy
- P-persistent: send with probability p in each idle slot

**CSMA/CD:**
- Listen while talking; abort + jam on collision; binary exponential backoff
- Min frame size = 2 × T_p × Bandwidth
- Efficiency = 1 / (1 + 6.44 · T_p/T_fr)

**CSMA/CA (wireless):**
- Cannot detect collisions; uses IFS + contention window + ACK
- Hidden terminal → use **RTS/CTS**
- NAV tells other stations how long to stay quiet

**Controlled Access:**
- Reservation: book a slot in mini-frame
- Polling: primary asks each secondary
- Token passing: hold the token = right to send

**Magic Numbers to memorize:**
- Pure ALOHA max = 1/(2e) = 18.4%
- Slotted ALOHA max = 1/e = 36.8%
- CSMA/CD efficiency factor = 6.44
- Ethernet jam signal = 48 bits
- Ethernet min frame = 64 bytes (= 512 bits)

---

# 14. How to Use This for Your Exam

1. **Today:** Read sections 0–11 once. Don't memorize. Just *follow the story* (each protocol fixes the previous one's flaw).
2. **Tomorrow:** Re-read; this time make your *own* one-page version of section 13 by hand. Writing = remembering.
3. **Day before exam:** Solve all 5 numericals in section 12 *without looking*. Then check.
4. **Day of exam:** Glance at section 13 cheat sheet. Done.

If anything here is unclear or you want me to expand a section (e.g., "explain binary exponential backoff in detail" or "redo problem 3 with different numbers"), just ask.
