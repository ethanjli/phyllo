# Transport

Data transport protocol functionality is decomposed into abstraction layers. Each layer exchanges data payloads [encapsulated](https://book.systemsapproach.org/foundation/architecture.html#encapsulation) as blobs of bytes together with some other associated data exchanged between peers. Wherever possible, layers also exchange a [demultiplexing key](https://book.systemsapproach.org/foundation/architecture.html#multiplexing-and-demultiplexing) together with each data payload to specify the protocol which should be used to handle the payload.

## Stream

### Basic Description

- Data unit name: Stream
- Layer name: Stream Link
- Protocol type code: `0x11`
- Purpose: Continuous exchange of arbitrary bytes.
- Services required from below:
    - Physical transport
- Services provided for above:
    - In-order byte streaming

### Service Interface

### Peer Interface


## Frame

### Basic Description

- Data unit name: Frame
- Layer name: Frame Link
- Protocol type code: `0x20`
- Purpose: Framing of discrete payloads of arbitrary bytes exchanged over a continuous byte stream.
- Services required from below:
    - In-order byte streaming
- Services provided for above:
    - Connectionless in-order byte buffer exchange

### Service Interface

### Peer Interface


## Datagram

### Basic Description

- Data unit name: Datagram
- Layer name: Datagram Link
- Protocol type code: `0x21`
- Purpose: Multiplexing of multiple communication protocols over a point-to-point link.
- Services required from below:
    - Byte buffer exchange
- Services provided for above:
    - Byte buffer exchange
    - Payload protocol type multiplexing

### Service Interface

### Peer Interface


## Validated Datagram

### Basic Description

- Data unit name: Validated Datagram
- Layer name: Validated Datagram Link
- Protocol type code: `0x22`
- Purpose: Error checking of payload contents and type.
- Services required from below:
    - Byte buffer exchange
    - Protocol type multiplexing
- Services provided for above:
    - Byte buffer exchange
    - Payload protocol type multiplexing
    - Payload data error detection

### Service Interface

### Peer Interface


## Reliable Buffer

### Basic Description

- Data unit name: Reliable Buffer
- Layer name: Reliable Buffer Link
- Protocol type code: `0x23`
- Purpose: Reliable exchange of byte buffers.
- Services required from below:
    - Byte buffer exchange
    - Protocol type multiplexing
    - Data error detection
- Services provided for above:
    - Connection-oriented + reliable byte buffer exchange
    - Payload protocol type multiplexing

### Service Interface

### Peer Interface


## Ported Buffer

### Basic Description

- Data unit name: Ported Buffer
- Layer name: Ported Buffer Link
- Protocol type code: `0x24`
- Purpose: Multiplexing of byte buffer channels between different applications.
- Services required from below:
    - Byte buffer exchange
    - Protocol type multiplexing
- Services provided for above:
    - Payload application multiplexing

### Service Interface

### Peer Interface