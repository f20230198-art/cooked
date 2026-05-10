# Previous Year Questions — Fully Worked Solutions

> **Coverage:** Queuing & Little's Law, circuit vs packet switching, Slotted ALOHA capacity, Internet checksum, CRC, CSMA/CD timing & min-frame sizing, CSMA/CA relay throughput.
> **Reference guides:**
> - [introduction_osi_performance_guide.md](introduction_osi_performance_guide.md) — delay/throughput basics
> - [link_layer_error_detection_guide.md](link_layer_error_detection_guide.md) — checksum, CRC
> - [multiple_access_protocols_guide.md](multiple_access_protocols_guide.md) — ALOHA, CSMA/CD, CSMA/CA

---

## 📚 Concepts You'll Need

### Concept A — Little's Law (the one queueing formula you need)

For any stable queue:

$$\boxed{L = \lambda \cdot W}$$

where
- **L** = average number of items in the queue (or system)
- **λ** = average arrival rate (items/sec)
- **W** = average time an item spends waiting (sec)

Use it both ways: if you know the load and waiting time → get queue length; if you know queue length and rate → get waiting time.

### Concept B — Packet Loss Rate

If sender pushes data in at rate λ_in and the link/receiver can absorb only λ_out, the **loss rate** is:

$$\text{Loss rate} = \frac{\lambda_{in} - \lambda_{out}}{\lambda_{in}}$$

### Concept C — Store-and-Forward Pipelining

For N packets traversing H links with H−1 store-and-forward switches:

$$T_{total} = (N + H - 1) \cdot T_{trans} + H \cdot T_{prop} + (H-1) \cdot T_{switch}$$

The "+ (H−1)" captures pipeline fill: the first packet fills the pipeline; the rest follow one transmission-time apart on the slowest link.

### Concept D — Slotted ALOHA Stability

Throughput $S = G e^{-G}$, peaks at G = 1. If each station offers load g, total offered load is G = n·g. For the system to operate at peak, **n·g ≤ 1** (G ≤ 1) — that gives the cap on station count.

### Concept E — Internet Checksum (one's complement, 16-bit)

1. Break message into 16-bit words.
2. Sum them; whenever a 17-bit carry appears, add the carry back into the LSB (end-around carry).
3. Take the bitwise complement of the final sum → that's the checksum.

### Concept F — CSMA/CD Worst-Case Collision Detection

A station has fully "captured" the medium only after it has been transmitting for **2·T_p** (round-trip propagation across the longest link). Before that, a collision started at the far end can still come back and surprise it.

$$\boxed{L_{min} \geq 2 \cdot T_p \cdot R}$$

### Concept G — CSMA/CA + RTS/CTS Single-Hop Cost

One full DCF exchange (per hop):

$$T_{hop} = DIFS + T_{RTS} + T_p + SIFS + T_{CTS} + T_p + SIFS + T_{DATA} + T_p + SIFS + T_{ACK} + T_p$$

For multi-hop relay (e.g., C→A→E), repeat the full cost per hop.

---

# 🧮 PROBLEMS

---

## Q1. Queueing on the Dubai → Abu Dhabi Path (Little's Law)

> Packets of 1000 bits each, end-to-end delay 50 ms with no queue, 125 ms with heavy congestion, **average delay 75 ms**. Sender pushes at 1 Mbps; receiver gets them at 1 Mbps without loss.
>
> (a) Average number of packets in the queue at the bottleneck switch.
> (b) Transmission rate raised to 2 Mbps. Receiver still gets only 1.6 Mbps. Avg queue length unchanged.
>   (i) Packet-loss rate at the switch.
>   (ii) Current average one-way delay.

### Setup

- Packet size L = 1000 bits.
- No-queue delay = 50 ms = **propagation + transmission** components (fixed, distance-dominated).
- Average **queuing delay** alone = avg total − no-queue floor = 75 − 50 = **25 ms**.
- Arrival rate (= delivered rate, since no loss): λ = 1 Mbps / 1000 bits = **1000 pkts/s**.

### Part (a) — Average packets in queue

Apply Little's Law to the queue:

$$L_q = \lambda \cdot W_q = 1000 \text{ pkt/s} \times 0.025 \text{ s} = \mathbf{25 \text{ packets}}$$

✅ **Answer (a): ≈ 25 packets in the queue.**

### Part (b)(i) — Loss rate when sender = 2 Mbps, receiver = 1.6 Mbps

Arrivals at the switch: λ_in = 2 Mbps / 1000 b = **2000 pkts/s**.
Departures (= delivered to receiver): λ_out = 1.6 Mbps / 1000 b = **1600 pkts/s**.

$$\text{Loss rate} = \frac{\lambda_{in}-\lambda_{out}}{\lambda_{in}} = \frac{2000-1600}{2000} = 0.20 = \mathbf{20\%}$$

(Equivalently, **400 pkts/s dropped**.)

### Part (b)(ii) — Current average one-way delay

Apply Little's Law again, but now the **throughput** (departure rate) sets the queue's clearance speed:

$$W_q = \frac{L_q}{\lambda_{out}} = \frac{25}{1600} = 0.015625 \text{ s} \approx 15.6 \text{ ms}$$

Total one-way delay = no-queue floor (50 ms) + new queuing delay:

$$W_{total} = 50 + 15.6 \approx \mathbf{65.6 \text{ ms}}$$

(Compared to 75 ms before — the delay actually dropped because, even though arrivals doubled, the surplus is *dropped* rather than queued, and the same 25-packet queue empties faster at the slightly higher departure rate of 1.6 Mbps vs 1.0 Mbps.)

✅ **Answers (b):** (i) **20% loss**, (ii) **~65.6 ms**.

> **Little's Law:** for any stable system, average # of items = arrival rate × average wait.
> **Bottleneck:** the slowest link/switch on the path; queue forms here when arrivals exceed its service rate.
> **End-to-end delay:** sum of propagation + transmission + processing + queuing along the entire path.

---

## Q2. Circuit vs Packet Switching for a 10 MB File

> Path: source → 4 links → 3 switches → destination.
> Each link: prop = 4 ms, BW = 10 Mbps.
> **Packet mode:** 50 B header + 1500 B payload per packet, switches add 2 ms store-and-forward processing per packet, no ACK waits.
> **Circuit mode:** 1 KB setup msg makes a round-trip; each switch adds 1 ms after receiving the setup; once circuit is up, file flows as continuous bitstream with no further switch delay.
>
> File size = 10 MB. Compute total time for each.

### Common numbers

- 10 MB = 10 × 10⁶ B = **8 × 10⁷ bits** (decimal MB; using SI throughout)
- 1 KB setup = 8000 bits
- BW = 10⁷ bps; prop = 4 ms/link; 4 links; 3 switches

### Packet switching

**Per packet:** L = (50 + 1500) × 8 = **12,400 bits** → T_trans = 12,400 / 10⁷ = **1.24 ms**

**Number of packets:** N = (10 × 10⁶) / 1500 ≈ 6666.67 → **6667 packets** (last one is partial; round up).

**Pipeline formula** (H = 4 links, H − 1 = 3 switches):

$$T_{total} = (N + H - 1)\cdot T_{trans} + H \cdot T_{prop} + (H-1)\cdot T_{switch}$$

$$= (6667 + 3)\times 1.24 + 4\times 4 + 3\times 2$$

$$= 6670 \times 1.24 + 16 + 6$$

$$= 8270.8 + 22 = \mathbf{8292.8 \text{ ms} \approx 8.29 \text{ s}}$$

### Circuit switching

**Setup:** the 1 KB setup msg traverses 4 links (with 1 ms processing per switch on the way) and comes back.

- T_trans of setup msg per link = 8000 / 10⁷ = 0.8 ms
- One-way time = 4 × 0.8 + 4 × 4 + 3 × 1 = 3.2 + 16 + 3 = **22.2 ms**
- Round-trip setup = 2 × 22.2 = **44.4 ms**

**File transmission** (continuous bitstream, no switch delay during data phase):

- T_trans of whole file on one link = 8 × 10⁷ / 10⁷ = **8000 ms = 8 s**
- The last bit still has to propagate across 4 links: 4 × 4 = **16 ms**
- Data phase = 8000 + 16 = **8016 ms**

**Total:**

$$T_{circuit} = 44.4 + 8016 = \mathbf{8060.4 \text{ ms} \approx 8.06 \text{ s}}$$

### Comparison

| Mode | Total time |
|---|---|
| Packet switching | ≈ **8.29 s** |
| Circuit switching | ≈ **8.06 s** |

Circuit switching wins here (saves ~230 ms): the setup cost is small compared to the file size, and packet mode pays a per-packet header tax (1550/1500 = 3.3 % overhead) plus 3 × 2 = 6 ms of switch processing.

---

## Q3. Maximum n in a Slotted ALOHA System

> n stations, each sends one 512-byte frame every 10 s on average. Channel rate = 156 Kbps. Find max n.

### Setup

- Frame L = 512 × 8 = **4096 bits**
- Channel R = 156 × 10³ bps
- Frame-time T_fr = 4096 / 156,000 ≈ **26.26 ms** = 0.02626 s

### Offered load G (in frames per frame-time)

Each station offers 1 frame / 10 s = 0.1 fps. In units of frames per **frame-time**:

$$g_{per\,station} = 0.1 \times T_{fr} = 0.1 \times 0.02626 = 0.002626$$

Total offered load $G = n \cdot g_{per\,station}$.

### Slotted ALOHA stability

Slotted ALOHA throughput $S = G e^{-G}$ is maximised at **G = 1**. Beyond G = 1 the curve falls (collisions dominate). For the system to stay in the usable region:

$$G = n \cdot 0.002626 \leq 1$$

$$\boxed{n \leq \frac{1}{0.002626} \approx 380.8}$$

✅ **Answer: n_max = 380 stations** (the largest integer keeping G ≤ 1, hence the system in its peak-throughput regime).

> **Frame-time (T_fr):** time to transmit one frame at the channel rate. The natural "clock tick" for ALOHA-family analysis.
> **Offered load G:** total transmission attempts (incl. retransmissions) per frame-time across all stations.

---

## Q4. Internet Checksum for "BITSPILANI"

> Each character is 8 bits (given in the table). Compute the Internet checksum.

### Step 1 — pair up bytes into 16-bit words

BITSPILANI = 10 characters = 80 bits = **five 16-bit words**.

| # | Pair | Hex |
|---|---|---|
| 1 | B I | `01000010 01001001` = **`4249`** |
| 2 | T S | `01010100 01010011` = **`5453`** |
| 3 | P I | `01010000 01001001` = **`5049`** |
| 4 | L A | `01001100 01000001` = **`4C41`** |
| 5 | N I | `01001110 01001001` = **`4E49`** |

### Step 2 — one's-complement sum

```
4249 + 5453 = 969C            (no carry)
969C + 5049 = E6E5            (no carry)
E6E5 + 4C41 = 1 3326          (carry-out → wrap)
              3326 + 1 = 3327
3327 + 4E49 = 8170            (no carry)
```

Final sum = **`0x8170`**.

### Step 3 — bitwise complement

```
8170 = 1000 0001 0111 0000
~    = 0111 1110 1000 1111  = 0x7E8F
```

✅ **Checksum = `0x7E8F`** (binary `0111 1110 1000 1111`).

### Verification

Receiver sums all 5 data words + checksum:

```
8170 + 7E8F = FFFF
~FFFF = 0000  →  ✓ no error
```

---

## Q5. CRC of `01011100` with Generator `10010`

> Find the CRC bits. Then show that flipping the LSB of the data word (so received `01011101`) is detected.

### Setup

- Data M = `01011100` (8 bits)
- G = `10010` (5 bits, so r = 4)
- Generator polynomial G(x) = x⁴ + x

### Step 1 — Append 4 zeros and divide

Dividend: `01011100 0000` (12 bits). Use mod-2 long division by `10010`:

```
   0 1 0 1 1 1 0 0 0 0 0 0
   (leading 0 → skip)
     1 0 1 1 1 0 0 0 0 0 0
     1 0 0 1 0                ← XOR
     ─────────────
     0 0 1 0 1 0 0 0 0 0 0
         1 0 0 1 0            ← XOR (next leading 1)
         ─────────────
         0 0 1 1 0 0 0 0 0
             1 0 0 1 0        ← XOR
             ─────────────
             0 1 0 1 0 0 0
               1 0 0 1 0      ← XOR
               ─────────────
               0 1 0 0 0
               
   Remainder = 1 0 0 0   (last 4 bits)
```

**Cross-check via polynomial reduction** (x⁴ ≡ x, x⁵ ≡ x², x⁶ ≡ x³, x⁷ ≡ x, x⁸ ≡ x², x⁹ ≡ x³, x¹⁰ ≡ x, …):

M(x)·x⁴ corresponds to x¹⁰ + x⁸ + x⁷ + x⁶ (data bits at positions 6,4,3,2 shifted up by 4).

Reduce: x¹⁰ + x⁸ + x⁷ + x⁶ ≡ x + x² + x + x³ = **x³** = bits **`1000`**. ✓

✅ **CRC = `1000`**

### Transmitted codeword

```
01011100 1000
```

### Step 2 — Error detection

If only the LSB of the data flips: received word's data half = `01011101` (and CRC half is unchanged `1000`). The error vector is `00000001 0000` = x⁴.

Syndrome = x⁴ mod G(x):

$$x^4 \bmod (x^4 + x) \equiv x \quad = \quad \mathtt{0010}$$

**Syndrome = `0010` ≠ 0** → error detected → receiver discards the frame.

✅ **Answer:** CRC = **`1000`**; codeword = `010111001000`; the bit flip yields a nonzero syndrome (`0010`) so the receiver catches it.

> **Generator polynomial G(x):** the agreed divisor. Both ends must use the same one.
> **Syndrome:** what the receiver actually computes — the remainder after dividing the received codeword by G(x). Zero ⇒ assume OK; nonzero ⇒ error.

---

## Q6. CSMA/CD Collision-Detection Window (8 Mbps, 4 km cable)

> BW = 8 Mbps; cable = 4 km; signal speed = 200,000 km/s; frame = 5120 bytes. Station starts transmitting at time T. Can the station conclude "no collision" at:
> (a) T + 11 µs
> (b) T + 28 µs
> (c) T + 42 µs
>
> Justify (no marks for direct answers).

### Key numbers

- One-way propagation: $T_p = \dfrac{4000\text{ m}}{2 \times 10^8\text{ m/s}} = \mathbf{20\,\mu s}$
- Round-trip / worst-case collision-detection window: $2 T_p = \mathbf{40\,\mu s}$
- Frame transmission time: $T_{fr} = \dfrac{5120 \times 8}{8\times 10^6} = \dfrac{40960}{8\times 10^6} = \mathbf{5120\,\mu s}$ (so the station is *still* transmitting at all three checkpoints — that's good, CSMA/CD requires it)

### The rule

A CSMA/CD station can declare "I have the channel" **only after 2T_p have elapsed without detecting a collision**. Before then, a colliding signal launched at the far end of the cable just before the local frame arrived can still come back and surprise the sender.

### Part (a) — T + 11 µs

11 µs < 2T_p (40 µs). **No.** A collision could have happened anywhere along the cable as recently as the past 11 µs, and its return signal hasn't reached the sender yet. The station cannot declare success.

✅ **(a) No** — too early; insufficient time for a worst-case far-end collision to be observed.

### Part (b) — T + 28 µs

Still 28 µs < 40 µs = 2T_p. **No.** A collision originating at the far end of the cable just before the local frame arrived there (which would have been at t ≈ 20 µs) would only return at t = 40 µs. At t = 28 µs we still haven't reached that worst-case round-trip.

✅ **(b) No** — still inside the vulnerability window.

### Part (c) — T + 42 µs

42 µs > 40 µs = 2T_p. **Yes.** The round-trip window has passed; if no collision was detected by t = 40 µs none can now appear. The station has effectively "captured" the channel and may safely continue transmitting the remainder of its 5120-µs frame.

✅ **(c) Yes** — past the 2T_p mark, no collision can still be lurking.

### Take-away

CSMA/CD's worst-case detection cost is fixed by physics (2T_p), not by frame size. That's why the **minimum frame size** is engineered so that $T_{trans} \geq 2T_p$ — exactly satisfied here: 5120 µs ≫ 40 µs.

---

## Q7. CSMA/CD Minimum Frame Size — Star vs Bus (15 Mbps)

> 15 Mbps CSMA/CD; signal speed = 2×10⁸ m/s; 8 computers; frame size today = 300 bits.

### (a) Star topology, each PC connected to a hub by a **2000 m** cable

Worst-case round-trip = farthest pair = **PC → hub → other PC** = 2000 m + 2000 m = 4000 m one-way.

$$T_p = \frac{4000}{2\times 10^8} = 20\,\mu s$$

$$L_{min} = 2 T_p \cdot R = 2 \times 20 \times 10^{-6} \times 15 \times 10^6 = \mathbf{600 \text{ bits}}$$

✅ **(a) Min frame = 600 bits.** The given 300-bit frames are **too small** — CSMA/CD would fail to detect some collisions on this network.

### (b) Bus topology, 8 PCs spaced **2000 m apart**

Worst-case is the two end PCs. Distance between them = 7 gaps × 2000 m = **14,000 m**.

$$T_p = \frac{14000}{2\times 10^8} = 70\,\mu s$$

$$L_{min} = 2 \times 70 \times 10^{-6} \times 15 \times 10^6 = \mathbf{2100 \text{ bits}}$$

✅ **(b) Min frame = 2100 bits.**

### Why the bus is worse

A bus's "effective diameter" is the *full* end-to-end length. A hub-star's effective diameter is **2 × (hub-to-host)**, so even with the same per-cable length the worst pair is much closer.

> **Star topology (with hub):** every host wires to a central device; worst-case round-trip = 2 × longest spoke.
> **Bus topology:** all hosts tap into one shared cable; worst-case round-trip = 2 × total cable length.

---

## Q8. CSMA/CA Multi-Hop Relay C → A → E (RTS/CTS)

> 5 wireless nodes laid out so that any pair of adjacent nodes are in range and non-adjacent pairs are not. C and E are non-adjacent, so data must be relayed via the intermediate adjacent node A: **C → A → E**.
>
> All nodes run CSMA/CA with RTS/CTS.
>
> | Quantity | Value |
> |---|---|
> | RTS | 20 B |
> | CTS | 14 B |
> | ACK | 14 B |
> | DATA | 1460 B |
> | DIFS | 52 µs |
> | SIFS | 52 µs |
> | Link length (one hop) | 200 m |
> | Capacity | 10 Mbps |
> | Signal speed | 3×10⁸ m/s |
>
> Compute total time to deliver a single frame from C to E.

### Step 1 — transmission times at 10 Mbps

| Frame | Bits | T = bits / 10⁷ |
|---|---|---|
| RTS  (20 B) | 160    | **16 µs** |
| CTS  (14 B) | 112    | **11.2 µs** |
| DATA (1460 B) | 11,680 | **1168 µs** |
| ACK  (14 B) | 112    | **11.2 µs** |

### Step 2 — propagation per hop

$$T_p = \frac{200}{3\times 10^8} = 0.667\,\mu s$$

### Step 3 — cost of one full DCF + RTS/CTS exchange (per hop)

```
DIFS → RTS → T_p → SIFS → CTS → T_p → SIFS → DATA → T_p → SIFS → ACK → T_p
```

$$T_{hop} = DIFS + T_{RTS} + SIFS + T_{CTS} + SIFS + T_{DATA} + SIFS + T_{ACK} + 4 T_p$$

Plug in:

```
T_hop = 52 + 16 + 52 + 11.2 + 52 + 1168 + 52 + 11.2 + 4(0.667)
      = 52 + 16 + 52 + 11.2 + 52 + 1168 + 52 + 11.2 + 2.667
      = 1417.07 µs
```

### Step 4 — two hops (C→A and A→E)

The relay node A must run a *full* exchange a second time to forward the frame onward; the two hops are sequential.

$$T_{total} = 2 \times T_{hop} = 2 \times 1417.07 \approx \mathbf{2834.13\,\mu s} \approx \mathbf{2.83 \text{ ms}}$$

✅ **Answer: ≈ 2834 µs (2.83 ms) end-to-end.**

### Why the propagation looks tiny

Each link is just 200 m → 0.667 µs each way. The cost is dominated by the **MAC overhead** (3× SIFS + DIFS + 4 short control frames per hop = ~250 µs) and **DATA transmission** (1168 µs per hop). The propagation contribution is < 0.2 % of total.

> **DIFS / SIFS:** inter-frame spaces. DIFS is the larger wait before starting a new transmission; SIFS is the shorter gap inside one exchange so the ACK always has priority.
> **RTS/CTS handshake:** small control frames so that everyone near the *receiver* (which may be hidden from the sender) learns to stay silent during the DATA exchange.
> **Relay / multi-hop:** the originating node and the intermediate node each run their own DCF cycle — there's no single end-to-end ACK.

---

## 📊 Master Summary

| Q | Topic | Headline answer |
|---|---|---|
| 1 | Little's Law / queueing | 25 packets; 20 % loss; ~65.6 ms |
| 2 | Circuit vs packet switch | Packet ≈ 8.29 s; circuit ≈ 8.06 s — circuit wins for a 10 MB file |
| 3 | Slotted ALOHA capacity | n_max ≈ 380 stations |
| 4 | Internet checksum | `0x7E8F` |
| 5 | CRC | `1000`; LSB flip → syndrome `0010` ≠ 0 → detected |
| 6 | CSMA/CD detection window | (a) no, (b) no, (c) yes — boundary is 2T_p = 40 µs |
| 7 | CSMA/CD min frame | Star: 600 bits; Bus: 2100 bits — 300-bit frames are too short for both |
| 8 | CSMA/CA two-hop relay | ≈ 2.83 ms end-to-end |

---

## 🔑 Formula Quick-Reference

```
┌──────────────────────────────────────────────────────────────┐
│  LITTLE'S LAW          L = λ·W                                │
│  LOSS RATE             (λ_in − λ_out) / λ_in                  │
│  STORE-AND-FORWARD     T = (N+H−1)·T_trans + H·T_p + (H−1)·T_sw│
│  SLOTTED ALOHA CAP     n·g ≤ 1 (peak throughput at G = 1)     │
│  CHECKSUM              ~( Σ words, with end-around carry )    │
│  CRC SYNDROME          received_codeword mod G(x)             │
│  CSMA/CD MIN FRAME     L_min ≥ 2·T_p·R                        │
│  CSMA/CA PER HOP       DIFS + RTS + 3·SIFS + CTS + DATA + ACK │
│                          + 4·T_p                              │
└──────────────────────────────────────────────────────────────┘
```

If any step is unclear (especially Q1's two interpretations of "average delay" or Q8's per-hop accounting), point at which line and I'll expand.
