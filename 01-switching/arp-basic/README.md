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

<img width="266" height="223" alt="image" src="https://github.com/user-attachments/assets/d4edb36c-a5f4-4f5b-996f-b08c94f78e33" />


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
show arp
```
<img width="226" height="130" alt="image" src="https://github.com/user-attachments/assets/98951203-2719-4dc6-817a-082d3095d033" />


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
<img width="349" height="125" alt="image" src="https://github.com/user-attachments/assets/dd25beb5-1abb-4391-ba47-d60c79a2d69d" />



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

<img width="607" height="157" alt="image" src="https://github.com/user-attachments/assets/c1b756dd-8c42-4560-8f01-38aeac24198d" />


### 5. Analyze the ARP Reply

PC2 receives the ARP Request and recognizes that the target IP address belongs to itself.

PC2 sends an ARP Reply containing its MAC address.

The reply allows PC1 to create an IP-to-MAC mapping for PC2.

<img width="541" height="217" alt="image" src="https://github.com/user-attachments/assets/65b87b99-c58d-4121-b403-c63cbf9dd4eb" />


### 6. Check the ARP Cache

After the ARP exchange, the ARP cache on PC1 is checked again:

```bash
show arp
```


An entry similar to the following should appear:

```text
192.168.1.20    <PC2-MAC-address>
```
<img width="646" height="132" alt="image" src="https://github.com/user-attachments/assets/5e430d20-2842-4fb1-b75a-93e43f2fbb3f" />



This confirms that PC1 has learned and stored PC2's MAC address.

### 7. Verify Subsequent Communication

A second ping is sent to PC2 while the ARP cache entry is still valid.

Wireshark is checked to determine whether another ARP Request is generated.

The expected result is that PC1 uses the existing ARP cache entry instead of sending another ARP Request.

<img width="373" height="286" alt="image" src="https://github.com/user-attachments/assets/87b41d16-0eb4-498c-bed3-f899dc526c19" />
<img width="833" height="311" alt="image" src="https://github.com/user-attachments/assets/98a7df84-7615-4cb5-843d-e8b338e4b043" />


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

After the ARP exchange, PC1 can encapsulate the IP packet into an Ethernet frame using PC2's MAC address as the destination MAC.

---

## Key Findings

1. ARP allows a host to dynamically learn the MAC address associated with an IPv4 address on the same Ethernet LAN.

2. The ARP Request is sent as an Ethernet broadcast because the sender does not initially know which device owns the target IP address.

3. The ARP Reply identifies the target host's IP address and corresponding MAC address.

4. The learned IP-to-MAC mapping is stored in the ARP cache.

5. Once the mapping exists in the ARP cache, subsequent packets can be sent without performing ARP again until the cache entry expires or is removed.

6. ARP operates within the local Ethernet LAN. It is used to discover the MAC address of the next directly reachable device, not an arbitrary remote host across routed networks.

---

## Conclusion

This lab successfully verified the basic ARP resolution process.

PC1 initially did not know the MAC address associated with PC2's IPv4 address. PC1 therefore sent an ARP Request as an Ethernet broadcast. PC2 responded with an ARP Reply containing its MAC address.

PC1 then stored the IP-to-MAC mapping in its ARP cache. Subsequent communication could use the cached mapping without generating another ARP Request.

The experiment demonstrates how ARP provides the information required to encapsulate an IPv4 packet inside an Ethernet frame.

---
