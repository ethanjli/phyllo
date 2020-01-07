# Transport

Data transport protocol functionality is decomposed into abstraction layers. Each layer exchanges data payloads [encapsulated](https://book.systemsapproach.org/foundation/architecture.html#encapsulation) as blobs of bytes together with some other associated data exchanged between peers. Above the frame layer, transport layers also exchange a [demultiplexing key](https://book.systemsapproach.org/foundation/architecture.html#multiplexing-and-demultiplexing) together with each data payload to specify how the payload should be handled.

Conventional stack configurations (with the top of the stack higher in the table and the bottom of the stack at the bottom of the table) are as follows:

| Transport Layer                                | Full | Standard | Minimal |
| ---------------------------------------------- | ---- | -------- | ------- |
| [Ported Buffer Link](#ported-buffer)           | Yes  | No       | No      |
| [Reliable Buffer Link](#reliable-buffer)       | Yes  | Yes      | No      |
| [Validated Datagram Link](#validated-datagram) | Yes  | Yes      | No      |
| [Datagram Link](#datagram)                     | Yes  | Yes      | Yes     |
| [Frame Link](#frame)                           | Yes  | Yes      | Yes     |
| [Chunked Stream Link](#chunked-stream)         | Yes  | Yes      | Yes     |
| [Stream Link](#stream)                         | Yes  | Yes      | Yes     |

These conventional configurations are most suitable for different use cases:

- Full: when either peer has multiple independent applications which need to share a data link.
- Standard: when reliable transmission guarantees are needed.
- Minimal: when the data link underlying the stream link already provides reliable transmission guarantees, and extremely high data throughput is required; or when the peer has extremely low memory.


## Stream

The __stream link__ is a generic specification for implementations to connect a pair of protocol stacks as peers either in software or over a physical medium (e.g. a serial connection through a USB cable). The stream link abstracts away any details of how bytes are exchanged as a byte stream over a physical medium. In some implementations, a serial connection may be wrapped within an explicit stream link. In other implementations, I/O code may be wrapped as an implicit stream link within calling code.

### Basic Description

- Data unit name: Byte
- Payload type: byte
- Layer name: Stream Link
- Protocol type code: `0x11` (`generic/stream`)
- Purpose: Continuous exchange of arbitrary bytes.
- Payload demultiplexing key: none
- Services required from below:
    - None, or implementation-dependent
- Services provided for above:
    - Byte stream connection

### Service Interface

Byte stream interface:

- __write__ a byte to put it on the (outbound) send buffer to eventually send to the other peer.
- __has read__ to check whether the (inbound) receive buffer has a byte received from the other peer which is ready to be consumed.
- __read__ to consume the next byte from the (inbound) receive buffer.

### Peer Interface

The peer interface is determined by the specific implementation of the stream link.


## Chunked Stream

The __chunked stream link__ delimits byte buffer consisting of non-null bytes, by inserting a null byte to mark the beginning and end of the chunk.

### Basic Description

- Data unit name: Stream Chunk
- Payload type: stream chunk
- Layer name: Chunked Stream Link
- Protocol type code: `0x12` (`generic/stream_chunk`)
- Purpose: Delimited exchange of buffers of non-null bytes, using null bytes as delimiters.
- Payload demultiplexing key: none
- Services required from below:
    - Byte stream connection
- Services provided for above:
    - (Connectionless + in-order) Non-null byte buffer exchange

### Service Interface

Byte buffer interface:

- __send__ a byte buffer to the other peer.
- __has receive__ to check whether a byte buffer has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received byte buffer from the other peer.

### Peer Interface


## Frame

The __frame link__ enables exchange of byte buffers with arbitrary contents over a chunked stream link, by encoding payloads with [consistent-overhead byte stuffing](https://en.wikipedia.org/wiki/Consistent_Overhead_Byte_Stuffing).

### Basic Description

- Data unit name: Frame
- Payload type: byte buffer
- Layer name: Frame Link
- Protocol type code: `0x20` (`transport/frame`)
- Purpose: Encoding of buffers of arbitrary bytes to remove null bytes.
- Payload demultiplexing key: none
- Services required from below:
    - Non-null byte buffer exchange
- Services provided for above:
    - Byte buffer exchange

### Service Interface

Byte buffer interface:

- __send__ a byte buffer to the other peer.
- __has receive__ to check whether a byte buffer has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received byte buffer from the other peer.

Error reporting:

- When received data cannot be decoded into a valid byte buffer, an error is reported to the layer above.

### Peer Interface


## Datagram

The __datagram link__ provides multiplexing and demultiplexing of multiple protocols over a data link, as well as detection of missing bytes in the payload.

### Basic Description

- Data unit name: Datagram
- Payload type: byte buffer
- Layer name: Datagram Link
- Protocol type code: `0x21` (`transport/datagram`)
- Purpose: Multiplexing of multiple communication protocols over a point-to-point link.
- Payload demultiplexing key: protocol type code
- Services required from below:
    - Byte buffer exchange
- Services provided for above:
    - Byte buffer exchange
    - Payload protocol type multiplexing
    - Payload missing data detection

### Service Interface

Typed byte buffer interface:

- __send__ a byte buffer of a specified protocol type to the other peer.
- __has receive__ to check whether a byte buffer has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received byte buffer (and its protocol type) from the other peer.

Payload length checking:

- Byte buffers whose lengths are inconsistent with the lengths specified in the header metadata are not available to be consumed by _receive_.

Error reporting:

- Any errors reported by the layer below are passed to the layer above.
- When a byte buffer with inconsistent length is received, an error is reported to the layer above.

### Peer Interface


## Validated Datagram

The __validated datagram link__ provides sophisticated error-checking with a [cyclic redundancy check](https://en.wikipedia.org/wiki/Cyclic_redundancy_check) (CRC) to ensure that corrupted received data is not passed up the protocol stack.

### Basic Description

- Data unit name: Validated Datagram
- Payload type: byte buffer
- Layer name: Validated Datagram Link
- Protocol type code: `0x22` (`transport/validated_datagram`)
- Purpose: Error checking of payload contents and type.
- Payload demultiplexing key: protocol type code
- Services required from below:
    - Byte buffer exchange
    - Protocol type multiplexing
- Services provided for above:
    - Byte buffer exchange
    - Payload protocol type multiplexing
    - Payload data error detection

### Service Interface

Typed byte buffer interface:

- __send__ a byte buffer of a specified protocol type to the other peer.
- __has receive__ to check whether a byte buffer has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received byte buffer (and its protocol type) from the other peer.

Payload length checking:

- Byte buffers whose CRCs are inconsistent with the CRCs specified in the header metadata are not available to be consumed by _receive_.

Error reporting:

- Any errors reported by the layer below are passed to the layer above.
- When a byte buffer with inconsistent CRC is received, an error is reported to the layer above.

### Peer Interface


## Reliable Buffer

The __reliable buffer link__ provides reliable transmission of data with an [automatic repeat query](https://en.wikipedia.org/wiki/Automatic_repeat_request) (ARQ) mechanism for error correction.

### Basic Description

- Data unit name: Reliable Buffer
- Payload type: byte buffer
- Layer name: Reliable Buffer Link
- Protocol type code: `0x23` (`transport/reliable_buffer`)
- Purpose: Reliable exchange of byte buffers.
- Payload demultiplexing key: protocol type code
- Services required from below:
    - Byte buffer exchange
    - Protocol type multiplexing
    - Data error detection
- Services provided for above:
    - (Connection-oriented + reliable) byte buffer exchange
    - Payload protocol type multiplexing

### Service Interface

Reliable typed byte buffer interface:

- __send__ a byte buffer of a specified protocol type to the other peer; send mode (reliable transmission with automatic retransmission until acknowledgement, or unreliable transmission without acknowledgement) also must be specified by the caller.
- __has receive__ to check whether a byte buffer has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received byte buffer (and its protocol type) from the other peer.

Error reporting:

- Any errors reported by the layer below are masked by ARQ.
- Does Reliable Buffer Link generate any errors of its own (either reported above or reported to the peer)?

### Peer Interface


## Ported Buffer

### Basic Description

- Data unit name: Ported Buffer
- Payload type: byte buffer
- Layer name: Ported Buffer Link
- Protocol type code: `0x24` (`transport/ported_buffer`)
- Purpose: Multiplexing of byte buffer channels between different applications.
- Payload demultiplexing key: (sender port, receiver port) pair
- Services required from below:
    - Byte buffer exchange
    - Protocol type multiplexing
- Services provided for above:
    - Byte buffer exchange
    - Payload application multiplexing

### Service Interface

### Peer Interface