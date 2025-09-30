# HTTP Header Compression

This document presents a Stateful HTTP Header Compression method.

This is useful for Layer 7 optimization over a constrained channel.

## Compression algorithm
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
    Copy@{ shape: hex, label: "Bufferize until stream end" } 
    TxLine@{ shape: hex, label: "Bufferize Line" } 
    TxRef@{ shape: hex, label: "Bufferize dictionnary<br>reference" }
    TxFCred@{ shape: hex, label: "Bufferize full<br>Credentials" } 
    TxPCred@{ shape: hex, label: "Bufferize partial<br>Credentials" }
    Tx@{ shape: lean-l, label: "Send buffer" }
    TxDeflate@{ shape: lean-l, label: "Send Deflate(buffer)" }
    End@{ shape: framed-circle, label: "End" }

    Init --> ReadL
    ReadL --> DFirstLine
    DFirstLine -->|Yes| TxLine
    DFirstLine -->|No| DEmpty

    DEmpty -->|Yes| Copy
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
    Copy --> DDeflateBuf

    DDeflateBuf -->|Yes| TxDeflate
    DDeflateBuf -->|No| Tx

    Tx --> End
    TxDeflate --> End
```

## Decompression
