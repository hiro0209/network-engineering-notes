# DNS Basic Lab

## Objective

The objective of this lab is to verify how the Domain Name System (DNS) resolves a hostname into an IP address.

This lab uses a client and a DNS server to generate DNS queries and responses, which are captured and analyzed using Wireshark.

The lab focuses on understanding:

* How a hostname is resolved into an IP address
* How a DNS query and response work
* How DNS uses UDP port 53
* How the client uses the resolved IP address for subsequent communication

---

## Topology

```text
+--------+                  +------------+
| Client | ---------------- | DNS Server |
+--------+                  +------------+
192.168.1.10               192.168.1.2
```

### Devices

| Device     | IP Address   | Role                                 |
| ---------- | ------------ | ------------------------------------ |
| Client     | 192.168.1.10 | Generates DNS queries                |
| DNS Server | 192.168.1.2  | Resolves hostnames into IP addresses |

---

## Configuration

### Client

```text
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
DNS Server:      192.168.1.2
```

### DNS Server

```text
IP Address:      192.168.1.2
Subnet Mask:     255.255.255.0
DNS Service:     Enabled
```

A DNS record is configured on the DNS server:

```text
server1.lab.local → 192.168.1.100
```

---

## Test Procedure

### 1. Generate DNS Query

The client is configured to use `192.168.1.2` as its DNS server.

A DNS lookup is performed for:

```text
server1.lab.local
```

The client sends a DNS query to the DNS server requesting the IP address associated with this hostname.

### 2. Capture Traffic

Wireshark is started on the client's network interface before generating the DNS query.

The DNS traffic is captured and saved as a `.pcapng` file for analysis.

### 3. Analyze DNS Query

The DNS request is inspected to verify:

* Source IP address
* Destination IP address
* UDP port
* DNS query name
* DNS query type

The expected query is:

```text
server1.lab.local
```

### 4. Analyze DNS Response

The DNS response is inspected to verify that the DNS server returns the corresponding IP address:

```text
192.168.1.100
```

---

## Wireshark Analysis

### DNS Query

```text
Source IP:       192.168.1.10
Destination IP:  192.168.1.2
Protocol:        UDP
Source Port:     Ephemeral port
Destination Port: 53
Query Name:      server1.lab.local
Query Type:      A
```

The `A` record query requests an IPv4 address.

### DNS Response

```text
Source IP:        192.168.1.2
Destination IP:   192.168.1.10
Protocol:         UDP
Source Port:      53
Destination Port: Ephemeral port
Query Name:       server1.lab.local
Answer:           192.168.1.100
```

The DNS server returns the IPv4 address associated with the requested hostname.

---

## Expected Result

The client should send a DNS query to the configured DNS server.

```text
Client
  |
  | DNS Query
  | "What is the IP address of server1.lab.local?"
  ↓
DNS Server
  |
  | DNS Response
  | "192.168.1.100"
  ↓
Client
```

The client should receive the IP address:

```text
server1.lab.local
        ↓
192.168.1.100
```

The DNS query and response should be visible in Wireshark.

---

## Results

| Item                | Observed Result     | Expected Result     | Status |
| ------------------- | ------------------- | ------------------- | ------ |
| DNS Query           | `server1.lab.local` | `server1.lab.local` | PASS   |
| Query Name          | `server1.lab.local` | `server1.lab.local` | PASS   |
| DNS Server          | `192.168.1.2`       | `192.168.1.2`       | PASS   |
| DNS Response        | Received            | Received            | PASS   |
| Resolved IP Address | `192.168.1.100`     | `192.168.1.100`     | PASS   |
| Transport Protocol  | UDP                 | UDP                 | PASS   |
| Destination Port    | 53                  | 53                  | PASS   |

---

## Key Findings

1. DNS allows a hostname to be resolved into an IP address.

2. The client sends a DNS query to the configured DNS server rather than placing the hostname directly into the destination IP field of an IP packet.

3. The DNS server returns the IPv4 address associated with the requested hostname.

4. DNS commonly uses UDP port 53 for standard DNS queries and responses.

5. Once the IP address has been resolved, the client can use the IP address for subsequent communication with the destination host.

---

## Communication Flow

The complete process can be represented as:

```text
Hostname
server1.lab.local
        |
        | DNS Query
        ↓
DNS Server
192.168.1.2
        |
        | DNS Response
        ↓
IP Address
192.168.1.100
        |
        | Subsequent IP communication
        ↓
Destination Server
```

This demonstrates the relationship between DNS name resolution and IP-based communication.

---

## Conclusion

This lab successfully verified the basic DNS name resolution process.

The client sent a DNS query for `server1.lab.local` to the DNS server. The DNS server responded with the corresponding IPv4 address, `192.168.1.100`.

Wireshark was used to observe the DNS query and response, including the source and destination IP addresses, UDP port 53, query name, and returned IP address.

The experiment demonstrates that DNS provides the translation between human-readable hostnames and IP addresses, allowing applications to use names while network communication ultimately uses IP addresses.

---

## Files

```text
dns-basic/
├── README.md
└── captures/
    └── dns-query-response.pcapng
```
