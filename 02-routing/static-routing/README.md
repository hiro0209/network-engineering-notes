# Static Routing

## Objective

* Understand the basic role of a router in connecting different IP networks.
* Understand how a router uses a routing table to forward packets.
* Configure static routes between two different networks.
* Verify connectivity between devices on different networks.

## Topology

<img width="575" height="185" alt="image" src="https://github.com/user-attachments/assets/4aef80b5-df3e-4170-86df-d198f27305d0" />


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
| R1     | F0/0      | 192.168.10.1  | /24         | -               |
| R1     | S0/0      | 10.0.0.1      | /30         | -               |
| R2     | S0/0      | 10.0.0.2      | /30         | -               |
| R2     | F0/0      | 192.168.20.1  | /24         | -               |
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

R1

<img width="472" height="71" alt="image" src="https://github.com/user-attachments/assets/891bc21c-d2a7-4ea3-9c99-0eeb76c390f6" />


R2

<img width="473" height="75" alt="image" src="https://github.com/user-attachments/assets/75c8e06a-4d01-4269-b66a-0eb6c36dea24" />

### 2. Check the routing table

On R1:

```text
show ip route
```

<img width="443" height="191" alt="image" src="https://github.com/user-attachments/assets/9fbcff7b-cf33-49b0-aa74-c93fd2963101" />

On R2:

<img width="438" height="187" alt="image" src="https://github.com/user-attachments/assets/88d465b0-c85b-44a2-adea-d8646ec017c9" />



### 3. Test connectivity

From PC1:

```text
ping 192.168.20.10
```

Expected:

```text
Success
```

<img width="337" height="92" alt="image" src="https://github.com/user-attachments/assets/1440aa33-1a66-47dc-9a6a-8605323e3546" />


Then test the reverse direction from PC2:

```text
ping 192.168.10.10
```

Expected:

```text
Success
```

<img width="332" height="78" alt="image" src="https://github.com/user-attachments/assets/e3181e42-ab2b-48a0-898c-d0d6dc4971fb" />


### 4. Test the path

From PC1:

```text
tracert 192.168.20.10
```

<img width="456" height="84" alt="image" src="https://github.com/user-attachments/assets/35b7c150-5c8f-4617-9ebb-6e7a7595be5a" />

## What I Learned

* A router connects different IP networks.
* A default gateway is used when a host needs to communicate with a different IP network.
* A router uses its routing table to determine where to forward packets.
* Connected routes are automatically added when router interfaces are configured and active.
* A static route is manually configured by an administrator.
* A route specifies a destination network and where packets should be forwarded.
* Two-way communication requires a valid route in both directions.
* `show ip route` can be used to examine a router's routing table.
