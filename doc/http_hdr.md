# HTTP Header Compression

This document presents a method which performs Stateful HTTP Header Compression.

This is useful for Layer 7 optimization on a 8-bit safe communication channel, with a bandwidth of 1200bps or below. 

Terrain feedback show 65~85% airtime reduction starting from the second HTTP query, with the same user agent.

To avoid race conditions, one dictionnary *MUST* be used per direction.
Once header transmission has ended, the compression scheme switches to transparent mode.

## Compression algorithm
When the DEFLATE algorithm is used to attempt compressing a payload, it *MUST* be configured in its `Best Compression` setting.

```mermaid
flowchart TD
    Init@{ shape: sm-circ, label: "Init" }
    ReadL@{ shape: lean-r, label: "Read Line" }
    DFirstLine@{ shape: diamond, label: "First Line?" }
    DEmpty@{ shape: diamond, label: "Empty Line?" }
    DAuthD@{ shape: diamond, label: "Authentication:<br />Digest?" }
    DAuthDC@{ shape: diamond, label: "Found same<br>Credentials?" }
    DProhibHdr@{ shape: diamond, label: "<b>Connection</b><br>or <b>Keep-Alive</b><br>header?"}
    DDicLine@{ shape: diamond, label: "Header line<br>in dictionnary?" }
    DDeflateBuf@{ shape: diamond, label: "Deflate(buffer) smaller<br>than buffer?" }
    Copy@{ shape: lean-l, label: "Transparently copy data<br> until connection closed" } 
    TxLine@{ shape: hex, label: "Bufferize Line" } 
    TxRef@{ shape: hex, label: "Bufferize dictionnary<br>reference" }
    TxFCred@{ shape: hex, label: "Bufferize full<br>Credentials" } 
    TxPCred@{ shape: hex, label: "Bufferize partial<br>Credentials" }
    Tx@{ shape: lean-l, label: "Send buffer" }
    TxDeflate@{ shape: lean-l, label: "Send <b>0xDF</b> then<br>Deflate(buffer)" }
    End@{ shape: framed-circle, label: "End" }

    Init --> ReadL
    ReadL --> DFirstLine
    DFirstLine -->|Yes| TxLine
    DFirstLine -->|No| DEmpty

    DEmpty -->|Yes| DDeflateBuf
    DEmpty -->|No| DAuthD

    DAuthD -->|Yes| DAuthDC
    DAuthD -->|No| DProhibHdr

    DProhibHdr -->|Yes| ReadL
    DProhibHdr -->|No| DDicLine

    DAuthDC -->|Yes| FmtCred(Format variable<br>attributes)
    DAuthDC -->|No| StoreDCred(Store invariable<br>attributes)
    FmtCred --> TxPCred
    StoreDCred --> FormatDC(Format digest<br>attributes)
    FormatDC --> TxFCred
    TxFCred --> ReadL
    TxPCred --> ReadL

    DDicLine -->|Yes| TxRef
    DDicLine -->|No| StoreLine(Store Line in dictionnary)

    StoreLine --> TxLine
    TxLine --> ReadL
    
    TxRef --> ReadL

    DDeflateBuf -->|Yes| TxDeflate
    DDeflateBuf -->|No| Tx

    Tx --> Copy
    TxDeflate --> Copy
    Copy --> End
```

## Decompression
