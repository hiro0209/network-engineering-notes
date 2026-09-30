# ARP Basic Lab

## Objective

The objective of this lab is to verify how the Address Resolution Protocol (ARP) dynamically resolves an IPv4 address into a MAC address on an Ethernet LAN.

This lab uses two hosts connected through an Ethernet switch and Wireshark to capture and analyze the ARP request and reply process.

The lab focuses on understanding:

* How an ARP Request discovers a MAC address
* Why an ARP Request is sent as an Ethernet broadcast
* How an ARP Reply provides the requested MAC address
* How the learned IP-to-MAC mapping is stored in the ARP cache
* Why ARP is not required for every packet transmission

---

## Topology

```text
+----------+                  +----------+
|   PC1    |                  |   PC2    |
|          |                  |          |
|192.168.1.10|               |192.168.1.20|
+-----+----+                  +----+-----+
      |                            |
      | Ethernet                   | Ethernet
      |                            |
      +---------+--------+---------+
                |
            +---+---+
            | Switch|
            +-------+
```

### Devices

| Device | IP Address      | Role                     |
| ------ | --------------- | ------------------------ |
| PC1    | 192.168.1.10/24 | ARP requester            |
| PC2    | 192.168.1.20/24 | ARP responder            |
| Switch | Layer 2 device  | Forwards Ethernet frames |

---

## Configuration

### PC1

```text
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
```

### PC2

```text
IP Address:      192.168.1.20
Subnet Mask:     255.255.255.0
```

Both hosts are configured in the same IPv4 subnet so that PC1 can communicate directly with PC2 over the Ethernet LAN.

---

## Test Procedure

### 1. Clear the ARP Cache

The ARP cache on PC1 is cleared before starting the test.

This ensures that PC1 does not already know the MAC address associated with PC2's IP address.

The ARP cache can be checked using:

```bash
arp -a
```

If necessary, the existing ARP entry is removed before the test.

### 2. Start Wireshark

Wireshark is started on PC1's Ethernet interface.

The capture begins before generating traffic so that the complete ARP exchange can be observed.

A display filter can be used to show only ARP packets:

```text
arp
```

### 3. Generate Traffic

From PC1, a ping is sent to PC2:

```bash
ping 192.168.1.20
```

Because PC1 does not yet know PC2's MAC address, PC1 first performs ARP resolution.

### 4. Analyze the ARP Request

The first ARP packet is inspected in Wireshark.

PC1 sends an ARP Request asking:

```text
Who has 192.168.1.20?
```

The Ethernet destination MAC address is the broadcast address:

```text
ff:ff:ff:ff:ff:ff
```

This allows all devices on the local Ethernet LAN to receive the request.

### 5. Analyze the ARP Reply

PC2 receives the ARP Request and recognizes that the target IP address belongs to itself.

PC2 sends an ARP Reply containing its MAC address.

The reply allows PC1 to create an IP-to-MAC mapping for PC2.

### 6. Check the ARP Cache

After the ARP exchange, the ARP cache on PC1 is checked again:

```bash
arp -a
```

An entry similar to the following should appear:

```text
192.168.1.20    <PC2-MAC-address>
```

This confirms that PC1 has learned and stored PC2's MAC address.

### 7. Verify Subsequent Communication

A second ping is sent to PC2 while the ARP cache entry is still valid.

Wireshark is checked to determine whether another ARP Request is generated.

The expected result is that PC1 uses the existing ARP cache entry instead of sending another ARP Request.

---

## Wireshark Analysis

### ARP Request

```text
Ethernet Destination MAC: ff:ff:ff:ff:ff:ff
Sender IP:                192.168.1.10
Sender MAC:               <PC1-MAC>
Target IP:                192.168.1.20
Target MAC:               00:00:00:00:00:00
```

The destination MAC is the Ethernet broadcast address because PC1 does not yet know which device owns `192.168.1.20`.

The ARP Request effectively asks:

```text
"Who has 192.168.1.20?"
```

### ARP Reply

```text
Ethernet Destination MAC: <PC1-MAC>
Sender IP:                192.168.1.20
Sender MAC:               <PC2-MAC>
Target IP:                192.168.1.10
Target MAC:               <PC1-MAC>
```

The ARP Reply is sent by PC2 and identifies the MAC address associated with `192.168.1.20`.

---

## Expected Result

The expected communication sequence is:

```text
PC1
 |
 | ARP Request
 | "Who has 192.168.1.20?"
 | Destination MAC = FF:FF:FF:FF:FF:FF
 ↓
Switch
 |
 | Broadcast
 +----------+----------+
            |
            ↓
           PC2
            |
            | ARP Reply
            | "192.168.1.20 is at <PC2-MAC>"
            ↓
           PC1
            |
            | Store mapping in ARP cache
            ↓
      IP → MAC mapping
      192.168.1.20
            ↓
       <PC2-MAC>
```

After the ARP exchange, PC1 can encapsulate the IP packet into an Ethernet frame using PC2's MAC address as the destination MAC.

---

## Results

| Item                | Observed Result            | Expected Result   | Status |
| ------------------- | -------------------------- | ----------------- | ------ |
| ARP Request         | Generated                  | Generated         | PASS   |
| ARP Broadcast       | `FF:FF:FF:FF:FF:FF`        | Broadcast         | PASS   |
| Target IP           | `192.168.1.20`             | `192.168.1.20`    | PASS   |
| ARP Reply           | Received from PC2          | Received from PC2 | PASS   |
| Learned MAC Address | `<PC2-MAC>`                | PC2's MAC         | PASS   |
| ARP Cache Entry     | `192.168.1.20 → <PC2-MAC>` | Mapping stored    | PASS   |
| Second Ping         | No new ARP required        | Use ARP cache     | PASS   |

---

## Key Findings

1. ARP allows a host to dynamically learn the MAC address associated with an IPv4 address on the same Ethernet LAN.

2. The ARP Request is sent as an Ethernet broadcast because the sender does not initially know which device owns the target IP address.

3. The ARP Reply identifies the target host's IP address and corresponding MAC address.

4. The learned IP-to-MAC mapping is stored in the ARP cache.

5. Once the mapping exists in the ARP cache, subsequent packets can be sent without performing ARP again until the cache entry expires or is removed.

6. ARP operates within the local Ethernet LAN. It is used to discover the MAC address of the next directly reachable device, not an arbitrary remote host across routed networks.

---

## Communication Flow

The complete process can be represented as:

```text
Destination IP
192.168.1.20
      |
      | ARP
      ↓
Destination MAC
<PC2-MAC>
      |
      ↓
Ethernet Frame
      |
      ↓
PC2
```

This demonstrates the relationship between IP addressing and Ethernet MAC addressing.

---

## Conclusion

This lab successfully verified the basic ARP resolution process.

PC1 initially did not know the MAC address associated with PC2's IPv4 address. PC1 therefore sent an ARP Request as an Ethernet broadcast. PC2 responded with an ARP Reply containing its MAC address.

PC1 then stored the IP-to-MAC mapping in its ARP cache. Subsequent communication could use the cached mapping without generating another ARP Request.

The experiment demonstrates how ARP provides the information required to encapsulate an IPv4 packet inside an Ethernet frame.

---
