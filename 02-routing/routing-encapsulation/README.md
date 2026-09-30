# Routing Encapsulation and ARP

## 1. Objective

The objective of this lab is to verify how a router forwards an IP packet across different Layer 2 technologies.

The lab focuses on observing how:

* An IP packet is encapsulated into different Layer 2 frames at each hop.
* Ethernet and HDLC are used on different links.
* Routers remove the incoming Layer 2 header and trailer before forwarding the IP packet.
* Routers create a new Layer 2 frame for the outgoing interface.
* ARP is used to resolve the destination host's MAC address on the final LAN.

Packet captures will be analyzed using Wireshark.

## 2. Concepts

* Layer 3 IP routing
* Layer 2 encapsulation
* Ethernet
* HDLC
* Frame Check Sequence (FCS)
* MAC addresses
* IP addresses
* ARP
* Next-hop forwarding
* End-to-end IP packet vs. per-hop Layer 2 frame

## 3. Topology

```text
PC1
 |
 | Ethernet
 |
R1
 |
 | Serial / HDLC
 |
R2
 |
 | Ethernet
 |
R3
 |
 | Ethernet
 |
PC2
```

### Devices

| Device | Role             | IP Address                |
| ------ | ---------------- | ------------------------- |
| PC1    | Source host      | 192.168.1.10              |
| R1     | Router           | 192.168.1.1 / WAN address |
| R2     | Router           | WAN addresses             |
| R3     | Router           | 192.168.4.1               |
| PC2    | Destination host | 192.168.4.10              |

### Links

| Source | Destination | Technology    |
| ------ | ----------- | ------------- |
| PC1    | R1          | Ethernet      |
| R1     | R2          | Serial / HDLC |
| R2     | R3          | Ethernet      |
| R3     | PC2         | Ethernet      |

## 4. Expected Forwarding Process

The packet will travel through the following path:

```text
PC1 → R1 → R2 → R3 → PC2
```

The expected Layer 2 encapsulation is:

```text
PC1 → R1     Ethernet
R1  → R2     HDLC
R2  → R3     Ethernet
R3  → PC2    Ethernet
```

The destination IP address remains the destination host throughout the routing process:

```text
Destination IP = 192.168.4.10
```

However, the Layer 2 frame is rebuilt at each hop.

```text
PC1 → R1
Ethernet Frame
    └── IP Packet → 192.168.4.10

R1 → R2
HDLC Frame
    └── IP Packet → 192.168.4.10

R2 → R3
Ethernet Frame
    └── IP Packet → 192.168.4.10

R3 → PC2
Ethernet Frame
    └── IP Packet → 192.168.4.10
```

## 5. Router Forwarding Process

At each router, the following process is expected:

```text
Incoming Layer 2 Frame
        ↓
Check FCS
        ↓
Remove Layer 2 Header / Trailer
        ↓
Inspect Destination IP
        ↓
Check Routing Table
        ↓
Select Outgoing Interface
        ↓
Create New Layer 2 Frame
        ↓
Forward
```

### R1

R1 receives an Ethernet frame from PC1.

R1 will:

1. Check the FCS.
2. Remove the Ethernet header and trailer.
3. Inspect the destination IP address.
4. Consult the routing table.
5. Select the serial interface toward R2.
6. Encapsulate the IP packet in an HDLC frame.
7. Forward the frame to R2.

### R2

R2 receives the HDLC frame from R1.

R2 will:

1. Check the frame for errors.
2. Remove the HDLC header and trailer.
3. Inspect the destination IP address.
4. Consult the routing table.
5. Select the Ethernet interface toward R3.
6. Encapsulate the IP packet in a new Ethernet frame.
7. Forward the frame to R3.

### R3

R3 receives the Ethernet frame from R2.

R3 will:

1. Check the FCS.
2. Remove the Ethernet header and trailer.
3. Inspect the destination IP address.
4. Determine that the destination network is directly connected.
5. Resolve PC2's MAC address using ARP if it is not already in the ARP cache.
6. Encapsulate the IP packet in a new Ethernet frame.
7. Forward the frame to PC2.

## 6. ARP Verification

The ARP process will be specifically verified at R3.

Before testing, the ARP cache should be cleared or verified to ensure that R3 does not already know PC2's MAC address.

Expected process:

```text
R3 → Broadcast
Who has 192.168.4.10?

PC2 → R3
192.168.4.10 is at <PC2 MAC>
```

R3 can then use the learned MAC address to create the Ethernet frame:

```text
Destination MAC = PC2
Source MAC      = R3
Destination IP  = 192.168.4.10
```

If the MAC address is already present in R3's ARP cache, an ARP request will not be generated.

## 7. Packet Capture

Wireshark will be used to capture and compare traffic at the different links.

### Capture 1 — PC1 → R1

Expected:

```text
Ethernet
 ├── Source MAC      = PC1
 └── Destination MAC = R1

IP
 ├── Source IP      = PC1
 └── Destination IP = PC2
```

### Capture 2 — R1 → R2

Expected:

```text
HDLC
 └── IP Packet
      ├── Source IP      = PC1
      └── Destination IP = PC2
```

The Ethernet header used on the previous link should no longer be present.

### Capture 3 — R2 → R3

Expected:

```text
Ethernet
 ├── Source MAC      = R2
 └── Destination MAC = R3

IP
 ├── Source IP      = PC1
 └── Destination IP = PC2
```

### Capture 4 — R3 → PC2

Expected:

```text
ARP Request
      ↓
ARP Reply
      ↓
Ethernet Frame
      ↓
IP Packet
```

The Ethernet destination MAC should be PC2's MAC address.

## 8. Results

| Hop      | Layer 2  | Destination MAC / Address | Destination IP |
| -------- | -------- | ------------------------- | -------------- |
| PC1 → R1 | Ethernet | R1 MAC                    | PC2 IP         |
| R1 → R2  | HDLC     | N/A                       | PC2 IP         |
| R2 → R3  | Ethernet | R3 MAC                    | PC2 IP         |
| R3 → PC2 | Ethernet | PC2 MAC                   | PC2 IP         |

### ARP

| Device | Action      | Result                   |
| ------ | ----------- | ------------------------ |
| R3     | ARP Request | Requests PC2's MAC       |
| PC2    | ARP Reply   | Provides its MAC address |
| R3     | ARP Cache   | Stores IP-to-MAC mapping |

## 9. Key Findings

1. The Layer 3 destination identifies the final destination of the packet.
2. The Layer 2 destination identifies the device that should receive the frame on the current link.
3. Layer 2 encapsulation changes at each hop.
4. Ethernet is used between PC1 and R1, while HDLC is used between R1 and R2.
5. R2 creates a new Ethernet frame when forwarding the packet to R3.
6. R3 uses ARP to discover PC2's MAC address when the address is not already in its ARP cache.
7. A single IP packet is therefore carried by multiple Layer 2 frames during its journey from PC1 to PC2.

## 10. Conclusion

This lab demonstrates the relationship between Layer 3 routing and Layer 2 forwarding.

The network layer determines where the packet needs to go, while the data-link layer provides the frame required to move the packet across the current link.

By capturing the traffic with Wireshark, the encapsulation process can be observed directly:

```text
Ethernet
   ↓
HDLC
   ↓
Ethernet
   ↓
ARP + Ethernet
```

This confirms that the Layer 3 packet is forwarded end-to-end while Layer 2 frames are created and replaced at each hop.
