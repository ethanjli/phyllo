# Transport

Data transport protocol functionality is decomposed into abstraction layers. Each layer exchanges data payloads [encapsulated](https://book.systemsapproach.org/foundation/architecture.html#encapsulation) as blobs of bytes together with some other associated data exchanged between peers. Above the frame layer, transport layers also exchange a [demultiplexing key](https://book.systemsapproach.org/foundation/architecture.html#multiplexing-and-demultiplexing) together with each data payload to specify how the payload should be handled.

## Conventional Configurations

Conventional transport stacks consist of two sub-stacks:

- The __Medium__ sub-stack provides exchange of byte buffers over an I/O interface.
- The __Logical__ sub-stack, which is layered on top of the medium sub-stack, provides all other transport-level services.

Conventional stack configurations (with the top of the stack higher in the table and the bottom of the stack at the bottom of the table) are as follows:

| Logical Sub-Stack Layer                        | Full | Standard | Reduced | Minimal |
| ---------------------------------------------- | ---- | -------- | ------- | ------- |
| [Ported Buffer Link](#ported-buffer)           | Yes  | No       | No      | No      |
| [Reliable Buffer Link](#reliable-buffer)       | Yes  | Yes      | No      | No      |
| [Validated Datagram Link](#validated-datagram) | Yes  | Yes      | Yes     | No      |
| [Datagram Link](#datagram)                     | Yes  | Yes      | Yes     | Yes     |

| Medium Sub-Stack Layer                         | Stream |
| ---------------------------------------------- | ------ |
| [Frame Link](#frame)                           | Yes    |
| [Chunked Stream Link](#chunked-stream)         | Yes    |
| [Stream Link](#stream)                         | Yes    |

These conventional configurations are most suitable for different use cases:

- Full: when either peer has multiple independent applications which need to share a data link.
- Standard: when reliable transmission guarantees are needed (always guarantees exactly-once delivery).
- Reduced: when error detection is needed but reliable transmission guarantees are not needed (guarantees at-most-once delivery on a point-to-point link).
- Minimal: when the data link underlying the stream link already provides reliable transmission guarantees, and extremely high data throughput is required; or when the peer has extremely low memory.


## Stream

The __stream link__ is a generic specification for implementations to connect a pair of protocol stacks as peers either in software or over a physical medium (e.g. a serial connection through a USB cable). The stream link abstracts away any details of how bytes are exchanged as a byte stream over a physical medium. In some implementations, a serial connection may be wrapped within an explicit stream link. In other implementations, I/O code may be wrapped as an implicit stream link within calling code.

### Basic Description

- Data unit name: Byte
- Payload type: byte
- Layer name: Stream Link
- Protocol type: `bytes/stream`
- Protocol type code: `0x11`
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
- Protocol type: `bytes/chunk`
- Protocol type code: `0x12`
- Purpose: Delimited exchange of buffers of non-null bytes, using null bytes as delimiters.
- Payload demultiplexing key: none
- Services required from below:
    - Byte stream connection
- Services provided for above:
    - (Connectionless + in-order) Non-null byte buffer exchange

### Service Interface

Byte buffer interface:

- __send__ a byte buffer to the other peer. The byte buffer should not contain any null bytes.
- __has receive__ to check whether a byte buffer has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received byte buffer from the other peer.

### Peer Interface

Each byte buffer exchanged due to the service interface is exchanged as a payload wrapped by a delimiter:

| __Name__     | Delimiter | Payload      | Delimiter |
| ------------ | --------- | ------------ | --------- |
| __Length__   | 1         | ...          | 1         |
| __Contents__ | `0x00`    | Buffer bytes | `0x00`    |

The delimiter consists of a single null byte (`0x00`). The receiver will treat any null byte as a delimiter between bodies (i.e. as a framing marker); thus, bodies should not have null bytes or else they will be misinterpreted as multiple buffers instead of a single buffer. Peers will ignore empty (zero-length) payloads between ending sequences.

#### Sending
When a peer begins to send a byte buffer as a payload, it will first transmit a delimiter to interrupt and cancel any payload which might have been in the middle of being transmitted; it will then transmit the bytes of the payload; it will finally transmit another delimiter to mark the end of the payload.

#### Receiving
When a peer receives a delimiter, it will begin receiving subsequent bytes as payload contents until it reaches the next delimiter. When the next delimiter is reached, it will expose the full payload for consumption in the service interface; however, if two consecutive delimiters are received, the empty payload will not be exposed for consumption in the service interface.

#### Rationale
The use of a single-byte chunk delimiter makes it simple to detect chunk boundaries. The transmission of a delimiter before the payload is not strictly necessary for a point-to-point stream link such as a serial connection, but it improves receiving robustness in case there is contention on a layer underneath, whether due to contention by multiple hosts or due to contention by multiple sending threads on a single host.

#### Example
A sequence of two byte buffer payloads (`0x01`, `0x02 0x03`) will be transmitted as the following byte stream:

| __Name__     | Delimiter | Payload | Delimiter | Delimiter | Payload     | Delimiter |
| ------------ | --------- | ------- | --------- | --------- | ----------- | --------- |
| __Length__   | 1         | 1       | 1         | 1         | 2           | 1         |
| __Contents__ | `0x00`    | `0x01`  | `0x00`    | `0x00`    | `0x02 0x03` | `0x00`    |

The receiver of this byte stream will receive the same sequence of byte buffer payloads (`0x01`, `0x02 0x03`, `0x04 0x05 0x06`).

## Frame

The __frame link__ enables exchange of byte buffers with arbitrary contents over a chunked stream link, by encoding payloads with [consistent-overhead byte stuffing](https://en.wikipedia.org/wiki/Consistent_Overhead_Byte_Stuffing).

### Basic Description

- Data unit name: Frame
- Payload type: byte buffer
- Layer name: Frame Link
- Protocol type: `transport/frame`
- Protocol type code: `0x20`
- Purpose: Encoding of buffers of arbitrary bytes to remove null bytes.
- Payload demultiplexing key: none
- Services required from below:
    - Non-null byte buffer exchange
- Services provided for above:
    - Byte buffer exchange

### Service Interface

Byte buffer interface:

- __send__ a byte buffer to the other peer. The byte buffer should not contain any more than 254 bytes.
- __has receive__ to check whether a byte buffer has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received byte buffer from the other peer.

Error reporting:

- When received data cannot be decoded into a valid byte buffer, an error is reported to the layer above.

### Peer Interface

Each byte buffer exchanged due to the service interface is exchanged as a body encoded by [consistent overhead byte stuffing](https://en.wikipedia.org/wiki/Consistent_Overhead_Byte_Stuffing) (COBS), without anything else:

| __Name__     | Body                      |
| ------------ | ------------------------- |
| __Length__   | ...                       |
| __Contents__ | COBS-encoded buffer bytes |

COBS encoding ensures that the encoded representation of the payload does not contain any null bytes.

#### Sending
When a peer sends a byte buffer as a payload, it will first encode it using COBS and then send the resulting encoded byte buffer as a body.

#### Receiving
When a peer receives a byte buffer as a body, it will first decode it using COBS and then expose the full payload for consumption in the service interface.

#### Example
A byte buffer payload of `0x11 0x22 0x00 0x33` will be sent as the following COBS-encoded byte buffer:

| __Type__     | Overhead | Byte   | Byte   | Byte   | Byte   |
| ------------ | -------- | ------ | ------ | ------ | ------ |
| __Length__   | 1        | 1      | 1      | 1      | 1      |
| __Contents__ | `0x03`   | `0x11` | `0x22` | `0x02` | `0x33` |

The receiver of this byte buffer body will receive the same byte buffer payload `0x11 0x22 0x00 0x33`.


## Datagram

The __datagram link__ provides multiplexing and demultiplexing of multiple protocols over a data link, as well as detection of missing bytes in the payload.

### Basic Description

- Data unit name: Datagram
- Payload type: byte buffer
- Layer name: Datagram Link
- Protocol type: `transport/datagram`
- Protocol type code: `0x21`
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

Each byte buffer exchanged due to the service interface is exchanged as a payload together with a header:

| __Name__     | Length         | Type                       | Payload      |
| ------------ | -------------- | -------------------------- | ------------ |
| __Length__   | 1              | 1                          | ...          |
| __Contents__ | Payload Length | Payload Protocol Type Code | Buffer bytes |

The length field specifies the number of bytes in the payload and is represented as a single byte (0 - 255). The length field allows for detection of missing bytes in the payload, as a simple and partial form of error detection.

The type field specifies the protocol which should be used to handle the payload and is represented as a single byte (`0x00` - `0xff`).

#### Sending
When a peer sends a byte buffer as a payload, it will first compute the length of the payload and prepend a header consisting of the length and the payload type.

#### Receiving
When a peer receives a byte buffer as a body, it will first check the payload length against the length specified in the header and, if lengths are consistent, expose the full payload for consumption in the service interface.

#### Example
A generic byte buffer payload of `0x01 0x02 0x03 0x04 0x00` will be sent as the following byte buffer (as a generic byte buffer of type `bytes/buffer` has type code `0x10` and the payload has 5 bytes):

| __Type__     | Length   | Type   | Byte   | Byte   | Byte   | Byte   | Byte   |
| ------------ | -------- | ------ | ------ | ------ | ------ | ------ | ------ |
| __Length__   | 1        | 1      | 1      | 1      | 1      | 1      | 1      |
| __Contents__ | `0x05`   | `0x10` | `0x01` | `0x02` | `0x03` | `0x04` | `0x00` |

The receiver of this byte buffer body will receive the same generic byte buffer payload `0x01 0x02 0x03 0x04 0x00`.


## Validated Datagram

The __validated datagram link__ provides sophisticated error-checking with a [cyclic redundancy check](https://en.wikipedia.org/wiki/Cyclic_redundancy_check) (CRC) to ensure that corrupted received data is not passed up the protocol stack.

### Basic Description

- Data unit name: Validated Datagram
- Payload type: byte buffer
- Layer name: Validated Datagram Link
- Protocol type: `transport/validated_datagram`
- Protocol type code: `0x22`
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

Each byte buffer exchanged due to the service interface is exchanged as a payload together with a header:

| __Name__     | CRC                   | Type                       | Payload      |
| ------------ | --------------------- | -------------------------- | ------------ |
| __Section__  | CRC                   | Protected                  | Protected    |
| __Length__   | 4                     | 1                          | ...          |
| __Contents__ | Protected Section CRC | Payload Protocol Type Code | Buffer bytes |

The CRC field specifies the computed 32-bit CRC of the protected section (consisting of the type field and the payload) and is represented as a big-endian 32-bit number. The type field specifies the protocol which should be used to handle the payload and is represented as a single byte (`0x00` - `0xff`). The CRC field allows for detection of data corruption in the type or payload, as a comprehensive form of error detection. The 32-bit CRC is computed using polynomial `0x000001ed` ([Ray32sub8](https://users.ece.cmu.edu/~koopman/pubs/ray06_crcalgorithms.pdf), which is exceptionally fast and computational cost similar to a normal CRC16 computation by the precomputed table lookup algorithm. The CRC parameters are as follows:

- Width: 32 bits
- Polynomial: `0x000001ed`
- Reflect Input (reverse data bytes): True
- Initial Value (XOR In): `0xffffffff`
- Reflect Output (reverse CRC result before final XOR): True
- Final XOR value (XOR Out): `0xffffffff`
- Expected CRC with input of `0x31 0x32 0x33 0x34 0x35 0x36 0x37 0x38 0x39`: `0xb303b455`

The type field specifies the protocol which should be used to handle the payload and is represented as a single byte (`0x00` - `0xff`).

#### Sending
When a peer sends a byte buffer as a payload, it will first build the protected section consisting of the payload protocol type code followed by the payload, then compute the CRC of the protected section, and then prepend a header consisting of the crc and the payload protocol type code.

#### Receiving
When a peer receives a byte buffer, it will parse the first 5 bytes as the header fields. Then it will check the CRC of the protected section against the CRC specified in the header and, if CRCs are consistent, expose the full payload for consumption in the service interface.

#### Example
A generic byte buffer payload of `0x01 0x02 0x03 0x04 0x00` will be sent as the following byte buffer (as a generic byte buffer of type `bytes/buffer` has type code `0x10`):

| __Type__     | CRC                   | Type   | Byte   | Byte   | Byte   | Byte   | Byte   |
| ------------ | --------------------- | ------ | ------ | ------ | ------ | ------ | ------ |
| __Length__   | 4                     | 1      | 1      | 1      | 1      | 1      | 1      |
| __Contents__ | `0xd2 0xaa 0x86 0xd0` | `0x10` | `0x01` | `0x02` | `0x03` | `0x04` | `0x00` |

The receiver of this byte buffer body will receive the same generic byte buffer payload `0x01 0x02 0x03 0x04 0x00`.


## Reliable Buffer

The __reliable buffer link__ provides reliable transmission of data with an [automatic repeat query](https://en.wikipedia.org/wiki/Automatic_repeat_request) (ARQ) mechanism for error correction.

WARNING: the speciication for the reliable buffer link is not yet finalized and will be changed.

### Basic Description

- Data unit name: Reliable Buffer
- Payload type: byte buffer
- Layer name: Reliable Buffer Link
- Protocol type: `transport/reliable_buffer`
- Protocol type code: `0x23`
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

The two peers in a Reliable Buffer Link essentially communicate by the [Go-Back-N](https://en.wikipedia.org/wiki/Go-Back-N_ARQ) (GBN) [sliding window algorithm](https://en.wikipedia.org/wiki/Sliding_window_protocol) for [automatic repeat query](https://en.wikipedia.org/wiki/Automatic_repeat_request) (ARQ), optionally with negative acknowledgements and selective acknowledgements.

Each byte buffer exchanged due to the service interface is exchanged as a payload together with a header:

| __Name__     | Seq Num         | Ack Num                | Flags | Type                       | Ext             | Payload      |
| ------------ | --------------- | ---------------------- | ----- | -------------------------- | --------------- | ------------ |
| __Length__   | 1               | 1                      | 1     | 1                          | ...             | ...          |
| __Contents__ | Sequence Number | Acknowledgement Number | Flags | Payload Protocol Type Code | Extended Header | Buffer bytes |

The seq num field is an incrementing counter associated with the payloads sent by a peer, represented as a one-byte number (0 - 255) which rolls over from 255 to 0. It increments by one for each new payload to be sent through the service interface on the sending peer.

The ack num field is a monotonically increasing counter which specifies the seq num expected to accompany the next payload received by the sending peer. Like the seq num field, the ack num field rolls over from 255 to 0. It increments by one for each new payload to be acknowledged as having been successfully received over the Reliable Buffer Link.

The flags field is a bitfield of 8 boolean flags, represented as a one-byte number (`0b00000000` - `0b11111111`). It consists of the following flags:

| Name | Position | Meaning                                                                                      |
| ---- | -------- | -------------------------------------------------------------------------------------------- |
| FIN  | 0 (LSB)  | Initiates teardown of connection; last Reliable Buffer from sender                           |
| SYN  | 1        | Initiates setup of connection; first Reliable Buffer from sender                             |
| NOS  | 2        | Ignore the Seq Num field (payload will not be transmitted with reliability guarantees)       |
| ACK  | 3        | Ack Num field should be read as a signal of the next sequence number expected to be received |
| NAK  | 4        | Negative acknowledgement; request other peer to re-send all in-flight Reliable Buffers       |
| SAK  | 5        | Selective acknowledgement; treat Ack Num field as selective instead of cumulative.           |
| RST  | 6        | Reset the connection                                                                         |
| EXT  | 7 (MSB)  | Extended header is present                                                                   |

The type field specifies the protocol which should be used to handle the payload and is represented as a single byte (`0x00` - `0xff`).

The ext section only exists if the EXT flag is set in the flags field; otherwise, the payload immediately follows the type field. This section can be used to extend the functionality of the Reliable Buffer Link, but is not yet defined. The first byte of the ext section specifies the length of the ext section.

#### Sending
When a peer sends a byte buffer as a payload, it will first set the header fields and prepend the resulting header:

- If the payload is sent in reliable transmission mode, then the NOS flag will remain unset (its bit will be 0), and the seq num field of the header will be set as the increment on the sequence number used for the last payload sent in reliable transmission mode; an acknowledgement of receipt will be required from the peer. Otherwise, the payload is sent in unreliable transmission mode, and the NOS flag will be set (its bit will be 1), and the sequence number can be of any value - by convention, the sequence number should be 0; no acknowledgement of receipt will be expected from the peer.
- The ACK flag will be set, and the ack num field will be set as the sequence number which the sending peer expects to receive in the header of the next reliably-transmitted payload.

The sender will then send the header and payload, and it will wait up to some timeout to receive a Reliable Buffer from the other peer whose ACK flag is set, whose NAK flag is unset, and whose ack num field is greater than or equal to the seq num of the Reliable Buffer which it had sent. If such an acknowledgement is not received before the timeout, then the sender will retransmit all Reliable Buffers which have not yet been acknowledged.

#### Receiving
When a peer receives a byte buffer, it will parse the first 4 bytes as the header fields.

If the NOS flag is unset and the seq num field does not match the value of the next expected sequence number, the Reliable Buffer will be discarded, and the payload will not be exposed for consumption in the service interface. Optionally, the receiver may send a packet with the ACK and NAK flags set and the ack num field set to the value of the next expected sequence number, to signal to the other peer that it needs to retransmit its Reliable Buffers starting from the one whose seq num field corresponds to the next expected sequence number.

Then it will check the EXT flag to determine whether an extended header is present; if so, it will check the first byte of the extended header to determine the length of the extended header and thus what the starting position of the payload is. Then it will parse the extended header section and expose the full payload for consumption in the service interface.

#### Example
A generic byte buffer payload of `0x11 0x22 0x33` will be sent as the following byte buffer from a peer whose last sent Reliable Buffer was 9 and whose last received Reliable Buffer was 249:

| __Type__     | Seq Num | Ack Num | Flags        | Type   | Byte  | Byte   | Byte   |
| ------------ | ------- | ------- | ------------ | ------ | ----- | ------ | ------ |
| __Length__   | 1       | 1       | 1            | 1      | 1     | 1      | 1      |
| __Contents__ | 10      | 250     | `0b00001000` | `0x10` | `0x11`| `0x22` | `0x33` |

The receiver of this byte buffer body will receive the same generic byte buffer payload `0x11 0x22 0x33`.


## Ported Buffer

The __ported buffer link__ provides multiplexing of multiple communication channels between any number of applications running on a pair of peers, over a single Reliable Buffer Link or Validated Datagram Link or Datagram Link between the two peers.

WARNING: the speciication for the ported buffer link is not yet finalized and may be changed.

### Basic Description

- Data unit name: Ported Buffer
- Payload type: byte buffer
- Layer name: Ported Buffer Link
- Protocol type: `transport/ported_buffer`
- Protocol type code: `0x24`
- Purpose: Multiplexing of byte buffer channels between different applications.
- Payload demultiplexing key: (sender port, receiver port) pair
- Services required from below:
    - Byte buffer exchange
    - Protocol type multiplexing
- Services provided for above:
    - Byte buffer exchange
    - Payload application multiplexing

### Service Interface

Ported typed byte buffer interface:

- __send__ a byte buffer of a specified protocol type and send port and receive port to the other peer; send mode (reliable transmission with automatic retransmission until acknowledgement, or unreliable transmission without acknowledgement) also may be specified by the caller and will be used if the ported buffer link is placed over a reliable buffer link.
- __has receive__ to check whether a byte buffer has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received byte buffer (and the send port and receive port and protocol type) from the other peer.

### Peer Interface

Each byte buffer exchanged due to the service interface is exchanged as a payload together with a header:

| __Name__     | Send Port          | Receive Port         | Type                       | Payload      |
| ------------ | ------------------ | -------------------- | -------------------------- | ------------ |
| __Length__   | 1                  | 1                    | 1                          | ...          |
| __Contents__ | Sender Port Number | Receiver Port Number | Payload Protocol Type Code | Buffer bytes |

The send port field specifies the application port on the sending peer from which the payload is sent and is represented as a single byte (0 - 255). The other peer can send data to this port.

The receive port field specifies the application port on the receiving peer to which the payload is sent and is represented as a single byte (0 - 255). The other peer is expected to be bound to this port to send and receive data.

The type field specifies the protocol which should be used to handle the payload and is represented as a single byte (`0x00` - `0xff`).

#### Sending
When a peer sends a byte buffer as a payload, it will first prepend a header consisting of the send port, receive port, and the payload type.

#### Receiving
When a peer receives a byte buffer as a body, it will first parse the header and expose the full payload for consumption in the service interface.

#### Example
A generic byte buffer payload of `0x01 0x02 0x03 0x04 0x00` will be sent as the following byte buffer (as a generic byte buffer of type `bytes/buffer` has type code `0x10` and the payload has 5 bytes):

| __Type__     | Length   | Type   | Byte   | Byte   | Byte   | Byte   | Byte   |
| ------------ | -------- | ------ | ------ | ------ | ------ | ------ | ------ |
| __Length__   | 1        | 1      | 1      | 1      | 1      | 1      | 1      |
| __Contents__ | `0x05`   | `0x10` | `0x01` | `0x02` | `0x03` | `0x04` | `0x00` |

The receiver of this byte buffer body will receive the same generic byte buffer payload `0x01 0x02 0x03 0x04 0x00`.
