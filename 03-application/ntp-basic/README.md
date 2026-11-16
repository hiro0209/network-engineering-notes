# Network Time Protocol (NTP) Fundamentals

## 1. Overview

Network Time Protocol (NTP) is used to synchronize clocks across networked devices. It allows computers, servers, network devices, and infrastructure services to maintain a consistent time reference even though their local hardware clocks naturally drift.

NTP is not simply a mechanism for copying the displayed time from one computer to another. It measures the timing relationship between systems, estimates clock offset and network delay, evaluates time sources, and gradually adjusts the local clock.

This document explains the protocol concepts that are useful when configuring and troubleshooting time synchronization in network and Windows environments.

## 2. Why Accurate Time Matters

Consistent time is an operational requirement, not just a display preference.

| Area | Why time accuracy matters |
|---|---|
| Authentication | Kerberos uses timestamps as part of its replay-protection mechanism. Excessive clock differences can cause authentication failures. |
| Security monitoring | Logs from endpoints, servers, firewalls, and identity systems need consistent timestamps to reconstruct an incident. |
| Distributed systems | Cluster members, databases, applications, and services use timestamps to coordinate and diagnose activity. |
| Certificates and cryptography | Certificate validity periods and some security protocols depend on correct system time. |
| Network operations | Event correlation across routers, switches, firewalls, and servers is much easier when clocks agree. |

For Windows domain authentication, the default Kerberos policy commonly allows a maximum clock difference of five minutes. This is a tolerance boundary, not a recommended synchronization target. The value can be changed through policy. See Microsoft's [Maximum tolerance for computer clock synchronization](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/maximum-tolerance-for-computer-clock-synchronization).

## 3. NTP Transport and Port

NTP normally uses:

| Property | Value |
|---|---|
| Transport protocol | UDP |
| Destination port | 123 |
| Service | Network Time Protocol |

NTP uses UDP because time synchronization needs timely measurements, not the reliable, ordered byte stream provided by TCP. Retransmitting an old time request can make its delay measurement less useful. The protocol estimates timing from request and response timestamps instead.

**Troubleshooting implication:** a command that tests TCP port 123 does not test NTP. For example, `Test-NetConnection server -Port 123` tests TCP by default. A failed TCP test does not prove that UDP-based NTP is unavailable.

Microsoft documents UDP 123 as the port used by the Windows Time service: [Windows Time Service Tools and Settings](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings?tabs=config).

## 4. How an NTP Exchange Works

In the basic client/server exchange, four timestamps are used:

| Timestamp | Meaning |
|---|---|
| T1 | Client sends the request |
| T2 | Server receives the request |
| T3 | Server sends the response |
| T4 | Client receives the response |

The client uses these timestamps to estimate two important values.

### 4.1 Clock Offset

The estimated offset (θ) is:

\[
\theta = \frac{(T2 - T1) + (T3 - T4)}{2}
\]

The offset estimates how far the client clock is ahead of or behind the server clock. The sign convention in a command's displayed output depends on the tool, so interpret the output according to that tool's documentation.

### 4.2 Round-Trip Delay

The estimated network round-trip delay (δ) is:

\[
\delta = (T4 - T1) - (T3 - T2)
\]

The client subtracts the time the server spent processing the request from the total elapsed time measured by the client.

### 4.3 Why Network Delay Matters

The calculation assumes that the forward and return network paths have sufficiently similar delays. If one direction is consistently slower, the offset estimate can be biased. Queueing, congestion, routing changes, virtualization overhead, and asymmetric paths can all affect the measurement.

NTP uses repeated measurements and filtering rather than trusting one response. The protocol's algorithms are specified in [RFC 5905 – Network Time Protocol Version 4](https://www.rfc-editor.org/rfc/rfc5905).

## 5. Clock Offset, Drift, and Adjustment

These terms describe different problems:

| Term | Meaning |
|---|---|
| Clock offset | The current difference between two clocks. |
| Clock drift | The rate at which a local clock gains or loses time relative to a reference clock. |
| Frequency error | The difference between the local clock's ticking rate and the ideal rate, often expressed in parts per million (ppm). |
| Jitter | Variation in measured timing values over repeated samples. |

A computer's clock is driven by a hardware oscillator. No physical oscillator is perfectly stable: temperature, hardware characteristics, power state, and elapsed time can affect its rate. Even when two systems are initially synchronized, their clocks can gradually diverge.

A time service therefore needs to manage both the clock's phase (the current offset) and its frequency (how quickly it gains or loses time). NTP implementations filter measurements and discipline the local clock to keep it aligned with the selected source.

## 6. Stratum and the Time Hierarchy

Stratum describes a system's position in the synchronization hierarchy. It is not a direct measurement of accuracy.

| Level | General meaning |
|---|---|
| Stratum 0 | Reference-clock hardware such as a GNSS receiver or atomic clock. It is not normally an ordinary network NTP server. |
| Stratum 1 | An NTP server synchronized directly to a reference clock. |
| Stratum 2 | A server synchronized to a Stratum 1 server. |
| Stratum 3 | A server synchronized to a Stratum 2 server. |

Each downstream server is normally one stratum higher than its upstream source. A lower stratum indicates a shorter hierarchy distance, but it does not guarantee that a server is more accurate in every situation. Network delay, clock quality, stability, and source health also matter.

**Windows-specific note:** `w32tm /query /status` may display `Stratum 0` with an unspecified reference when the Windows Time service does not yet have a valid synchronized source. In that context, Stratum 0 should not be interpreted as proof that the machine is connected to a physical Stratum 0 reference clock.

Microsoft's explanation is available in [How the Windows Time Service Works](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/how-the-windows-time-service-works).

## 7. Root Delay, Root Dispersion, and Jitter

Windows Time status can expose timing-quality fields. Their names are related to NTP's measures of synchronization distance and uncertainty.

| Field | Meaning |
|---|---|
| Root Delay | Accumulated round-trip delay between the local system and the primary reference clock through the synchronization path. |
| Root Dispersion | An estimate of the accumulated maximum error or uncertainty associated with the time source. |
| Jitter | Variation in offset measurements. A high value can indicate unstable measurements or network conditions. |

These fields should be considered together with the source, stratum, last successful synchronization time, and direct offset measurements. A single field is not enough to determine whether a system is correctly synchronized.

## 8. NTP, SNTP, and Windows Time (W32Time)

NTP is the protocol family defined by the IETF. Simple Network Time Protocol (SNTP) is a simplified subset of NTP. Windows uses the Windows Time service (`W32Time`) to synchronize clocks and communicate using NTP/SNTP-compatible mechanisms.

Useful Windows commands include:

| Command | What it tells you |
|---|---|
| `w32tm /query /source` | The currently selected time source. |
| `w32tm /query /status` | Current synchronization status, including source and timing metadata. |
| `w32tm /query /configuration` | Effective Windows Time configuration. |
| `w32tm /stripchart /computer:<server> /samples:5 /dataonly` | Repeated measurements of the offset to another time source. |
| `w32tm /resync /rediscover` | Requests rediscovery of time sources and a synchronization operation. |

A successful resynchronization request should still be followed by checks of the selected source and service status.

For command syntax and configuration options, see Microsoft's [Windows Time Service Tools and Settings](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings?tabs=config).

## 9. Active Directory Time Synchronization

Active Directory uses a domain time hierarchy. Domain-joined computers normally follow this hierarchy automatically.

In a typical AD DS forest:

1. The PDC Emulator in the forest-root domain is configured to obtain time from a reliable external or hardware time source.
2. Other domain controllers synchronize through the domain hierarchy.
3. Domain member servers and client computers normally synchronize from an appropriate domain controller.

The PDC Emulator is a Flexible Single Master Operations (FSMO) role. Its role in the time hierarchy is separate from its other domain functions.

### 9.1 NT5DS

The Windows Time client type `NT5DS` means the computer obtains time through the Active Directory domain hierarchy. This is the normal configuration for most domain-joined Windows computers.

A registry value named `NtpServer` may still contain a value such as `time.windows.com,0x9`; that value alone does not prove that an `NT5DS` client is using that peer directly. Check the effective type and selected source.

### 9.2 PDC Emulator and External Time

The PDC Emulator in the forest-root domain is normally the point where the domain time hierarchy connects to an external reference. Microsoft recommends configuring the authoritative time server appropriately and ensuring that its source is reliable.

Before marking a domain controller as a reliable time source, verify that it actually holds the relevant PDC Emulator role. See [Configure an authoritative time server](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/configure-authoritative-time-server).

## 10. Time Zone Is Not Time Synchronization

A time zone determines how UTC is displayed as local civil time. Changing the time zone changes the displayed local time; it does not synchronize the underlying system clock with another computer or NTP server.

When diagnosing time problems, distinguish between:

- **Time-zone difference:** two systems display different local times because they use different time-zone settings.
- **Clock offset:** the underlying system clocks differ from one another.

Kerberos clock-skew problems concern the actual time difference between systems, not merely the time-zone labels shown in their user interfaces.

## 11. Troubleshooting Method

Use this order to avoid mixing network, DNS, protocol, and clock problems.

1. **Check the local configuration.** Inspect the time source, status, time zone, and current network interfaces.
2. **Check routing.** Confirm that the system has a route to the external time source. A Host-only network by itself normally provides private lab connectivity, not internet routing.
3. **Check DNS.** Resolve the configured NTP hostname before investigating the time protocol.
4. **Test the correct transport.** NTP normally uses UDP 123. Do not treat a failed TCP 123 test as proof that NTP is blocked.
5. **Measure the time source.** Use an NTP-aware tool such as `w32tm /stripchart`.
6. **Check the Windows Time configuration.** Review the configured peer, source-selection mode, and whether the computer should follow the AD hierarchy.
7. **Request synchronization.** Use `w32tm /resync /rediscover` when appropriate.
8. **Verify after synchronization.** Recheck the source and status, then measure the offset. Record the exact output and time of the test.

## 12. Key Takeaways

- NTP synchronizes clocks by exchanging timestamped UDP messages and estimating offset and delay.
- NTP normally uses UDP port 123; a TCP port test does not validate NTP.
- Stratum describes hierarchy distance, not guaranteed accuracy.
- Clock offset and clock drift are different: one is the current difference, the other is the rate of change.
- Windows domain clients normally use `NT5DS` and follow the AD time hierarchy.
- The forest-root PDC Emulator normally connects the domain hierarchy to an external or hardware time source.
- A time-zone change does not synchronize the clock.
- Always verify the selected source and synchronization status after requesting a resync.

## 13. Practical Lab Example

The concepts in this document were applied in a Windows Server 2025 Active Directory lab. The implementation record covers the DC01 NTP peer configuration, the additional NAT adapter, Windows Time commands, the `NT5DS` domain client configuration, and the measured client-to-domain-controller offset.

See [Windows Server Time Synchronization Configuration and Troubleshooting](https://github.com/hiro0209/windows-server-active-directory-lab/blob/main/docs/03-time-synchronization.md).

## 13. References

1. IETF, [RFC 5905 – Network Time Protocol Version 4: Protocol and Algorithms Specification](https://www.rfc-editor.org/rfc/rfc5905).
2. Microsoft Learn, [How the Windows Time Service Works](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/how-the-windows-time-service-works).
3. Microsoft Learn, [Windows Time Service Tools and Settings](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings?tabs=config).
4. Microsoft Learn, [Configure an authoritative time server](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/configure-authoritative-time-server).
5. Microsoft Learn, [Maximum tolerance for computer clock synchronization](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/maximum-tolerance-for-computer-clock-synchronization).
