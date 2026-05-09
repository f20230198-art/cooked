# Tutorial 1 — MCQ Answers (60 Questions)

> **Chapter:** Introduction, OSI Model & Performance (Chapter 1)
> **Reference guide:** [introduction_osi_performance_guide.md](introduction_osi_performance_guide.md)

---

## Quick Answer Key

| Q | Ans | Q | Ans | Q | Ans | Q | Ans | Q | Ans |
|---|-----|---|-----|---|-----|---|-----|---|-----|
| 1 | B | 14 | B | 27 | A | 40 | D | 53 | B |
| 2 | C | 15 | A | 28 | C | 41 | B | 54 | A |
| 3 | B | 16 | C | 29 | A | 42 | A | 55 | C |
| 4 | C | 17 | D | 30 | B | 43 | B | 56 | D |
| 5 | B | 18 | B | 31 | A | 44 | C | 57 | B |
| 6 | C | 19 | C | 32 | D | 45 | C | 58 | D |
| 7 | A | 20 | B | 33 | D | 46 | B | 59 | A |
| 8 | A | 21 | D | 34 | A | 47 | A | 60 | B |
| 9 | A | 22 | B | 35 | C | 48 | B | 61 | C |
| 10 | B | 23 | B | 36 | D | 49 | A |    |    |
| 11 | C | 24 | B | 37 | B | 50 | A |    |    |
| 12 | A | 25 | A | 38 | C | 51 | D |    |    |
| 13 | D | 26 | D | 39 | B | 52 | C |    |    |

---

## Detailed Answers with Explanations

### Section A — Basics (Q1–Q22)

**Q1. The _______ is the physical path over which a message travels.**
**Answer: B) Medium**
> Physical path = transmission medium (wire, fiber, air). Protocol = rules; signal = the encoded data.

**Q2. The information to be communicated in a data communications system is the _______.**
**Answer: C) Message**
> The actual data being communicated is the message (one of the 5 components: sender, receiver, message, medium, protocol).

**Q3. Frequency of failure and network recovery time after a failure are measures of the _______ of a network.**
**Answer: B) Reliability**
> Failure frequency + recovery time = reliability metrics. Performance = throughput/delay; security = unauthorized access.

**Q4. An unauthorized user is a network _______ issue.**
**Answer: C) Security**
> Unauthorized access = security concern by definition.

**Q5. Which topology requires a central controller or hub?**
**Answer: B) Star**
> Star topology has a central hub/switch as the focal point.

**Q6. Which topology requires a multipoint connection?**
**Answer: C) Bus**
> Bus = single shared cable that all devices tap into = multipoint connection.

**Q7. Communication between a computer and a keyboard involves ______________ transmission.**
**Answer: A) Simplex**
> Keyboard → computer is one-way only. The screen → eyes is also simplex.

**Q8. A television broadcast is an example of _______ transmission.**
**Answer: A) Simplex**
> TV station broadcasts to viewers; viewers can't broadcast back.

**Q9. A _______ connection provides a dedicated link between two devices.**
**Answer: A) Point-to-point**
> By definition: point-to-point = exactly two devices, dedicated link.

**Q10. In a _______ connection, more than two devices can share a single link.**
**Answer: B) Multipoint**
> Multipoint = multipoint sharing of one link.

**Q11. In _______ transmission, the channel capacity is shared by both communicating devices at all times.**
**Answer: C) Full-duplex**
> Both transmit simultaneously → channel capacity divided/shared at all times.

**Q12. _______ is the protocol suite for the current Internet.**
**Answer: A) TCP/IP**
> TCP/IP is THE Internet protocol suite.

**Q13. Data flow between two devices can occur in a _______ way.**
**Answer: D) All of the above**
> Three modes possible: simplex, half-duplex, full-duplex.

**Q14. In a ______ connection, two and only two devices are connected by a dedicated link.**
**Answer: B) Point-to-point**
> Definition of point-to-point.

**Q15. In a ________ connection, three or more devices share a link.**
**Answer: A) Multipoint**
> Definition of multipoint.

**Q16. ______ refers to the physical or logical arrangement of a network.**
**Answer: C) Topology**
> Topology = how devices/cables are arranged.

**Q17. Devices may be arranged in a _____ topology.**
**Answer: D) All of the above**
> Mesh, ring, bus (and star, hybrid) are all valid topologies.

**Q18. A _______ is a data communication system within a building, plant, or campus.**
**Answer: B) LAN**
> LAN = Local Area Network (small scope).

**Q19. A ______ is a data communication system spanning states, countries, or the whole world.**
**Answer: C) WAN**
> WAN = Wide Area Network (largest scope).

**Q20. ________ is a collection of many separate networks.**
**Answer: B) An internet**
> "Internet" (lowercase) = network of networks. The capital-I Internet is one specific example.

**Q21. There are ______________ Internet service providers.**
**Answer: D) All of the above**
> ISPs exist at local, regional, national, and international tiers.

**Q22. A ________ is a set of rules that governs data communication.**
**Answer: B) Protocol**
> Protocol = rules. Standard = formal agreement; forum = discussion body.

---

### Section B — OSI / TCP/IP Layers (Q23–Q39)

**Q23. The Internet model consists of _______ layers.**
**Answer: B) Five**
> TCP/IP "Internet model" = 5 layers (Physical, Data Link, Network, Transport, Application). OSI = 7.

**Q24. The process-to-process delivery of the entire message is the responsibility of the _______ layer.**
**Answer: B) Transport**
> Transport layer = process-to-process. Network = host-to-host. Data Link = hop-to-hop.

**Q25. The _______ layer is the layer closest to the transmission medium.**
**Answer: A) Physical**
> Physical layer (Layer 1) sits directly on the wire/medium.

**Q26. Mail services are available to network users through the _______ layer.**
**Answer: D) Application**
> Mail (SMTP, POP3, IMAP) = application layer protocols.

**Q27. As the data packet moves from the upper to the lower layers, headers are _______.**
**Answer: A) Added**
> Encapsulation: each layer adds its own header going DOWN; receiver strips them going UP.

**Q28. The _______ layer lies between the network layer and the application layer.**
**Answer: C) Transport**
> Order: Network (3) → Transport (4) → Session (5) → Presentation (6) → Application (7).

**Q29. Layer 2 lies between the physical layer and the _______ layer.**
**Answer: A) Network**
> Layer 2 (Data Link) sits between Layer 1 (Physical) and Layer 3 (Network).

**Q30. When data are transmitted from device A to device B, the header from A's layer 4 is read by B's _______ layer.**
**Answer: B) Transport**
> Peer-to-peer principle: each layer talks to the same layer on the other side. L4 header is read by L4.

**Q31. The _______ layer changes bits into electromagnetic signals.**
**Answer: A) Physical**
> Physical layer = bit ↔ signal conversion.

**Q32. Which of the following is an application layer service?**
**Answer: D) All the above**
> Remote login (Telnet/SSH), file transfer (FTP), mail (SMTP) — all application layer.

**Q33. The physical layer is concerned with the movement of _______ over the physical medium.**
**Answer: D) Bits**
> Physical layer's PDU = bits.

**Q34. To deliver a message to the correct application program running on a host, the _______ address must be consulted.**
**Answer: A) Port**
> Port number identifies the process/application on a host.

**Q35. IPv6 has _______ -bit addresses.**
**Answer: C) 128**
> IPv6 = 128 bits; IPv4 = 32 bits.

**Q36. ICMPv6 includes _______.**
**Answer: D) a and b (IGMP and ARP)**
> In IPv6, ICMPv6 absorbs the functionality of IGMP and ARP.

**Q37. The ______ layer is responsible for moving frames from one hop (node) to the next.**
**Answer: B) Data Link**
> Data link = hop-to-hop frame delivery; PDU = frame.

**Q38. The ______ layer adds a header to the packet that includes the logical addresses of the sender and receiver.**
**Answer: C) Network**
> Network layer adds the IP (logical) addresses to packets.

**Q39. The _________ layer is responsible for the delivery of a message from one process to another.**
**Answer: B) Transport**
> Process-to-process = Transport (uses port numbers to identify processes).

---

### Section C — Protocols & Addresses (Q40–Q45)

**Q40. The Internetworking Protocol (IP) is a ________ protocol.**
**Answer: D) None of the above**
> IP is **unreliable AND connectionless** — neither (a) reliable nor (b) connection-oriented.

**Q41. _______ is a process-to-process protocol that adds only port addresses, checksum error control, and length information.**
**Answer: B) UDP**
> UDP = lightweight: just ports + length + checksum. TCP adds way more.

**Q42. __________ provides full transport layer services to applications.**
**Answer: A) TCP**
> TCP = reliable, ordered, flow-controlled, congestion-controlled = full service.

**Q43. The ________ address, also known as the link address, is the address of a node as defined by its LAN or WAN.**
**Answer: B) Physical**
> Physical = MAC address = link address.

**Q44. Ethernet uses a ______ physical address that is imprinted on the network interface card (NIC).**
**Answer: C) 6-byte**
> Ethernet MAC = 48 bits = 6 bytes.

**Q45. A port address in TCP/IP is ______ bits long.**
**Answer: C) 16**
> Port = 16 bits (range 0–65535). MAC = 48; IPv4 = 32.

---

### Section D — OSI Model in Depth (Q46–Q59)

**Q46. The ____ created a model called the Open Systems Interconnection.**
**Answer: B) ISO**
> ⚠ **TRAP!** ISO (organization) created OSI (model). Don't confuse them.

**Q47. The seven-layer _____ model provides guidelines for the development of universally compatible networking protocols.**
**Answer: A) OSI**
> The 7-layer model itself is called OSI.

**Q48. The physical, data link, and network layers are the ______ support layers.**
**Answer: B) Network**
> Layers 1–3 handle network-related functions = "network support layers."

**Q49. The session, presentation, and application layers are the ____ support layers.**
**Answer: A) User**
> Layers 5–7 handle user/application-related functions = "user support layers."

**Q50. The _______ layer links the network support layers and the user support layers.**
**Answer: A) Transport**
> Transport (Layer 4) bridges Layers 1–3 and Layers 5–7.

**Q51. The _______ layer coordinates the functions required to transmit a bit stream over a physical medium.**
**Answer: D) Physical**
> Physical layer = bit-stream transmission.

**Q52. The _______ layer is responsible for delivering data units from one station to the next without errors.**
**Answer: C) Data Link**
> Data link = error-free hop-to-hop delivery (uses CRC, ACKs).

**Q53. The ______ layer is responsible for the source-to-destination delivery of a packet across multiple network links.**
**Answer: B) Network**
> Network = source-to-destination (across many hops via routing).

**Q54. The ________ layer is responsible for the process-to-process delivery of the entire message.**
**Answer: A) Transport**
> Transport = process-to-process delivery of the whole message.

**Q55. The ______ layer establishes, maintains, and synchronizes the interactions between communicating devices.**
**Answer: C) Session**
> Session layer = dialog control + synchronization + checkpointing.

**Q56. The _______ layer ensures interoperability through transformation of data into a mutually agreed upon format.**
**Answer: D) Presentation**
> Presentation = format translation, encryption, compression.

**Q57. The _________ layer enables the users to access the network.**
**Answer: B) Application**
> Application layer = user's interface to the network.

**Q58. TCP/IP is a ______ hierarchical protocol suite developed ____ the OSI model.**
**Answer: D) Five-layer; after**
> TCP/IP has 5 layers. (Note: textbook context — TCP/IP design predates OSI's finalization, but it's commonly compared/described AFTER the OSI model in teaching.)

**Q59. The TCP/IP _______ layer is equivalent to the combined session, presentation, and application layers of the OSI model.**
**Answer: A) Application**
> TCP/IP collapses OSI's top 3 layers into one "Application" layer.

---

### Section E — Address Identification (Q60–Q61)

**Q60. The ____ address uniquely defines a host on the Internet.**
**Answer: B) IP**
> IP address = unique host identifier on the Internet.

**Q61. The _____ address identifies a process on a host.**
**Answer: C) Port**
> Port = identifies which process/app within a host.

---

## Common Traps & Things to Remember

| Trap | The fix |
|---|---|
| ISO vs OSI (Q46/47) | **ISO** is the organization; **OSI** is the model they created |
| IP is reliable? (Q40) | NO — IP is **unreliable + connectionless** |
| TCP/IP layer count (Q23/58) | **5 layers** (not 7 like OSI) |
| Process-to-process vs host-to-host | Process = **Transport** (ports); Host = **Network** (IP) |
| Address sizes | MAC = **48 bits** (6 bytes); IPv4 = **32 bits**; IPv6 = **128 bits**; Port = **16 bits** |
| Hop-to-hop vs end-to-end | Data Link = hop; Network = end host; Transport = end process |

---

## Score Tracker

- [ ] Total attempted: ___ / 61
- [ ] Correct: ___
- [ ] Topics to revisit: ___

Cross-reference any wrong answers with [introduction_osi_performance_guide.md](introduction_osi_performance_guide.md) sections 12–18 (OSI) and section 19 (Performance).
