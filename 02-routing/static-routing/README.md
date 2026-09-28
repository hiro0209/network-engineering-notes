# Static Routing

## Objective

* Understand the basic role of a router in connecting different IP networks.
* Understand how a router uses a routing table to forward packets.
* Configure static routes between two different networks.
* Verify connectivity between devices on different networks.

## Topology

```text
PC1 ─── R1 ───── R2 ─── PC2
```

### Devices

* 2 × Routers
* 2 × PCs

### Network Structure

```text
192.168.10.0/24
      |
     PC1
      |
     R1
      |
   10.0.0.0/30
      |
     R2
      |
     PC2
      |
192.168.20.0/24
```

## IP Addressing

| Device | Interface | IP Address    | Subnet Mask | Default Gateway |
| ------ | --------- | ------------- | ----------- | --------------- |
| PC1    | NIC       | 192.168.10.10 | /24         | 192.168.10.1    |
| R1     | G0/0      | 192.168.10.1  | /24         | -               |
| R1     | G0/1      | 10.0.0.1      | /30         | -               |
| R2     | G0/0      | 10.0.0.2      | /30         | -               |
| R2     | G0/1      | 192.168.20.1  | /24         | -               |
| PC2    | NIC       | 192.168.20.10 | /24         | 192.168.20.1    |

## Configuration

### 1. Configure PC1

Configure:

```text
IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

### 2. Configure PC2

Configure:

```text
IP Address: 192.168.20.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
```

### 3. Configure R1

Configure the interfaces:

```text
interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface g0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
```

Then configure a static route to the network behind R2:

```text
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

### 4. Configure R2

Configure the interfaces:

```text
interface g0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown

interface g0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
```

Then configure a static route to the network behind R1:

```text
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

## Verification

### 1. Check interface status

On R1 and R2:

```text
show ip interface brief
```

Verify that the required interfaces are:

```text
up    up
```

### 2. Check the routing table

On R1:

```text
show ip route
```

Look for:

```text
C 192.168.10.0/24
C 10.0.0.0/30
S 192.168.20.0/24
```

On R2:

```text
show ip route
```

Look for:

```text
C 192.168.20.0/24
C 10.0.0.0/30
S 192.168.10.0/24
```

### 3. Test connectivity

From PC1:

```text
ping 192.168.20.10
```

Expected:

```text
Success
```

Then test the reverse direction from PC2:

```text
ping 192.168.10.10
```

Expected:

```text
Success
```

### 4. Test the path

From PC1:

```text
tracert 192.168.20.10
```

Observe that the packet passes through the routers before reaching PC2.

## Result

### Interface Verification

Write what you observed from:

```text
show ip interface brief
```

### Routing Table Verification

Write what you observed from:

```text
show ip route
```

### Connectivity Test

Record the results:

```text
PC1 → PC2:
PC2 → PC1:
```

### Traceroute

Record the path observed with:

```text
tracert 192.168.20.10
```

## What I Learned

* A router connects different IP networks.
* A default gateway is used when a host needs to communicate with a different IP network.
* A router uses its routing table to determine where to forward packets.
* Connected routes are automatically added when router interfaces are configured and active.
* A static route is manually configured by an administrator.
* A route specifies a destination network and where packets should be forwarded.
* Two-way communication requires a valid route in both directions.
* `show ip route` can be used to examine a router's routing table.
