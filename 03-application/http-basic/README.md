# HTTP Basic — Apache Web Server

## Objective

The objective of this lab is to build a real web server and deepen my understanding of HTTP communication by capturing and analyzing the traffic with Wireshark.

In this lab, I will:

* Build a web server using Debian 13 and Apache.
* Generate HTTP traffic between a client and the web server.
* Capture the traffic using Wireshark.
* Analyze the HTTP packets and their underlying TCP/IP communication.
* Develop a deeper understanding of how HTTP communication works in practice.

---

## Environment

| Item            | Details            |
| --------------- | ------------------ |
| Host OS         | Windows            |
| Virtualization  | VirtualBox         |
| Server OS       | Debian 13          |
| Web Server      | Apache HTTP Server |
| HTTP Client     | curl / Web Browser |
| Packet Analysis | Wireshark          |

---

## Topology

```text
Windows Host
     |
     | HTTP
     |
VirtualBox Host-Only Network
     |
     |
Debian 13
Apache Web Server
```

---

## Configuration

### 1. Install Apache

Apache was installed on the Debian server.

```bash
sudo apt update
sudo apt install apache2
```

### 2. Install curl

`curl` is used as an HTTP client to generate HTTP requests.

```bash
sudo apt install curl
```

### 3. Configure the VirtualBox Network

The Debian server was connected to the Windows host using a VirtualBox Host-Only Network.

This allows HTTP traffic to be generated between the Windows host and the Debian web server without exposing the lab server directly to the external network.

---

# Verification

## 1. Verify Apache Service

```bash
sudo systemctl status apache2
```

### Result

<img width="398" height="179" alt="image" src="https://github.com/user-attachments/assets/85633579-c431-4965-8acf-70e0a67d91ce" />

The Apache service was running successfully.

---

## 2. Verify HTTP Response from the Server

```bash
curl http://localhost
```

### Result

<img width="400" height="237" alt="image" src="https://github.com/user-attachments/assets/a6addeb9-8219-4362-9cab-7d89a64934cc" />

The Apache web server returned an HTTP response successfully.

---

## 3. Verify Port 80

```bash
sudo ss -lntp | grep :80
```

### Result

<img width="402" height="53" alt="image" src="https://github.com/user-attachments/assets/80c403fe-033a-4c0c-98a2-69ced4872f9a" />

Apache was listening for HTTP connections on TCP port 80.

---

## 4. Generate HTTP Traffic from the Client

From the Windows host, access the Debian web server using its Host-Only IP address.

```text
http://<Debian-IP-address>
```

Alternatively, HTTP traffic can be generated using:

```bash
curl http://<Debian-IP-address>
```

### Result

[Add screenshot here]

The Windows client successfully communicated with the Debian web server using HTTP.

---

## 5. Capture HTTP Traffic with Wireshark

Wireshark was used to capture the communication between the client and the web server.

Display filter:

```text
http
```

Alternatively:

```text
tcp.port == 80
```

### Result

[Add Wireshark screenshot here]

The HTTP communication between the client and the web server was captured successfully.

---

## 6. Analyze the HTTP Communication

The captured packets were analyzed to identify the sequence of communication.

### TCP Connection

```text
Client → Server : SYN
Server → Client : SYN, ACK
Client → Server : ACK
```

This establishes the TCP connection before HTTP data is transmitted.

### HTTP Request

```text
Client → Server : HTTP GET
```

The client sends an HTTP request to the web server.

### HTTP Response

```text
Server → Client : HTTP Response
```

The Apache server returns the requested web content.

### Communication Structure

```text
HTTP
 ↓
TCP
 ↓
IP
 ↓
Ethernet
```

---

# Results

The Debian 13 web server was successfully built using Apache.

HTTP communication was successfully generated between the client and the web server and captured using Wireshark.

The captured packets allowed the communication process to be observed at the TCP/IP and HTTP levels.

---

# What I Learned

* How to build a real web server using Debian and Apache.
* How an HTTP client generates an HTTP request.
* How Apache receives an HTTP request and returns an HTTP response.
* How TCP establishes a connection before HTTP communication.
* How HTTP communication can be observed and analyzed using Wireshark.
* How the HTTP, TCP, IP, and Ethernet layers work together during real network communication.
