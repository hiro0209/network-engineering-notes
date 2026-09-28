# OSPF Basic

## Objective

* Understand the basic purpose of dynamic routing.
* Understand how OSPF allows routers to exchange routing information.
* Configure OSPF on multiple routers.
* Understand OSPF neighbour relationships.
* Verify dynamically learned routes in the routing table.
* Compare dynamic routing with static routing.
* Observe how OSPF can select an alternative path when a link fails.

---

## Topology

Use three Cisco 3745 routers.

```text
             R1
            /  \
           /    \
          /      \
        R2 ────── R3
```

The routers will form a triangle so that there are multiple possible paths between networks.

### Devices

* 3 × Cisco 3745 routers
* 3 × VPCS PCs

---

## Network Structure

Each router will have one local LAN and two router-to-router links.

```text
PC1
 |
192.168.10.0/24
 |
R1
 | \
 |  \
 |   \
R2---R3
|     |
|     |
PC2   PC3
```

### Router-to-Router Networks

```text
R1 ── 10.0.12.0/30 ── R2

R2 ── 10.0.23.0/30 ── R3

R1 ── 10.0.13.0/30 ── R3
```

---

## IP Addressing

| Device | Interface     | IP Address    | Subnet Mask |
| ------ | ------------- | ------------- | ----------- |
| PC1    | NIC           | 192.168.10.10 | /24         |
| R1     | LAN interface | 192.168.10.1  | /24         |
| R1     | R2 link       | 10.0.12.1     | /30         |
| R1     | R3 link       | 10.0.13.1     | /30         |
| R2     | R1 link       | 10.0.12.2     | /30         |
| R2     | LAN interface | 192.168.20.1  | /24         |
| R2     | R3 link       | 10.0.23.1     | /30         |
| R3     | R1 link       | 10.0.13.2     | /30         |
| R3     | R2 link       | 10.0.23.2     | /30         |
| R3     | LAN interface | 192.168.30.1  | /24         |
| PC2    | NIC           | 192.168.20.10 | /24         |
| PC3    | NIC           | 192.168.30.10 | /24         |

### Default Gateways

```text
PC1 → 192.168.10.1
PC2 → 192.168.20.1
PC3 → 192.168.30.1
```

---

## Configuration

### 1. Configure the router interfaces

Configure the IP addresses and enable the interfaces with:

```text
interface <interface>
 ip address <IP> <MASK>
 no shutdown
```

Verify the interfaces with:

```text
show ip interface brief
```

All required interfaces should eventually show:

```text
Status     Protocol
up         up
```

---

### 2. Configure OSPF on R1

Start OSPF:

```text
router ospf 1
```

Advertise the connected networks:

```text
network 192.168.10.0 0.0.0.255 area 0
network 10.0.12.0 0.0.0.3 area 0
network 10.0.13.0 0.0.0.3 area 0
```

---

### 3. Configure OSPF on R2

```text
router ospf 1
```

Advertise:

```text
network 192.168.20.0 0.0.0.255 area 0
network 10.0.12.0 0.0.0.3 area 0
network 10.0.23.0 0.0.0.3 area 0
```

---

### 4. Configure OSPF on R3

```text
router ospf 1
```

Advertise:

```text
network 192.168.30.0 0.0.0.255 area 0
network 10.0.13.0 0.0.0.3 area 0
network 10.0.23.0 0.0.0.3 area 0
```

---

## Verification

### 1. Check interface status

On each router:

```text
show ip interface brief
```

Confirm the required interfaces are:

```text
up    up
```

---

### 2. Check OSPF neighbours

Run:

```text
show ip ospf neighbor
```

Each router should discover its connected OSPF neighbours.

For example, R1 should have neighbours with R2 and R3.

Understand:

> An OSPF neighbour is another router with which OSPF has established a relationship and exchanges routing information.

---

### 3. Check the routing table

Run:

```text
show ip route
```

Look for routes marked with:

```text
O
```

`O` means the route was learned through OSPF.

For example, R1 should learn:

```text
O 192.168.20.0/24
O 192.168.30.0/24
```

---

### 4. Test connectivity

From PC1:

```text
ping 192.168.20.10
ping 192.168.30.10
```

Both should succeed.

From PC2:

```text
ping 192.168.10.10
ping 192.168.30.10
```

Both should succeed.

---

### 5. Observe the path

Use:

```text
traceroute 192.168.30.10
```

or the appropriate traceroute command available in the VPCS environment.

Observe which routers the packet passes through.

---

## OSPF Concepts to Understand

### Dynamic Routing

Static routing requires an administrator to manually configure routes.

```text
ip route <network> <mask> <next-hop>
```

Dynamic routing protocols allow routers to exchange routing information automatically.

Examples include:

* OSPF
* EIGRP
* BGP
* RIP

This lab focuses on OSPF.

---

### OSPF

OSPF stands for:

**Open Shortest Path First**

It is a dynamic routing protocol that allows routers to exchange information about available networks and calculate paths through the network.

---

### OSPF Area

This lab uses:

```text
area 0
```

Area 0 is the backbone area of OSPF.

For this basic lab, all routers will be placed in area 0.

---

### OSPF Neighbour

Routers running OSPF on the same network can establish neighbour relationships.

```text
R1 ←→ R2
```

Once neighbours are established, the routers can exchange routing information.

Verify this with:

```text
show ip ospf neighbor
```

---

### OSPF Route

Routes learned through OSPF appear in the routing table with:

```text
O
```

For example:

```text
O 192.168.20.0/24
```

This means the router learned that network through OSPF.

---

### Administrative Distance

Observe the routing table format:

```text
O 192.168.20.0/24 [110/....]
```

The first value represents the administrative distance.

For OSPF:

```text
Administrative Distance = 110
```

Do not focus heavily on this yet. The main goal of this lab is understanding how OSPF learns routes.

---

## Failover Test

This is an important part of this lab.

Because the topology is:

```text
       R1
      /  \
     R2──R3
```

there are multiple paths between routers.

First verify:

```text
PC1 → PC3
```

works.

Then shut down one of the links between R1 and R3.

For example, on R1:

```text
interface <R1-R3 interface>
 shutdown
```

Check the routing table again:

```text
show ip route
```

Then test:

```text
PC1 → PC3
```

again.

Observe whether OSPF selects an alternative path:

```text
PC1
 ↓
R1
 ↓
R2
 ↓
R3
 ↓
PC3
```

This demonstrates one of the major advantages of dynamic routing: routers can adapt when the topology changes.

---

## Result

### Interface Verification

Record the output/results from:

```text
show ip interface brief
```

---

### OSPF Neighbour Verification

Record the OSPF neighbours discovered with:

```text
show ip ospf neighbor
```

---

### Routing Table Verification

Record the OSPF-learned routes from:

```text
show ip route
```

Identify routes marked with:

```text
O
```

---

### Connectivity Test

Record:

```text
PC1 → PC2:
PC1 → PC3:
PC2 → PC1:
PC2 → PC3:
```

---

### Failover Test

Record what happened after shutting down one router-to-router link.

```text
Before failure:

Path:

After failure:

Path:
```

---

## What I Learned

* A router can learn routes dynamically instead of requiring every route to be manually configured.
* OSPF is a dynamic routing protocol.
* OSPF routers establish neighbour relationships to exchange routing information.
* OSPF uses areas to organise routing information.
* Area 0 is the OSPF backbone area.
* OSPF-learned routes are identified by `O` in the routing table.
* OSPF has an administrative distance of 110.
* Dynamic routing can adapt to changes in network topology.
* Multiple paths can provide redundancy when a link fails.
* `show ip ospf neighbor` can be used to verify OSPF neighbour relationships.
* `show ip route` can be used to verify routes learned through OSPF.
