# Link Layer & Error Detection — Full Study Guide (Chapter 2)

> **How to use:** Each section names the slide(s) it covers. Open your PPT side-by-side. ASCII diagrams render best in monospace.

---

## 0. The Big Picture — What This Chapter Is About

The **Link Layer** is responsible for getting bits from one device to the *next* device on the wire/wireless link (one hop at a time). Two big jobs:

1. **Frame the data** + handle access to the medium (we covered this in Chapter 1: ALOHA, CSMA, etc.)
2. **Make sure the bits arrive correctly** — detect (and sometimes fix) errors caused by noise on the wire

This chapter focuses on job #2: **error detection and correction**.

```
   Sender                                          Receiver
   ┌──────────┐                                   ┌──────────┐
   │ Original │                                   │ Original │
   │  data    │                                   │  data    │
   └────┬─────┘                                   └────▲─────┘
        │                                              │
        ▼  add redundancy bits                         │  check + fix/discard
   ┌──────────┐    →→→ NOISE on wire →→→         ┌──────────┐
   │ Codeword │  ─────────────────────────►       │ Received │
   └──────────┘  (some bits may flip!)            │ codeword │
                                                  └──────────┘
```

The whole game = "add a few extra bits so the receiver can tell whether what arrived is what was sent."

---

## 1. Link Layer Introduction (📑 Slides 1–2)

### Terminology

- **Nodes** = hosts and routers (anything that has a network interface)
- **Links** = the actual physical channels connecting nodes (wired Ethernet, fiber, WiFi, etc.)
- **Frame** = the link-layer data unit (a layer-2 packet). It *encapsulates* the IP datagram from above.

```
What goes on the wire:

┌──────────┬────────────────────────────┬──────────┐
│  Header  │   IP datagram (payload)    │  Trailer │
│ (MAC etc)│                            │  (CRC)   │
└──────────┴────────────────────────────┴──────────┘
                  ↑
              The whole thing = a FRAME
```

### Two link types

- **Wired** (twisted pair, fiber, coax) — low error rate
- **Wireless** — much higher error rate (interference, fading)

That's why wireless links use stronger error-correcting codes than wired links.

---

## 2. Link Layer Services (📑 Slides 2–4)

The link layer provides several services. Know all of them:

### (a) Framing & Link Access
- Encapsulate datagram into a frame (add header + trailer)
- Use **MAC addresses** in the frame header for source/destination
- ⚠ MAC address ≠ IP address (this was Chapter 3)

### (b) Reliable Delivery (between adjacent nodes)
- Seldom used on wired links (already low error rate — handle errors at higher layers)
- Common on wireless (high error rate → fix here, don't waste end-to-end retries)

### (c) Flow Control
- Pace sender so it doesn't overwhelm receiver's buffer
- DLL (Data Link Layer) typically uses **feedback-based** flow control (receiver tells sender to slow down)
- Alternative: **rate-based** (explicit limit on send rate) — less common at link layer

### (d) Error Detection
- Errors caused by signal attenuation, noise, interference
- Receiver detects errors → asks for retransmission OR drops frame

### (e) Error Correction
- Receiver not only detects but **corrects** the error itself, no retransmission
- Common in wireless (where retransmissions are expensive)

### (f) Half-Duplex vs Full-Duplex
- **Half-duplex** — both ends can transmit, but not simultaneously (walkie-talkie style)
- **Full-duplex** — both ends can transmit at the same time (modern Ethernet)

---

## 3. Where Is the Link Layer Implemented? (📑 Slide 5)

In every host and router. Specifically, in the **adapter** (also called **NIC** = Network Interface Card):

```
   ┌─────────────────────────────┐
   │           Host              │
   │ ┌─────────────────────────┐ │
   │ │   Operating System      │ │
   │ │   (Network protocols    │ │
   │ │    in software)         │ │
   │ └────────────┬────────────┘ │
   │              │ system bus   │
   │ ┌────────────▼────────────┐ │
   │ │     NIC Adapter         │ │
   │ │  • Ethernet, WiFi, ...  │ │
   │ │  • MAC sublayer in HW   │ │
   │ │  • Implements link layer│ │
   │ └────────────┬────────────┘ │
   └──────────────┼──────────────┘
                  │
              [Network]
```

A NIC is a combination of **hardware + software + firmware**.

---

## 4. Adaptors Communicating (📑 Slide 6)

```
   Sending Host                          Receiving Host
   ┌─────────────┐                       ┌─────────────┐
   │  Datagram   │                       │  Datagram   │
   └──────┬──────┘                       └──────▲──────┘
          │ pass down                           │ pass up
   ┌──────▼──────┐                       ┌──────┴──────┐
   │   Sending   │ ── frames over wire ► │  Receiving  │
   │   adapter   │                       │   adapter   │
   │             │                       │             │
   │  • Encaps   │                       │  • Look at  │
   │    datagram │                       │    header   │
   │  • Add MAC, │                       │  • Check CRC│
   │    CRC, etc.│                       │  • Extract  │
   └─────────────┘                       │    datagram │
                                         └─────────────┘
```

- **Sending side:** wraps the datagram in a frame (adds MAC, CRC, error-control bits)
- **Receiving side:** checks for errors, strips header, passes datagram up

---

## 5. Why Error Detection at All? (📑 Slide 7)

> **No real channel is error-free.** Period.

- As bits-per-area or transmission rate goes up, errors become more likely
- It's **impossible to detect or correct 100% of errors** — no code is perfect
- Goal: detect the *types* of errors most likely on this medium with reasonable overhead

---

## 6. Types of Errors (📑 Slides 8–10)

```
                    Errors
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
    Single-bit   Multiple-bit     Burst
```

### (a) Single-Bit Error
Exactly **one** bit in the unit changed.

```
Sent:     0 0 0 0 0 0 1 0
Received: 0 0 0 0 1 0 1 0
                  ↑
              one bit flipped
```

Common in *parallel* transmission (each bit on its own wire — a glitch on one wire affects only that bit). Rare in serial links.

### (b) Multiple-Bit Error
Two or more **non-consecutive** bits flipped.

```
Sent:     0 1 0 0 0 0 1 0
Received: 0 0 0 0 0 1 1 0
            ↑       ↑
        two non-adjacent flips
```

### (c) Burst Error
Two or more **consecutive** bits flipped (a "run" of errors).

```
Sent:     0 1 0 0 1 1 0 0 1 0 0 0 1 1
Received: 0 1 0 1 1 0 0 1 0 0 0 0 1 1
                ▲▲▲▲▲▲▲▲▲▲▲
                burst of errors
```

Most common in **serial** transmission (a noise spike lasting microseconds knocks out many bits in a row). This is what we usually design for.

---

## 7. Error Control Strategies (📑 Slide 11)

Two ways to handle errors:

### (a) Error Detection + Retransmission (ARQ)
- **Error Detection Codes** — detect the error happened
- **Automatic Repeat reQuest (ARQ)** — receiver detects error → ask sender to resend
- Used on wired links where errors are rare → cheap to retransmit when they happen

### (b) Forward Error Correction (FEC)
- **Error Correction Codes** — receiver can both detect AND fix the error itself
- No retransmission needed
- Used on wireless / satellite where retransmission is expensive

---

## 8. Flow Control (📑 Slide 12)

Different from error control! Flow control = pacing.

- **Problem:** sender might send faster than receiver can process → receiver's buffer overflows → packets lost
- **Feedback-based:** receiver sends explicit "slow down" / "stop" / "go" messages — this is what DLL typically uses
- **Rate-based:** sender is configured with a maximum rate — no feedback

---

## 9. Redundancy — The Core Idea (📑 Slide 13)

To detect or correct errors, the sender must add **extra bits** that depend on the data. These extra bits = **redundancy**.

```
   Sender                                      Receiver
   ┌──────────┐                               ┌──────────┐
   │ Message  │                               │ Message  │ → if OK, accept
   └────┬─────┘                               └────▲─────┘ → else discard
        │                                          │
        ▼                                          │
   ┌──────────┐                              ┌─────┴──────┐
   │ Generator│                              │  Checker   │
   └────┬─────┘                              └─────▲──────┘
        │                                          │
   ┌────▼─────────────┐  unreliable wire   ┌───────┴─────────┐
   │ Message + redund │ ──────────────────►│ Received w/redund│
   └──────────────────┘                    └─────────────────┘
```

The redundancy bits are mathematically derived from the message bits. If the received bits don't satisfy that relationship → error detected.

---

## 10. Coding (📑 Slide 14)

**Coding** = the process of adding redundancy. Two families:

### (a) Block Codes
- Divide message into k-bit **datawords**
- Add r redundant bits → form (k+r)-bit **codewords**
- Each codeword depends only on its own dataword (memoryless)
- Examples: Parity, CRC, Hamming code

### (b) Convolutional Codes
- Treat data as a continuous stream
- Each output bit depends on the current AND past input bits (has memory)
- Used in deep-space comms, mobile networks
- (Not heavily emphasized in your slides)

---

## 11. Block Coding Mechanics (📑 Slide 15)

```
k bits dataword:        2^k possible datawords
                              │
                              ▼ add r redundant bits
n = k + r bits codeword: 2^n possible codewords
                              │
                              └─ but only 2^k are VALID
                                 (the rest are "invalid" → indicate error!)
```

**Key idea:** since 2^n > 2^k, there are extra "spare" bit patterns that should never appear in valid transmission. If the receiver gets one of those → must be an error.

### Error Detection Process (📑 Slide 16)

```
Sender:                                    Receiver:
   ┌──────────┐                              ┌─────────┐
   │ Dataword │ k bits                       │Dataword │ k bits → extract
   └────┬─────┘                              └────▲────┘
        │                                         │
        ▼                                         │
   ┌──────────┐                              ┌────┴────┐
   │ Generator│                              │ Checker │ → if OK, extract
   └────┬─────┘                              └────▲────┘ → else, discard
        │                                         │
   ┌────▼─────┐  unreliable wire    ┌─────────────┴──┐
   │ Codeword │ ───────────────────►│Received codeword│
   └──────────┘                     └─────────────────┘
       n bits                              n bits
```

### Limitation (📑 Slide 17)

- Any code can detect only the **types of errors it was designed for**
- Some error patterns may go undetected (if the corrupted bits happen to form *another* valid codeword!)
- **No code can detect every possible error** → it's a tradeoff

---

## 12. Common Detection Methods (📑 Slide 18)

The three you must know:

1. **Parity Check** (simplest)
2. **Cyclic Redundancy Check (CRC)** (used in Ethernet, WiFi, etc.)
3. **Checksum** (used in IP, TCP, UDP)

---

## 13. Parity Check (📑 Slides 19–25)

### Single Parity Check Code

For a k-bit dataword, add **one** parity bit so the total number of 1s is even (even parity) or odd (odd parity).

**Codeword length = k + 1** bits.

### Even parity rule
The parity bit = (XOR of all dataword bits) → ensures total 1s in codeword is even.

### How it detects errors

- 1 bit flips → number of 1s changes by ±1 → parity is wrong → DETECTED ✓
- 2 bits flip → number of 1s changes by 0 or ±2 → parity stays correct → NOT DETECTED ✗
- 3 bits flip → parity wrong → DETECTED ✓
- In general: **detects any odd number of bit errors; misses any even number of errors**

### Worked Example: (4,3) Even Parity Code (📑 Slide 22)

Dataword length k=3 → 8 possible datawords. Add 1 parity bit → codeword length n=4.

| Dataword | Sum of 1s | Parity bit | Codeword |
|---|---|---|---|
| 000 | 0 (even) | 0 | 0000 |
| 001 | 1 (odd)  | 1 | 0011 |
| 010 | 1 (odd)  | 1 | 0101 |
| 011 | 2 (even) | 0 | 0110 |
| 100 | 1 (odd)  | 1 | 1001 |
| 101 | 2 (even) | 0 | 1010 |
| 110 | 2 (even) | 0 | 1100 |
| 111 | 3 (odd)  | 1 | 1111 |

(Parity bit makes total 1s even.)

### How the receiver checks (📑 Slide 23)

1. Receiver gets a codeword
2. Computes the sum of all bits mod 2 (= **syndrome**)
3. **Syndrome = 0** → assume no error, accept dataword (drop parity bit)
4. **Syndrome = 1** → error detected, discard

### IMPORTANT: Parity can DETECT but not CORRECT

- The error could be in *any* of the n positions (including the parity bit itself)
- Receiver knows "something is wrong" but not "which bit"
- Cannot detect: even-numbered burst of errors

### Worked Example 3: (📑 Slides 24–25) Detailed scenarios

> Sender sends dataword `10111`. Codeword created (assume even parity) = `101111` (one extra parity bit at the end).

Let's go through 5 scenarios:

| # | Scenario | Received codeword | Syndrome | Result |
|---|---|---|---|---|
| 1 | No error | `101111` | 0 | ✓ Accept, dataword = 10111 |
| 2 | Bit a₁ flipped | `001111` | 1 | ✗ Detected, discard |
| 3 | Two bits flipped | `101001` | 0 | ✗ MISSED — accepted as 10100 (wrong!) |
| 4 | Three bits flipped | hits depend | 1 | ✓ Detected |
| 5 | Four bits flipped | various | 0 | ✗ MISSED |

**Pattern:** odd number of errors → caught. Even number → missed.

---

## 14. 2-D Parity (Horizontal + Vertical) (📑 Slides 26–28)

To catch *more* errors, arrange data in a grid and add parity for **each row AND each column**.

```
Original 4×7 data block (with 5th row & 8th col added for parity):

Row →    1 1 0 0 1 1 1 │ 1   ← row parity
         1 1 0 1 0 0 0 │ 1
         0 1 0 1 0 1 1 │ 0
         1 0 0 0 0 1 0 │ 0
         ─────────────── 
         1 1 0 0 1 1 0   ← column parity
```

### What it can detect

- **1-bit error:** one row parity AND one column parity both flip → tells you EXACT location → can also CORRECT it ✓
- **2-bit error in same row:** column parities catch it (but can't pinpoint)
- **3-bit errors:** usually detected
- **4-bit errors arranged in a rectangle:** can be MISSED (corners cancel out)

```
Example of an undetectable 4-error pattern:
   X . . X     ← bits flipped at corners
   . . . .
   . . . .
   X . . X     ← row parities unchanged, col parities unchanged
```

### Take-away

2-D parity is much stronger than 1-D parity but still not foolproof. Used as a teaching example; CRC is what's actually used in real networks.

---

## 15. Code Rate (📑 Slide 29)

$$\text{Code rate} = \frac{k}{n}$$

where k = dataword bits, n = codeword bits.

- **Higher code rate** (closer to 1) → less overhead, but weaker error correction
- **Lower code rate** (closer to 0) → more overhead, but stronger error correction

Tradeoff between **bandwidth efficiency** and **error protection**.

---

## 16. Cyclic Codes & CRC (📑 Slides 30–34)

### Cyclic Codes

A special class of block codes with a beautiful mathematical property:

> **Property:** If you take a valid codeword and *cyclically shift* it, you get another valid codeword.

```
Example: 1011000 is a codeword
         0110001 (shifted left by 1) is also a codeword
         1100010 is also a codeword
         ...
```

This makes them efficient to encode/decode with simple shift-register hardware. **CRC** is the most famous cyclic code.

### CRC = Cyclic Redundancy Check

Used in **Ethernet, WiFi, hard disks, ZIP files** — practically everything.

---

## 17. CRC — How It Works (📑 Slides 31–34)

### The setup

Sender and receiver agree on a **divisor (generator polynomial)** in advance. Call it G, which is (r+1) bits long. The CRC will be r bits.

### Sender side — encoding

```
1. Take the k-bit dataword.
2. Append r zero bits to the right → makes a (k+r)-bit number.
3. Divide that number by G using BINARY (mod-2) division.
   ⚠ Mod-2 means: subtraction = XOR, no borrow/carry.
4. The REMAINDER (r bits) is the CRC.
5. Append the CRC to the original dataword → CODEWORD.
```

```
Dataword:  D D D D ... D D
Step 2:    D D D D ... D D 0 0 0 0   ← appended r zeros
                              │
                              ▼ divide by G
                           Remainder R (r bits)
Codeword:  D D D D ... D D R R R R   ← original + CRC
```

### Receiver side — checking

```
1. Take received codeword (k+r bits).
2. Divide by SAME G using mod-2 division.
3. If remainder = 0 → assume no error, strip CRC, accept dataword.
4. If remainder ≠ 0 → error detected, discard.
```

The remainder is called the **syndrome**.

### Why does it work?

Sender constructs the codeword so that it's divisible by G with zero remainder. If errors don't change the codeword in a way that keeps it divisible by G, the remainder won't be zero → caught.

---

## 18. CRC — Worked Examples (📑 Slides 33–34)

### 🧮 Example 1: Data = 1001, Divisor = 1011

**Step 1:** Append r = 3 zeros (since divisor has 4 bits, r = 4-1 = 3):
```
   1001 → 1001 000
```

**Step 2:** Mod-2 divide 1001000 by 1011:

```
                1 0 1
            ─────────────
   1 0 1 1 │ 1 0 0 1 0 0 0
            1 0 1 1                ← XOR
            ─────────
              0 1 0 1 0 0 0
                1 0 1 1            ← shift, XOR (when leading bit = 1)
                ─────────
                0 0 0 1 0 0 0
                    1 0 1 1        ← (leading bit was 0, so quotient bit = 0)
                    
(continue until last 3 digits remain → that's the remainder)
```

After working through:
- **Remainder = 110**
- **Codeword = 1001 110**

That's what gets sent on the wire.

### 🧮 Example 2: Data = 1010, Divisor = 10111

Divisor has 5 bits → r = 4 zeros to append.

**Step 1:** 1010 → 1010 0000

**Step 2:** Divide 10100000 by 10111 mod-2:

```
                1 1 1
           ─────────────────
   1 0 1 1 1 │ 1 0 1 0 0 0 0 0
              1 0 1 1 1
              ─────────
              0 0 1 1 1 0 0 0 0
                  1 0 1 1 1
                  ─────────
                  0 1 0 0 1 0 0
                    1 0 1 1 1
                    ─────────
                    Remainder = 1 1 1 1 (4 bits)
```

- **Remainder = 1111**
- **Codeword = 1010 1111**

### Receiver check (using Example 1)

Receiver gets 1001110. Divides by 1011:
- If remainder = 000 → no error
- If anything else → error

(In practice, hardware does this in a few clock cycles using an XOR shift register.)

### Power of CRC

A well-chosen generator polynomial of degree r can detect:
- All single-bit errors
- All double-bit errors
- All burst errors of length ≤ r
- Most longer bursts

That's why it's the workhorse of all real networks.

---

## 19. Checksum (📑 Slides 35–43)

Used by **TCP, UDP, IP** (i.e., higher layers, but conceptually similar — your slides include it here).

### Idea

Treat the data as a series of equal-length numbers. Add them up. Send the sum (or its complement) along with the data. Receiver re-adds and checks.

### Naive Example (📑 Slide 36)

> Send 5 numbers: (7, 11, 12, 0, 6). Sum = 36. Send: 7, 11, 12, 0, 6, 36.

Receiver adds first 5 numbers, compares to the 6th:
- Match → no error
- Mismatch → error

### Better Example (📑 Slide 37)

> Sender sends (7, 11, 12, 0, 6, **−36**) — sends the *negative* of the sum.

Receiver adds all 6 numbers:
- If sum = 0 → no error
- If sum ≠ 0 → error

This is more elegant — receiver just checks for zero.

### One's Complement Arithmetic (📑 Slides 38–43)

In actual networks, we work with fixed-size unsigned binary (no negatives). To make "subtraction = check for zero" work, we use **one's complement** arithmetic.

#### What is one's complement?
The one's complement of a binary number = flip every bit.

#### Examples (📑 Slides 41–42)

- Number 21 in 5 bits: `10101`
  - Wrap leftmost bit to right of 4-bit: `0101 + 1 = 0110` = **6** (in 4-bit one's complement form)
- Number −6 in one's complement (4 bits) = flip bits of 0110 = **1001**

(Don't get tangled in the bit gymnastics — the *concept* is what matters: one's complement gives us a way to represent negatives, and "sum to zero" = "no error.")

---

## 20. Internet Checksum (📑 Slides 44–46)

This is the actual algorithm used in IPv4, TCP, UDP.

### Sender Side

```
1. Divide the message into 16-bit words.
2. If the message length isn't a multiple of 16, pad with zeros.
3. Add ALL the 16-bit words using one's complement addition.
4. Take the one's complement of the final sum → this is the CHECKSUM.
5. Send: data + checksum.
```

### Receiver Side

```
1. Divide received message (data + checksum) into 16-bit words.
2. Add them all using one's complement addition.
3. Take the one's complement of the result.
4. If result = 0 → message accepted.
   If result ≠ 0 → message rejected.
```

### Worked Example (📑 Slide 46)

> Send 8 hex bytes: `46 6F 72 75` and `6F 75 73 21` (= "Forous!" or similar)

**Sender side:**

| Position | Bytes | 16-bit word |
|---|---|---|
| 1 | 46 6F | `0100 0110 0110 1111` |
| 2 | 72 75 | `0111 0010 0111 0101` |
| 3 | 6F 75 | `0110 1111 0111 0101` |
| 4 | 73 21 | `0111 0011 0010 0001` |

Add all four 16-bit words using one's complement (carry-around) addition:
- Sum (with carry-around) = some 16-bit value, let's call it S
- Checksum = one's complement of S = flip all bits of S

Send the 4 words + the checksum.

**Receiver side:**
- Adds all 5 words (data + checksum) using one's complement
- Takes one's complement of the result
- If 0 → accept; else → reject

### Why one's complement specifically?

- Easy to compute in hardware (just XOR and add)
- Has the elegant "sum should equal 0" check
- Catches single-bit, most multi-bit, and some burst errors
- **Weaker than CRC** — but cheap, used at higher layers where CRC also runs underneath

---

## 21. Comparison: Parity vs CRC vs Checksum

| | Parity | CRC | Checksum |
|---|---|---|---|
| Where used | Educational, simple links | Ethernet, WiFi, disks (link layer) | TCP, UDP, IP (transport/network layer) |
| Math | XOR | Polynomial division (mod 2) | One's complement addition |
| Overhead | 1 bit | 16 / 32 bits typical | 16 bits |
| Detects single-bit | ✓ | ✓ | ✓ |
| Detects double-bit | ✗ | ✓ | usually ✓ |
| Detects burst errors | weak | very strong (≤r bits guaranteed) | moderate |
| Hardware cost | trivial | low (shift register) | trivial |
| Speed | fast | fast | very fast |

---

## 22. NUMERICAL & CONCEPTUAL QUESTIONS — Worked

### 🧮 Q1: Even parity bit

> Compute the even parity bit for the dataword `1011001`.

Count the 1s: 1+0+1+1+0+0+1 = **4 (even)** → parity bit = **0** → codeword = `10110010`.

### 🧮 Q2: Odd parity bit

> Compute the odd parity bit for `1100110`.

Count of 1s = 4 (even) → for odd parity, we need to *make* it odd → parity bit = **1** → codeword = `11001101`.

### 🧮 Q3: Will a 2-bit error be detected by single parity?

> Sent `01100110` (even parity). Received `01001110`. Detected?

Count 1s in received: 0+1+0+0+1+1+1+0 = 4 (still even) → syndrome = 0 → **NOT detected**.

(This shows the weakness — 2 bits flipped, parity unchanged.)

### 🧮 Q4: CRC computation

> Data = `110101`, Generator = `1001`. Find the codeword.

**Step 1:** r = 3 (since divisor has 4 bits). Append 3 zeros: `110101 000`.

**Step 2:** Mod-2 divide 110101000 by 1001:

```
         1 1 1 1 0 1
       ─────────────────
1 0 0 1 │ 1 1 0 1 0 1 0 0 0
         1 0 0 1
         ─────────
         0 1 0 0 0 1 0 0 0
             1 0 0 1
             ─────────
             0 0 1 1 1 0 0 0
                 1 0 0 1
                 ─────────
                 0 1 1 1 0 0
                     1 0 0 1
                     ─────────
                     0 1 1 1 0
                       (only 4 bits left)
                     Remainder = 0 1 1 (last 3 bits)
```

(Following the slide-style algorithm carefully, the **remainder = 011**.)

**Codeword = 110101 011**

### 🧮 Q5: CRC check at receiver

> Receiver gets `110101011`. Same generator `1001`. Is there an error?

Divide 110101011 by 1001 mod-2. If you do the math correctly, **remainder = 000** → no error → strip last 3 bits → dataword = `110101` ✓.

### 🧮 Q6: Checksum computation

> Numbers to send: 7, 11, 12, 0, 6. Compute checksum (use simple sum for clarity).

Sum = 7+11+12+0+6 = 36.
Send: 7, 11, 12, 0, 6, **36**. (Or in real Internet checksum: send 7,11,12,0,6 + one's complement of 36.)

Receiver adds the first 5 → 36 → matches → ✓.

If a number got corrupted (say 11 → 12), receiver computes 7+12+12+0+6 = 37 ≠ 36 → ✗ error.

### 🧮 Q7: Code rate

> A (7, 4) Hamming code has k=4, n=7. Code rate?

Code rate = k/n = **4/7 ≈ 0.57**. Means 57% of transmitted bits are useful data; 43% is overhead for error correction.

---

## 23. EXAM CHEAT SHEET

```
┌─────────────────────────────────────────────────────────────┐
│  LINK LAYER SERVICES                                         │
│  • Framing + link access (uses MAC addresses)                │
│  • Reliable delivery (rare on wired, common on wireless)     │
│  • Flow control (feedback-based usually)                     │
│  • Error detection / correction                              │
│  • Half- vs full-duplex                                      │
├─────────────────────────────────────────────────────────────┤
│  LAYER LIVES IN: NIC adapter (HW + SW + firmware)            │
├─────────────────────────────────────────────────────────────┤
│  TYPES OF ERRORS                                             │
│  • Single-bit: 1 bit flipped (rare on serial)                │
│  • Multiple-bit: ≥2 non-adjacent flips                       │
│  • Burst: ≥2 consecutive flips (most common!)                │
├─────────────────────────────────────────────────────────────┤
│  ERROR CONTROL                                               │
│  • Detection + ARQ retransmit (wired)                        │
│  • Forward Error Correction (wireless/satellite)             │
├─────────────────────────────────────────────────────────────┤
│  REDUNDANCY: extra bits derived from data so receiver        │
│  can detect/correct corruption                               │
├─────────────────────────────────────────────────────────────┤
│  BLOCK CODE BASICS                                           │
│  • k bits dataword → n=k+r bits codeword                     │
│  • 2^k valid codewords out of 2^n total                      │
│  • Code rate = k/n  (higher = less protection)               │
├─────────────────────────────────────────────────────────────┤
│  PARITY (single-bit parity)                                  │
│  • Add 1 bit so total 1s is even (or odd)                    │
│  • Detects ANY ODD number of bit errors                      │
│  • Misses any EVEN number of errors                          │
│  • Cannot correct                                            │
├─────────────────────────────────────────────────────────────┤
│  2-D PARITY                                                  │
│  • Row + column parity                                       │
│  • Can correct 1-bit errors (row+col pinpoints location)     │
│  • Misses 4 errors at corners of a rectangle                 │
├─────────────────────────────────────────────────────────────┤
│  CRC                                                         │
│  • Sender: append r zeros to data, divide by G mod-2,        │
│    remainder = CRC, codeword = data + CRC                    │
│  • Receiver: divide received by G mod-2; remainder=0 → OK    │
│  • Detects ALL bursts of length ≤ r                          │
│  • Used in Ethernet, WiFi, disks                             │
├─────────────────────────────────────────────────────────────┤
│  INTERNET CHECKSUM                                           │
│  • 16-bit words, one's complement sum                        │
│  • Send data + complement of sum                             │
│  • Receiver: sum all words; if complement = 0 → OK           │
│  • Used in IP, TCP, UDP                                      │
└─────────────────────────────────────────────────────────────┘
```

---

## 24. Study Plan

1. **Day 1:** Sections 0–9 (link layer basics, error types, redundancy concept)
2. **Day 2:** Sections 10–15 (block coding, parity in detail, code rate)
3. **Day 3:** Sections 16–20 (CRC + Checksum — these are the high-yield exam topics)
4. **Day 4:** Solve all worked problems in section 22 by hand. Glance at cheat sheet.

If anything's fuzzy, ask:
- "Show me CRC division step-by-step with different numbers"
- "Walk me through Internet checksum with a real example bit-by-bit"
- "Explain why parity misses even-bit errors with a diagram"
- "Quiz me on detection vs correction"
