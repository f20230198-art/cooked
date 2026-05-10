# Tutorial 4 — CRC, Checksum, & Packet vs Circuit Switching Efficiency

> **Topics:** CRC encoding/decoding, Internet Checksum, header overhead in circuit vs packet switching
> **Reference guide:** [link_layer_error_detection_guide.md](link_layer_error_detection_guide.md)

---

## 📚 Concepts You Need First

### Concept A — CRC as Polynomial Division

A bit string can be treated as the coefficients of a polynomial.
Example: `1101` = $1·x^3 + 1·x^2 + 0·x + 1 = x^3 + x^2 + 1$

A CRC generator polynomial like $x^3 + 1$ corresponds to the bit pattern `1001`.

**Encoding steps:**
1. Take the data polynomial M(x)
2. Multiply by $x^r$ (where r = degree of generator) — this appends r zero bits
3. Divide by the generator G(x) using **mod-2 (XOR) arithmetic**
4. The remainder R(x) is the CRC
5. Transmit: M(x) + R(x) (data with CRC appended)

**Decoding (at receiver):**
1. Divide received codeword by G(x)
2. Remainder = 0 → no error detected
3. Remainder ≠ 0 → error detected

### Concept B — Internet Checksum (One's Complement)

Used by IP, TCP, UDP. Algorithm:
1. Split data into n-bit words (16-bit usually; this problem uses 4-bit)
2. **Sum all words using one's complement addition** — if sum overflows the n bits, **wrap the carry back into the LSB**
3. Take the **one's complement (flip all bits)** of the final sum → that's the checksum

### Concept C — Why Circuit Switching Saves Bytes vs Packet Switching

In **packet switching**, every packet carries a header → the more packets, the more overhead bytes.
In **circuit switching**, only **setup/teardown messages** carry overhead → file size doesn't add per-byte header tax.

So for a large enough file, **circuits become cheaper in total bytes sent** even though they have a fixed setup cost.

---

# 🧮 PROBLEMS

---

## Q1. CRC Encoding & Error Detection

> Transmit message **`11100011`** using CRC polynomial **$x^3 + 1$**.
>
> (a) Use polynomial long division to find the transmitted message.
> (b) If the leftmost bit gets inverted (noise), what does the receiver compute? How does the receiver detect the error?

### Setup
- Data M = `11100011` (8 bits)
- Generator G = $x^3 + 1$ → bit pattern **`1001`** (4 bits, so r = 3)

### Part (a) — Compute CRC

**Step 1:** Append r = 3 zero bits to data
```
M' = 11100011 000  (11 bits)
```

**Step 2:** Mod-2 long division of `11100011000` by `1001`

```
                1 1 1 1 1 0 1 0
              ─────────────────────
   1 0 0 1 │ 1 1 1 0 0 0 1 1 0 0 0
            1 0 0 1                ← XOR
            ─────────
            0 1 1 1 0 0 1 1 0 0 0
              1 0 0 1              ← XOR
              ─────────
              0 1 1 1 1 1 1 0 0 0
                1 0 0 1            ← XOR
                ─────────
                0 1 1 0 0 1 0 0 0
                  1 0 0 1          ← XOR
                  ─────────
                  0 1 0 1 1 0 0 0
                    1 0 0 1        ← XOR
                    ─────────
                    0 0 1 0 1 0 0
                          (only 4 bits left, but proceed)
                      0 0 0 0      ← skip (leading 0)
                      ─────────
                      1 0 1 0 0
                        1 0 0 1    ← XOR
                        ─────────
                        0 0 1 1 0
                              ↓
                          (continue)
                          1 1 0    ← only 3 bits ← REMAINDER
```

After full division: **Remainder R = `011`** (3 bits)

**Step 3:** Transmitted message = data + CRC

```
Transmitted = 11100011 011  (11 bits)
```

✅ **Answer (a): `11100011011`**

### Part (b) — Receiver detects error

Suppose noise inverts the **leftmost bit**: `11100011011` → **`01100011011`**

**Receiver divides `01100011011` by `1001`:**

```
                0 1 1 0 1 1 0 1
              ─────────────────────
   1 0 0 1 │ 0 1 1 0 0 0 1 1 0 1 1
                  (leading 0, skip)
              1 1 0 0 0 1 1 0 1 1
              1 0 0 1
              ─────────
              0 1 0 1 0 1 1 0 1 1
                1 0 0 1
                ─────────
                0 1 1 1 1 1 0 1 1
                  1 0 0 1
                  ─────────
                  0 1 1 0 1 0 1 1
                    1 0 0 1
                    ─────────
                    0 1 0 0 1 1 1
                      1 0 0 1
                      ─────────
                      0 0 1 0 1
                            ↓
                          1 0 1   ← REMAINDER
```

**Remainder = `101` (≠ 0)**

✅ **Answer (b):** Receiver gets a **non-zero remainder = `101`**. Since the remainder isn't zero, the receiver knows an error occurred and **discards the frame** (or requests retransmission).

### 🔑 Key insight
A correctly-received codeword always divides evenly by G. **Any non-zero remainder = error detected.** A well-chosen G can detect all single-bit errors, all odd-bit-error patterns, and all burst errors of length ≤ r.

---

## Q2. CRC with $x^3 + x^2 + 1$ Generator

> Data = **`01011100`**, generator = $x^3 + x^2 + 1$ → `1101`. Compute CRC bits. Then show that flipping the leftmost bit (giving `11011100`) is detected.

### Setup
- M = `01011100` (8 bits)
- G = `1101` (4 bits, r = 3)

### Step 1 — Append 3 zeros: `01011100 000` (11 bits)

### Step 2 — Mod-2 division of `01011100000` by `1101`

The leading 0 of the dividend contributes a 0 in the quotient and shifts in the next bit, so we effectively divide `1011100000` (10 bits) by `1101`:

```
   1 0 1 1 1 0 0 0 0 0
   1 1 0 1                ← XOR (leading bit was 1)
   ─────────────
   0 1 1 0 1 0 0 0 0 0
     1 1 0 1              ← XOR (next leading bit is 1)
     ─────────────
     0 0 0 0 0 0 0 0 0    ← from here all bits are 0
                            no further XOR needed

   Remainder = 0 0 0  (last 3 bits)
```

✅ **CRC bits = `000`** — the data happens to already be a multiple of G(x).

### Transmitted message: `01011100 000`

> **Sanity check using polynomial arithmetic:**
> M(x)·x³ = x⁹ + x⁷ + x⁶ + x⁵.
> Reducing each term mod G(x) = x³ + x² + 1:
> x⁹ ≡ x², x⁷ ≡ 1, x⁶ ≡ x²+x, x⁵ ≡ x+1.
> Sum ≡ x² + 1 + (x² + x) + (x + 1) = 2x² + 2x + 2 ≡ **0** (mod 2). ✓

### Step 3 — Receiver gets `11011100000` (leftmost flipped)

Flipping bit position 10 adds x¹⁰ to the codeword. Since the original codeword had remainder 0, the new remainder = x¹⁰ mod G(x):

x¹⁰ ≡ x · x⁹ ≡ x · x² = x³ ≡ x² + 1 → bits **`101`**.

So the receiver computes:

```
Remainder = 1 0 1   (≠ 0)  →  error detected, frame discarded
```

✅ **Answer:**
- CRC bits = `000`; transmitted codeword = `01011100000`.
- Leftmost-bit flip → received codeword `11011100000` → receiver gets remainder `101` ≠ 0 → error detected.

### 🔑 Key insight
Any single-bit flip at position k changes the received remainder by $x^k \bmod G(x)$. As long as G(x) has the term `1` (i.e., the constant term is nonzero) and degree ≥ 1, a single $x^k$ on its own is **never** a multiple of G(x), so the syndrome is always nonzero → **all single-bit errors are detected**.

> **Generator polynomial G(x):** the agreed divisor used for CRC; both sender and receiver must use the same one.
> **Syndrome:** the remainder the receiver computes after dividing the received codeword by G(x). Zero → assume no error; nonzero → error detected.

---

## Q3. Internet Checksum (4-bit words)

> Message: **`1001 1100 1010 0011`** (16 bits, four 4-bit words). Find the checksum.

### Setup
4 words of 4 bits each:
- W1 = `1001` (= 9)
- W2 = `1100` (= 12)
- W3 = `1010` (= 10)
- W4 = `0011` (= 3)

### Step 1 — One's complement sum of all words

In 4-bit one's complement, max value before wrap = 15 (`1111`).

```
  1001
+ 1100
─────
 10101  (5 bits — overflow!)
```

**Wrap carry:** `0101 + 1 = 0110`

So `1001 + 1100 = 0110` (in one's complement 4-bit)

```
  0110
+ 1010
─────
 10000  (overflow!)
```

Wrap: `0000 + 1 = 0001`

So running sum = `0001`

```
  0001
+ 0011
─────
  0100   (no overflow)
```

**Final sum = `0100`**

### Step 2 — Take one's complement (flip all bits)

```
Sum:     0100
Flipped: 1011
```

✅ **Checksum = `1011`**

### Verification (receiver side)
Receiver adds all 4 data words + checksum:
```
  1001 + 1100 + 1010 + 0011 + 1011
= 0100 (running) + 1011
= 1111
```

One's complement of `1111` = `0000` → ✓ no error.

> **One's complement (of an n-bit number):** flip every bit. Used so that "subtraction" can be expressed as addition + bit-flip — that's why the receiver's check is "sum everything; result should be all-zeros after flipping."
> **Carry-around (end-around carry):** when an n-bit one's-complement addition overflows, the carry bit is added back into the LSB. That's what makes the operation closed within n bits.

---

## Q4. Circuit vs Packet Switching — When Are Circuits More Efficient (in Bytes)?

> Path: source → 7 point-to-point links → 5 switches → destination.
> - Each link: 2 ms propagation, 4 Mbps bandwidth
> - Switches support both circuit + packet switching
> - **Packet mode:** 24-byte header, 1000-byte payload per packet → packet size = **1024 bytes**
> - **Store-and-forward** processing at each switch: 1 ms after fully receiving
> - Packets sent continuously (no ACK wait)
> - **Circuit setup:** 1-KB message makes one round-trip on the path, with 1 ms delay at each switch after receiving
>
> (a) For what file size n bytes is the **total bytes sent** across the network LESS for circuits than for packets?

### Setup
- Packet mode overhead per packet = **24 bytes header / 1000 bytes payload**
- Circuit setup overhead = **1 KB round-trip** = setup message goes forward + comes back = **2 × 1024 = 2048 bytes**

### Part (a) — Bytes sent comparison

**Bytes sent for packet switching** (n-byte file, 1000 bytes payload per packet):

Number of packets = n / 1000

Total bytes per packet = 24 (header) + 1000 (payload) = 1024

So total bytes transmitted = (n/1000) × 1024 = **1.024n**

**Bytes sent for circuit switching** (n-byte file):

- Setup overhead = 2048 bytes (round-trip 1-KB setup message)
- File transmitted as one contiguous bit stream = exactly n bytes (no per-packet headers)
- Total = **n + 2048 bytes**

### Find the crossover point

Circuits use fewer bytes when:
$$n + 2048 < 1.024 n$$
$$2048 < 0.024 n$$
$$n > \frac{2048}{0.024} = 85{,}333 \text{ bytes}$$

Round to nearest multiple of 1000 (since file size is a multiple of 1000):

$$n \geq 86{,}000 \text{ bytes}$$

### Verification
- At n = 86,000 bytes:
  - Packets: 1.024 × 86,000 = 88,064 bytes
  - Circuit: 86,000 + 2048 = 88,048 bytes
  - Circuit wins by 16 bytes ✓
- At n = 85,000 bytes:
  - Packets: 1.024 × 85,000 = 87,040 bytes
  - Circuit: 85,000 + 2048 = 87,048 bytes
  - Packets win by 8 bytes ✓

✅ **Answer: For files of size n ≥ 86,000 bytes, circuit switching transmits fewer total bytes.**

### 🔑 Key insight
Circuit switching has a fixed setup cost (~2 KB) but **zero per-byte overhead** during data transfer. Packet switching has no setup cost but pays **2.4% overhead per byte** (24 bytes header / 1000 bytes payload).

So:
- **Small files** → packets win (no setup waste)
- **Large files** → circuits win (overhead amortizes to nothing)
- **Crossover at ~85 KB**

---

## Q5. IPv4 Header Checksum, TTL & Fragment-Offset Corruption

> Five-node path N1→N2→N3→N4→N5. Node 1 builds the IPv4 header below (bold values are initialized; checksum is to be computed). All answers in **hex**.
>
> ```
> Version=4  IHL=4  ToS=0      Total Length=24
>      Identification=1024    Flags=0  Fragment Offset=0
>      TTL=5      Protocol=6        Header Checksum=?
>          Source Address = 20.7.8.7
>          Destination Address = 47.5.5.8
> ```
>
> (a) Compute the header checksum at Node 1.
> (b) Packet arrives error-free at Node 3. Between N3 and N4 the **LSB of the fragment-offset field** flips. Show how Node 4 detects this.
> (c) For a longer path N1→N2→N3→N4→N5→N6 with no errors, what happens at Node 6?

### Setup — header as ten 16-bit words

The IPv4 header is 20 bytes = ten 16-bit words. Field-pack the given values (20 dec = 0x14, 47 dec = 0x2F):

| # | Field(s) | Hex word |
|---|---|---|
| 1 | Version, IHL, ToS | **`4500`** |
| 2 | Total Length (24) | **`0018`** |
| 3 | Identification (1024) | **`0400`** |
| 4 | Flags + Fragment Offset | **`0000`** |
| 5 | TTL (5), Protocol (6) | **`0506`** |
| 6 | Header Checksum | `????` (treat as 0 while computing) |
| 7 | Src high (20.7) | **`1407`** |
| 8 | Src low (8.7) | **`0807`** |
| 9 | Dst high (47.5) | **`2F05`** |
| 10 | Dst low (5.8) | **`0508`** |

### Part (a) — Compute Node 1's checksum

**Step 1:** One's-complement sum of the 9 non-checksum words.

```
4500 + 0018 = 4518
4518 + 0400 = 4918
4918 + 0000 = 4918
4918 + 0506 = 4E1E
4E1E + 1407 = 6225
6225 + 0807 = 6A2C
6A2C + 2F05 = 9931
9931 + 0508 = 9E39      ← no 17-bit carry anywhere
```

Sum = **`0x9E39`**.

**Step 2:** Take the one's complement (flip all 16 bits).

```
9E39 = 1001 1110 0011 1001
~    = 0110 0001 1100 0110  = 0x61C6
```

✅ **Answer (a): Header checksum = `0x61C6`**

### Part (b) — Node 4 detects a flipped LSB in Fragment Offset

After N3, the Flags+FragOffset word changes from `0000` → `0001` (LSB flipped).

Node 4 sums **all ten** words (including the received checksum) and checks whether the complement is zero.

- Sum of 9 non-checksum words (with the bit flip): `9E39 + 1 = 9E3A`
- Add the received checksum: `9E3A + 61C6 = 1 0000` → wrap carry → **`0001`**
- One's complement: `~0001 = ` **`0xFFFE`** ≠ 0 → **ERROR**

(For comparison: with no errors, the same calculation yields `9E39 + 61C6 = FFFF` → complement = 0 → OK.)

✅ **Answer (b):** The complement of Node 4's sum is `0xFFFE` instead of `0x0000`, so the checksum check fails. **Node 4 silently discards the packet.** (IP itself does not retransmit; recovery is left to higher layers like TCP.)

### Part (c) — Packet reaches Node 6

Every router that *forwards* an IPv4 packet decrements **TTL by 1** and recomputes the checksum. Starting from TTL = 5 at N1:

```
N1 sends with TTL=5
N2 → decrement → TTL=4, forward
N3 → decrement → TTL=3, forward
N4 → decrement → TTL=2, forward
N5 → decrement → TTL=1, forward
N6 → decrement → TTL=0  ← STOP
```

When a router's decrement produces TTL = 0, the packet **must be discarded** and the router sends an **ICMP "Time Exceeded" (Type 11)** message back to the source IP `20.7.8.7`.

✅ **Answer (c):** Node 6 drops the packet (TTL expired) and sends an **ICMP Time Exceeded** message back to source `20.7.8.7`. The packet never reaches destination `47.5.5.8`.

### 🔑 Key insights

- The IPv4 checksum covers **only the header**, not the payload. Recomputed at every hop because TTL changes each time.
- A single bit flip anywhere in the header almost always changes the sum, so the checksum reliably catches single-bit header errors.
- TTL is the loop-prevention mechanism. `traceroute` exploits the ICMP Time Exceeded reply by sending probes with increasing TTLs (1, 2, 3, …) to reveal each hop along the path.

> **TTL (Time To Live):** an 8-bit hop counter in the IPv4 header. Every forwarding router decrements it; when it hits 0 the packet is dropped to prevent infinite routing loops.
> **ICMP Time Exceeded (Type 11):** the diagnostic message a router returns to the source when it drops a packet because TTL reached 0.
> **Fragment Offset:** a 13-bit field saying where this fragment's data sits inside the original (pre-fragmentation) datagram, in units of 8 bytes.
> **IHL (Internet Header Length):** the header length in 32-bit words. IHL = 4 ⇒ 20-byte header (no options).

---

## 📊 Summary

| Q | Topic | Key result |
|---|---|---|
| 1 | CRC encoding | Codeword = `11100011011`; flipped bit → non-zero remainder = detected |
| 2 | CRC encoding + detection | CRC = `000`; codeword = `01011100000`; flipped-bit syndrome = `101` ≠ 0 → detected |
| 3 | Internet checksum | Checksum of `1001 1100 1010 0011` = `1011` |
| 4 | Bytes overhead | Circuits beat packets in bytes when file ≥ 86 KB |
| 5 | IPv4 header checksum + TTL | (a) `0x61C6` (b) sum complement = `0xFFFE` → drop (c) TTL=0 at N6 → ICMP Time Exceeded |

---

## 🔑 Formula Cheat Sheet

```
┌─────────────────────────────────────────────────────────────┐
│  CRC ENCODING                                                │
│  1. Append r zeros to data (r = degree of generator)         │
│  2. XOR-divide by generator polynomial                       │
│  3. Append remainder as CRC                                  │
│  Codeword = Data || CRC                                      │
├─────────────────────────────────────────────────────────────┤
│  CRC DECODING                                                │
│  Divide received codeword by G                               │
│  Remainder = 0 → OK; remainder ≠ 0 → ERROR                   │
├─────────────────────────────────────────────────────────────┤
│  INTERNET CHECKSUM (n-bit words)                             │
│  1. Sum all words (one's complement: wrap carry to LSB)      │
│  2. Flip all bits of the sum → checksum                      │
│  Receiver sums all + checksum; flips → must equal 0          │
├─────────────────────────────────────────────────────────────┤
│  CIRCUIT vs PACKET BYTE OVERHEAD                             │
│  Packet bytes = (1 + h/p) × n     [h=header, p=payload]      │
│  Circuit bytes = n + 2 × setup_msg_size                      │
│  Crossover: n > 2·setup / (h/p)                              │
└─────────────────────────────────────────────────────────────┘
```

---

If your CRC division gives a different remainder, redo the long-division carefully (or verify with the polynomial-arithmetic shortcut shown in Q2's sanity check) — the **method** matters more than the final 3 bits.
