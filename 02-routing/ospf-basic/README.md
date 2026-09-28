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

<img width="368" height="348" alt="image" src="https://github.com/user-attachments/assets/3358a198-00d2-4045-bc2b-508089a4cfeb" />


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

### 1. Configure OSPF on R1

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

### 2. Configure OSPF on R2

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

### 3. Configure OSPF on R3

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

### 1. Check OSPF neighbours

Run:

R1

<img width="451" height="74" alt="image" src="https://github.com/user-attachments/assets/2cdef195-073b-4482-88e4-1ced734980fb" />

R2

<img width="469" height="74" alt="image" src="https://github.com/user-attachments/assets/305d94c8-ca03-4084-8a07-a0f569de0b11" />

R3

<img width="470" height="71" alt="image" src="https://github.com/user-attachments/assets/ad16d637-04cd-4ad1-952f-8147431b28a5" />


Each router should discover its connected OSPF neighbours.

For example, R1 should have neighbours with R2 and R3.

Understand:

> An OSPF neighbour is another router with which OSPF has established a relationship and exchanges routing information.

---

### 2. Check the routing table

R1

<img width="439" height="278" alt="image" src="https://github.com/user-attachments/assets/69227bcd-ca30-4e95-bd31-33bf8d6283e9" />

R2

<img width="437" height="281" alt="image" src="https://github.com/user-attachments/assets/b74ac08b-e9ff-4f36-81bd-194d7ae8fccf" />

R3

<img width="440" height="281" alt="image" src="https://github.com/user-attachments/assets/e475a537-02da-4910-8785-209407c63fe5" />



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

The result of: trace 192.168.30.10

<img width="463" height="71" alt="image" src="https://github.com/user-attachments/assets/7b69bc1f-30ab-4834-af67-f5acd7b67322" />

---

## OSPF Concepts to Understand

### Dynamic Routing

Static routing requires an administrator to manually configure routes.

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

<img width="653" height="108" alt="image" src="https://github.com/user-attachments/assets/306b9e1f-bfea-4dbb-99d5-073254093a63" />




Check the routing table again:

```text
show ip route
```
The next hop has changed to R2.

<img width="437" height="260" alt="image" src="https://github.com/user-attachments/assets/ef498b18-965a-4099-af20-0851f1ebcb7f" />


Then test:

```text
PC1 → PC3
```
<img width="347" height="91" alt="image" src="https://github.com/user-attachments/assets/45973fc7-c43c-4972-8da5-b43cd0383e6c" />

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
