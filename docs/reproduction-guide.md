# Reproduction Guide

> Capture only traffic you are authorized to inspect. Packet captures can contain sensitive information.

## Part A — Observe normal browser behavior

1. Start Wireshark on the active network interface.
2. Visit `https://example.com` in a browser.
3. Stop the capture after the page loads.
4. Inspect DNS traffic with:

```text
dns
```

5. Check returned A/AAAA addresses.
6. If an IPv6 destination is used, inspect it with:

```text
ipv6.addr == <observed IPv6 address>
```

7. If QUIC appears, narrow the view with:

```text
quic && ipv6.addr == <observed IPv6 address>
```

Look for Initial, Handshake, and Protected Payload packets.

## Part B — Create a controlled TCP/TLS session

1. Open Command Prompt.
2. Flush the DNS resolver cache:

```powershell
ipconfig /flushdns
```

3. Start a fresh Wireshark capture.
4. Run:

```powershell
curl.exe -4 --http1.1 https://example.com/ -o NUL
```

5. Stop the capture immediately after the command completes.
6. Find TCP/443 traffic:

```text
tcp.port == 443
```

7. Identify the relevant SYN/SYN-ACK pair and inspect the TCP stream index.
8. Isolate that connection:

```text
tcp.stream eq <stream-number>
```

9. Confirm the three-way handshake and inspect the TLS Client Hello.
10. Expand the `server_name` extension and confirm the SNI hostname.
11. Check retransmissions:

```text
tcp.stream eq <stream-number> && tcp.analysis.retransmission
```

12. Open **Statistics → Conversations → TCP** and limit results to the display filter.
13. Open **Statistics → I/O Graphs**, filter to the TCP stream, and use a short interval such as 100 ms for a short-lived connection.

## Expected differences

Results can vary by operating system, browser, DNS resolver, network, server/CDN behavior, IPv6 availability, and protocol negotiation. A browser may choose QUIC/HTTP/3 while a forced `curl --http1.1` request follows TCP/TLS instead.
