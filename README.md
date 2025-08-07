# sockspeppet
A Performance-enhancing Proxy to transport web traffic over a constrained channel

Beyond content optimization on the server's side, a few interesting properties of the SOCKS 5 protocol can be exploited to improve browsing performance over a narrow and/or a high latency link.

## General Idea
This project intends to:
  * Specify an optimized version of the SOCKS protocol to adapt constrained channels (reduced protocol turnaround, parallel transactions over a single bearer, transparent compression when possible, remote DNS requests, client-side filtering).
  * Provide an implementation that can bridge a browser's HTTP requests over a balanced-mode AX.25 bearer.
