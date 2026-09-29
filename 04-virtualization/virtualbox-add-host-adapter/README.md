# Adding a Host Adapter to Enable Windows–VM Communication

## Objective

The objective of this note is to document how to add a second virtual network adapter to a VirtualBox virtual machine so that the Windows host can communicate directly with the VM.

This configuration is useful for network labs where the Windows host needs to act as a client, while the VM provides services such as a web server, SSH server, or other network services.

---

## 1. Network Architecture

The VM uses two virtual network adapters.

```text
                         VirtualBox VM
                              │
                 ┌────────────┴────────────┐
                 │                         │
              Adapter 1                 Adapter 2
                 │                         │
                NAT                    Host-Only
                 │                         │
                 ▼                         ▼
             Internet              VirtualBox Host-Only
                                    Virtual Network
                                           │
                                           │
                                           ▼
                                      Windows Host
```

### Adapter 1 — NAT

Used to provide the VM with Internet access.

```text
VM → NAT → Internet
```

For example, this allows the VM to run:

```bash
sudo apt update
```

### Adapter 2 — Host-Only

Used to create direct communication between the Windows host and the VM.

```text
Windows Host
192.168.56.1
      │
      │ Host-Only Network
      │
      ▼
VM
192.168.56.102
```

---

# 2. Why Add a Second Adapter?

A VM configured only with NAT can access external networks, but the network configuration is not designed primarily for direct host-to-VM communication.

For a lab environment, it is useful to separate the two purposes:

```text
Adapter 1
    ↓
Internet connectivity

Adapter 2
    ↓
Windows ↔ VM communication
```

This allows the Windows host to communicate directly with a service running on the VM.

For example:

```text
Windows
   │
   │ HTTP
   ▼
Debian VM
   │
   ▼
Apache Web Server
```

---

# 3. Check the Virtual Machine

Open PowerShell and list the VirtualBox VMs.

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" list vms
```

Identify the VM that needs host communication.

Example:

```text
"Debian for http verification"
```

---

# 4. Check the Existing Network Adapters

Display the VM's network configuration.

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" showvminfo "Debian for http verification"
```

Before the configuration, the VM had:

```text
NIC 1: Attachment: NAT
NIC 2: disabled
```

Therefore:

```text
NIC 1 → NAT
NIC 2 → Disabled
```

---

# 5. Check the Windows Host Adapter

Check the Host-Only adapters available in VirtualBox.

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" list hostonlyifs
```

The available adapter was:

```text
Name: VirtualBox Host-Only Ethernet Adapter
IPAddress: 192.168.56.1
NetworkMask: 255.255.255.0
Status: Up
```

The Windows host therefore uses:

```text
192.168.56.1/24
```

for the Host-Only network.

---

# 6. Add Adapter 2 to the VM

Configure the second virtual NIC as a Host-Only adapter.

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyvm "Debian for http verification" --nic2 hostonly --hostonlyadapter2 "VirtualBox Host-Only Ethernet Adapter"
```

This changes the VM's configuration from:

```text
NIC 1 → NAT
NIC 2 → Disabled
```

to:

```text
NIC 1 → NAT
NIC 2 → Host-Only
```

---

# 7. Verify the Configuration

Run:

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" showvminfo "Debian for http verification"
```

Confirm that the second adapter is connected to the Host-Only network.

Expected:

```text
NIC 1: Attachment: NAT
NIC 2: Attachment: Host-only Interface 'VirtualBox Host-Only Ethernet Adapter'
```

---

# 8. Check the VM's New Network Interface

Start the VM and check the network interfaces from inside Linux.

```bash
ip addr
```

The second VirtualBox adapter appeared as:

```text
enp0s8
```

The interface was assigned:

```text
192.168.56.102
```

The resulting connection is:

```text
Windows
192.168.56.1
      │
      │ Host-Only
      │ 192.168.56.0/24
      │
      ▼
VM
enp0s8
192.168.56.102
```

---

# 9. Test Windows → VM Communication

From Windows, the VM can now be accessed using its Host-Only IP address.

For example:

```text
http://192.168.56.102
```

If Apache is running on the VM:

```text
Windows Browser
      │
      │ HTTP
      ▼
192.168.56.102:80
      │
      ▼
Apache
```

This configuration also allows tools such as Wireshark running on the Windows host to capture the communication.

---

# 10. Final Configuration

```text
                         Internet
                            │
                            │
                           NAT
                            │
                    ┌───────┴───────┐
                    │      VM       │
                    │               │
                    │  Adapter 1    │
                    │      NAT      │
                    │               │
                    │  Adapter 2    │
                    │   enp0s8      │
                    │192.168.56.102  │
                    └───────┬───────┘
                            │
                       Host-Only
                    192.168.56.0/24
                            │
                            ▼
                    Windows Host
                    192.168.56.1
```

# Key Takeaways

* A VirtualBox VM can use multiple virtual network adapters.
* Adapter 1 can be used for NAT and Internet connectivity.
* Adapter 2 can be added for direct Windows-to-VM communication.
* `VBoxManage modifyvm` can configure the additional adapter from PowerShell.
* The Linux VM detects the additional adapter as a network interface such as `enp0s8`.
* When the Windows and VM interfaces are configured in the same subnet, they can communicate directly.
* This setup is useful for web server labs, SSH labs, security testing, packet capture, and other network engineering exercises.
