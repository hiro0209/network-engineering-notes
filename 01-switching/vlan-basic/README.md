# VLAN Basic

## Objective

* Create VLANs on a Layer 2 switch.
* Assign switch ports to the appropriate VLANs.
* Understand the role of an access port.
* Verify communication between hosts in the same VLAN.
* Verify that hosts in different VLANs cannot communicate without Layer 3 routing.

## Topology

<img width="226" height="172" alt="image" src="https://github.com/user-attachments/assets/1dbcaca1-e0ab-4a24-a0e5-1b431ca3cdd8" />


### IP Addressing

| Device | IP Address       | VLAN    |
| ------ | ---------------- | ------- |
| PC1    | 192.168.10.10/24 | VLAN 10 |
| PC2    | 192.168.10.20/24 | VLAN 10 |
| PC3    | 192.168.20.10/24 | VLAN 20 |

## Configuration

### VLAN Configuration

```text
vlan 10
 name USERS

vlan 20
 name SERVERS
```

### Access Port Configuration

```text
interface fa0/1
 switchport mode access
 switchport access vlan 10

interface fa0/2
 switchport mode access
 switchport access vlan 10

interface fa0/3
 switchport mode access
 switchport access vlan 20
```

## Verification

Verify VLANs and port membership:

<img width="406" height="154" alt="image" src="https://github.com/user-attachments/assets/41524b98-efc0-4c77-b454-74974f39b1c1" />



Verify the switchport configuration:

Vlan10: Fa0/1, Fa0/2

<img width="230" height="85" alt="image" src="https://github.com/user-attachments/assets/066ee7b3-9023-4679-82d7-69902a9a54e5" />
<img width="230" height="77" alt="image" src="https://github.com/user-attachments/assets/f9970fe5-e1b6-489a-94c1-642dd25ac36a" />


Vlan20: Fa0/3

<img width="229" height="79" alt="image" src="https://github.com/user-attachments/assets/4f48f2d9-e1af-4e75-9a97-a0e3dcfcebd9" />



Verify MAC address learning:

<img width="229" height="101" alt="image" src="https://github.com/user-attachments/assets/0d1425f7-0b84-4922-9ddb-f201056fc6e8" />


Test connectivity:

PC1 → PC2


<img width="289" height="129" alt="image" src="https://github.com/user-attachments/assets/dca4c063-6cd8-461b-9307-2db5b3a90f7f" />

PC1 → PC3


<img width="298" height="109" alt="image" src="https://github.com/user-attachments/assets/9c6f4cff-6ecf-46ab-973f-2a3b7690683a" />



Expected results:

* PC1 → PC2: **Successful**
* PC1 → PC3: **Unsuccessful**

## Result

The switch successfully separated the network into VLAN 10 and VLAN 20.

PC1 and PC2 were able to communicate because they belonged to the same VLAN.

PC1 and PC3 could not communicate because they belonged to different VLANs and no Layer 3 routing was configured.

## What I Learned

* A VLAN creates a separate Layer 2 broadcast domain.
* An access port is assigned to a single VLAN and is typically used to connect end devices.
* Multiple access ports can belong to the same VLAN.
* Devices in the same VLAN can communicate at Layer 2 when they are correctly configured.
* Devices in different VLANs are isolated at Layer 2 and require Layer 3 routing to communicate.
