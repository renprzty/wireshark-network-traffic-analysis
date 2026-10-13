# Analysis Notes

## DNS

The DNS A response for `example.com` returned `104.20.23.154` and `172.66.147.243`. The captured DNS response time was 84.2681 ms. The exchange used UDP, with the DNS server on port 53.

## Why the first TCP search did not match

The browser session selected an IPv6 destination (`2606:4700:10::ac42:93f3`) and used QUIC over UDP/443. This was the key troubleshooting finding: DNS resolution had succeeded, but the browser did not follow the initially expected IPv4/TCP path.

The QUIC capture showed Initial, Handshake, and Protected Payload packets. The public client IPv6 address is redacted in the repository screenshot.

## Controlled TCP/TLS comparison

A separate capture was created after flushing the DNS cache. The following command forced IPv4 and HTTP/1.1:

```powershell
curl.exe -4 --http1.1 https://example.com/ -o NUL
```

The resulting HTTPS connection used `172.66.147.243:443` and was isolated as `tcp.stream eq 37`.

### TCP establishment

Observed sequence:

1. Client SYN
2. Server SYN/ACK
3. Client ACK

The elapsed time from the initial SYN to the final ACK was approximately 32.7 ms.

### TLS

After TCP establishment, the capture showed a TLS Client Hello with SNI `example.com`, followed by Server Hello/certificate/key-exchange traffic and encrypted Application Data. The Client Hello advertised 21 cipher suites.

### Retransmissions

The filter below returned no packets:

```text
tcp.stream eq 37 && tcp.analysis.retransmission
```

This supports the limited conclusion that Wireshark did not detect TCP retransmissions in this specific analyzed stream.

### Conversation statistics

- 20 total packets
- ~7 kB total traffic
- Client → Server: 10 packets / 977 bytes
- Server → Client: 10 packets / ~6 kB

The server therefore accounted for most of the transferred byte volume despite an equal packet count in each direction.

### I/O Graph

The graph was filtered to TCP stream 37, configured for byte measurement, and displayed using 100 ms intervals. The largest visible burst was approximately 5 kB per 100 ms. The graph is used only as a traffic-volume visualization; individual spikes are not assigned to particular TLS or HTTP messages without timestamp correlation.
