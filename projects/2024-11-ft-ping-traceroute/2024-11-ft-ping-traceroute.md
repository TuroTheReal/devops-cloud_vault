# ft_ping & ft_traceroute - Learning Report

## 📋 Metadata

```yaml
tags: [project, networking, c, icmp, layer3, raw-sockets, 2024-11]
duration: ~90h (estimated across both projects)
status: completed
```

**Technologies**: C, Raw Sockets, ICMP Protocol, Linux Networking
**Goal**: Implement ping and traceroute from scratch to deeply understand Layer 3 networking
**Repos**:
- [ft_ping](https://github.com/TuroTheReal/ft_ping)
- ft_traceroute (42 School project - private)

---

## 🎯 Project Context

### Objective
Recreate the famous `ping` and `traceroute` network diagnostic tools from scratch in C. These are two of the most fundamental tools in network administration, used worldwide to diagnose connectivity issues and understand network topology.

**Why this matters for DevOps**:
- Understanding Layer 3 (Network layer) of OSI model
- ICMP protocol is essential for troubleshooting cloud/container networking
- Foundation for understanding VPC routing, Kubernetes pod networking, Docker overlay networks

### Architecture

```
Application (ft_ping/ft_traceroute)
          ↓
    Raw Socket (SOCK_RAW)
          ↓
    ICMP Protocol (Layer 3)
          ↓
    IP Layer (Layer 3)
          ↓
    Network Interface (eth0, wlan0)
          ↓
    Physical Network
```

**Key concept**: Bypassing OS network stack to craft custom ICMP packets directly.

### Tech Stack
- **Language**: C (low-level network programming)
- **Protocols**: ICMP (Internet Control Message Protocol), IP
- **System**: Raw sockets (requires root privileges)
- **DNS**: getaddrinfo() for hostname resolution
- **Timing**: gettimeofday() for microsecond precision
- **I/O**: select() for non-blocking socket operations

---

## 📚 What I Learned

### 1. OSI Layer 3 (Network Layer) - ICMP Protocol

- **Difficulty**: ⭐⭐⭐⭐ (4/5)
- **Time**: 35h
- **Key insight**:
  - **ICMP** sits on top of IP but is part of Layer 3 (not Layer 4 like TCP/UDP)
  - ICMP types: Echo Request (8), Echo Reply (0), Time Exceeded (11), Destination Unreachable (3)
  - **TTL (Time To Live)**: Decrements at each router hop, enables traceroute's path discovery
  - Raw sockets allow crafting packets from scratch, bypassing OS network stack

- **Extracted to**: [[networking-fundamentals#OSI Model]]

**Mental model**:
- `ping` = "Are you alive?" (ICMP Echo Request → Reply)
- `traceroute` = "Show me the path" (TTL manipulation to reveal each hop)

### 2. Raw Socket Programming

- **Difficulty**: ⭐⭐⭐⭐ (4/5)
- **Time**: 25h
- **Key insight**:
  - Raw sockets (SOCK_RAW) require root privileges (security risk if misused)
  - Must manually construct ICMP headers: type, code, checksum, identifier, sequence number
  - Checksum calculation = one's complement of one's complement sum (RFC 792)
  - PID as ICMP identifier to distinguish packets from concurrent ping processes

**Code structure learned**:
```c
// Create raw socket
int sockfd = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);

// Craft ICMP header
struct icmp {
    uint8_t  icmp_type;  // 8 for Echo Request
    uint8_t  icmp_code;  // 0
    uint16_t icmp_cksum; // Checksum
    uint16_t icmp_id;    // Process ID
    uint16_t icmp_seq;   // Sequence number
};

// Calculate checksum
uint16_t checksum(void *data, int len);

// Send packet
sendto(sockfd, packet, len, 0, (struct sockaddr *)&dest, sizeof(dest));

// Receive with timeout
select(sockfd + 1, &readfds, NULL, NULL, &timeout);
recvfrom(sockfd, buffer, sizeof(buffer), 0, NULL, NULL);
```

### 3. Network Diagnostics & RTT Measurement

- **Difficulty**: ⭐⭐⭐ (3/5)
- **Time**: 15h
- **Key insight**:
  - RTT (Round Trip Time) = time between sending Echo Request and receiving Echo Reply
  - Microsecond precision with `gettimeofday()` (not milliseconds!)
  - Packet loss calculation: `(transmitted - received) / transmitted * 100`
  - Standard deviation (mdev) shows network stability/jitter
  - `select()` for timeout handling (detect lost packets without blocking forever)

**Statistics learned**:
```bash
# Output format
--- google.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 12.1/13.4/15.2/1.2 ms
```

### 4. TTL Manipulation for Path Discovery (traceroute)

- **Difficulty**: ⭐⭐⭐⭐⭐ (5/5)
- **Time**: 20h
- **Key insight**:
  - TTL starts at 64 (typically) and decrements at each router
  - When TTL reaches 0, router sends ICMP Time Exceeded (Type 11) back to source
  - **traceroute trick**: Send packets with TTL=1, 2, 3, ... to reveal each hop
  - Each router along the path responds, building a map of the network topology

**How traceroute works**:
1. Send packet with TTL=1 → first router responds with Time Exceeded
2. Send packet with TTL=2 → second router responds with Time Exceeded
3. Continue until destination reached (Echo Reply) or max hops (30)

```c
// Set TTL on socket
int ttl = 1;
setsockopt(sockfd, IPPROTO_IP, IP_TTL, &ttl, sizeof(ttl));

// Increment TTL for each hop
for (int ttl = 1; ttl <= max_hops; ttl++) {
    // Send probe packets
    // Wait for ICMP Time Exceeded (Type 11) or Echo Reply (Type 0)
}
```

### 5. DNS Resolution for Network Tools

- **Difficulty**: ⭐⭐ (2/5)
- **Time**: 5h
- **Key insight**:
  - `getaddrinfo()` is modern, IPv4/IPv6 compatible (replaces old `gethostbyname()`)
  - Converts hostname → IP address before crafting packets
  - Must handle both IPv4 addresses and domain names

```c
struct addrinfo hints = {0};
hints.ai_family = AF_INET;      // IPv4
hints.ai_socktype = SOCK_RAW;

struct addrinfo *result;
getaddrinfo("google.com", NULL, &hints, &result);
// result contains IP address
```

---

## ⚠️ Challenges & Solutions

### Challenge 1: Checksum Calculation Always Wrong

**Problem**: ping sent packets but never received replies. Wireshark showed malformed ICMP packets with incorrect checksums.

**Root Cause**: Forgot to zero out checksum field before calculation. RFC 792 requires checksum field = 0 during computation, then filled with computed value.

**Solution**:
```c
// WRONG
icmp->icmp_cksum = checksum(icmp, sizeof(struct icmp));

// CORRECT
icmp->icmp_cksum = 0;  // Zero out first!
icmp->icmp_cksum = checksum(icmp, sizeof(struct icmp));
```

**Time wasted**: 4h debugging with Wireshark
**Lesson**: Always read RFC specifications carefully - small details matter in protocol implementation

---

### Challenge 2: Receiving Packets from Other ping Processes

**Problem**: When multiple users ran ping on the same system, my ft_ping received their replies too, showing incorrect RTT and duplicate responses.

**Root Cause**: ICMP is connectionless - all Echo Replies arrive at all listening sockets. No built-in filtering like TCP ports.

**Solution**: Use Process ID (PID) as ICMP identifier field to distinguish our packets:
```c
icmp->icmp_id = getpid();  // Unique per process

// When receiving, filter by ID
if (received_icmp->icmp_id != getpid()) {
    continue;  // Not our packet, ignore
}
```

**Time wasted**: 3h
**Lesson**: In shared protocols (ICMP, broadcast), always include unique identifiers to filter traffic

---

### Challenge 3: select() Timeout Logic for Lost Packets

**Problem**: When packets were lost, ft_ping blocked forever waiting for replies that would never come.

**Root Cause**: `recvfrom()` blocks indefinitely without timeout mechanism.

**Solution**: Use `select()` with timeout to detect lost packets:
```c
struct timeval timeout = {
    .tv_sec = 1,   // 1 second timeout
    .tv_usec = 0
};

fd_set readfds;
FD_ZERO(&readfds);
FD_SET(sockfd, &readfds);

if (select(sockfd + 1, &readfds, NULL, NULL, &timeout) == 0) {
    // Timeout - packet lost
    printf("Request timeout for icmp_seq %d\n", seq);
}
```

**Time wasted**: 2h
**Lesson**: Never use blocking I/O without timeouts in network programming - packets get lost

---

### Challenge 4: TTL Decrement Understanding (traceroute)

**Problem**: traceroute showed only the final destination, not intermediate hops.

**Root Cause**: Didn't understand that routers send ICMP Time Exceeded when TTL=0, not the destination host.

**Solution**: Listen for **both** ICMP types:
- Type 11 (Time Exceeded) = intermediate router response
- Type 0 (Echo Reply) = destination reached

```c
// Parse received ICMP packet
if (icmp->icmp_type == ICMP_TIME_EXCEEDED) {
    printf("%d  %s  %.2f ms\n", ttl, router_ip, rtt);
}
else if (icmp->icmp_type == ICMP_ECHOREPLY) {
    printf("%d  %s  %.2f ms (destination)\n", ttl, dest_ip, rtt);
    break;  // Reached destination
}
```

**Time wasted**: 6h reading RFC 792 and testing
**Lesson**: Understanding protocol behavior is more important than coding skills

---

## 🔧 Reusable Assets

**Concepts extracted**:
- [[networking-fundamentals#OSI Model]] - Added Layer 3 explanation with ICMP examples

**Knowledge added**:
- OSI Layer 3 deep dive (IP, ICMP, routing)
- Raw socket programming patterns
- Network diagnostics methodology (ping → traceroute → deeper tools)

**Commands discovered**:
```bash
# Capture ICMP traffic to debug
sudo tcpdump -i any icmp -vvv

# Show raw packet hex dump
sudo tcpdump -i any icmp -XX

# Test with system ping to compare behavior
ping -c 5 -v 8.8.8.8

# Check routing table
ip route show
```

**Updated**:
- [[networking-fundamentals]] - Added OSI Model section
- [[MOC-Networking-Fundamentals]] - Created learning path
- [[cheatsheets/linux/networking]] - Could add ICMP troubleshooting commands (TODO)

---

## 📊 Time Breakdown

| Phase | Assisted | Autonomous | Total |
|-------|----------|------------|-------|
| Learning ICMP/RFC | 10h | 5h | 15h |
| ft_ping implementation | 15h | 23h | 38h |
| ft_traceroute implementation | 12h | 15h | 27h |
| Debugging | 5h | 5h | 10h |
| **TOTAL** | **42h (47%)** | **48h (53%)** | **~90h** |

**Ratio**: 47% assisted ⚠️ (target <30% not met - complex low-level C project)

**Skills gained**:
- Network programming: 45h
- Protocol implementation: 30h
- System-level C: 25h
- Debugging with Wireshark/tcpdump: 10h

---

## 🎯 Outcomes

### Success Metrics
- ✅ **ft_ping**: 100% compatible with system ping output format
- ✅ **ft_traceroute**: Correctly reveals network paths with RTT per hop
- ✅ **Learning**: Deep understanding of Layer 3 networking (can explain OSI model now!)
- ✅ **Skills**: Raw socket programming, ICMP protocol, network diagnostics

### What Worked Well
1. **Reading RFCs first**: RFC 792 (ICMP) gave complete protocol specification - saved time vs trial-and-error
2. **Using Wireshark**: Packet capture tool was invaluable for debugging malformed packets
3. **Incremental approach**: Built ping first (simpler) before tackling traceroute (complex)

### What Could Be Improved
1. **IPv6 support**: Only implemented IPv4 - modern networks need IPv6
2. **Error handling**: Some edge cases not fully handled (fragmented packets, unreachable hosts)
3. **Testing**: Should have written automated tests comparing with system ping/traceroute output

### DevOps Relevance
These projects taught me the **fundamentals** that apply to DevOps/Cloud:
- **Layer 3 understanding**: I deeply understand how IP routing works (critical for VPC, K8s pod networking)
- **ICMP protocol**: Know when to use ping/traceroute for troubleshooting
- **OSI model**: Can identify which layer is failing (L3 routing? L4 port? L7 app?)
- **Network diagnostics**: Systematic approach to debugging connectivity issues

**Future application**:
- Docker overlay networks (will understand VXLAN because I know IP encapsulation)
- AWS VPC configuration (will understand routing tables because I implemented routing - when AWS concepts available)
- Kubernetes pod networking (will understand CNI plugins because I know Layer 3 - when K8s concepts available)
- Load balancers L4 vs L7 (will understand the difference because I know OSI layers)

**Troubleshooting approach learned**:
1. Layer 3: Can I reach the host? (`ping`)
2. Layer 4: Is the port open? (`nc -zv`)
3. Layer 7: Is the application responding? (`curl`)

This systematic layer-by-layer approach will be essential when working with cloud/container networking.

---

## 📝 Knowledge Base Updates

✅ Completed:
- [x] Created [[MOC-Networking-Fundamentals]] (MOC for network learning path)
- [x] Updated [[networking-fundamentals]] with OSI Model section
- [x] Added this learning report to projects/

📝 TODO (optional future work):
- [ ] Create [[troubleshooting/networking/icmp-not-working]] guide
- [ ] Add networking commands to [[cheatsheets/linux/networking]]
- [ ] Write [[concepts/networking/raw-sockets-programming]] concept doc

---

**Completed**: 2024-11 (ft_traceroute finished November 2024)
**Last updated**: 2025-12-29
**Learning extracted**: OSI Model, ICMP protocol, Layer 3 networking fundamentals
**Next use**: Applying this knowledge to Kubernetes networking, cloud VPC configuration
