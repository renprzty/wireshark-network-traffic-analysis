# Portfolio Blurb

**Wireshark Network Traffic Analysis — DNS, QUIC, TCP & TLS**

Analyzed a controlled web session in Wireshark from DNS resolution through encrypted transport. Identified a browser selecting IPv6/QUIC over UDP/443, then designed a controlled IPv4/HTTP/1.1 test with `curl` to inspect the TCP three-way handshake, TLS Client Hello/SNI, encrypted application data, retransmission behavior, conversation statistics, and I/O traffic patterns. The analyzed TCP session contained 20 packets (~7 kB) with no detected retransmissions.
