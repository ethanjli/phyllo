# phyllo

Phyllo is a point-to-point communication protocol suite and application framework specification designed for use with embedded systems.

Phyllo provides a specification for high-throughput + reliable + asynchronous message exchange between exactly two (2) peers. Phyllo also provides a specification for applications to communicate with each other across a pair of hosts (which may be either an embedded device or a regular computer) or distributed across multiple hosts.

This repository manages specification of the phyllo protocol suite and application framework.


## Design

Phyllo is a communication protocol suite consisting of __layers__ which can be mixed-and-matched to suit the service requirements and computational capabilities of hosts, ranging from 8-bit microcontrollers to computers. To support such interoperability, each layer of a communication protocol is designed to be stackable on various other phyllo layers; each such layer is called a __link__, in the general sense of a [data link](https://en.wikipedia.org/wiki/Data_link) which connects one location to another to transmit and receive digital information between the two locations.

Phyllo layers are designed such that that they can be decoupled from each other and grafted onto other protocol libraries, whether for transport layer communication or application layer frameworks. If you want to bring your own portion of the communication protocol stack or application framework stack, it should generally be possible to use it with phyllo layers to supply any additional desired functionality.

### Transport

Phyllo's transport layers are designed exclusively for communication between exactly two peers in a point-to-point connection.

#### Correspondence to OSI Model
If the phyllo transport layers are stacked directly on top of a physical layer such as a serial data link (e.g. on a USB cable), then the peers are directly connected to each other with nothing in between, and the layers would be in [layer 2](https://en.wikipedia.org/wiki/Data_link_layer) (data link layer) of the [OSI model](https://en.wikipedia.org/wiki/OSI_model). If phyllo's transport layers are stacked on top of some other external transport-layer protocol (such as TCP or UDP), then the peers may be connected through a network, in which case they would all be at layer 5 (presentation layer) of the OSI model. If phyllo's "transport layers" are stacked on top of some other external network-layer protocol (such as [PJON](https://www.pjon.org/)), in which case they would all be at layer 4 (transport layer) of the OSI model.

#### Framing
Phyllo provides a way to transport discrete byte sequences in __frames__ over a byte stream, such as what might be exposed by a serial link or a TCP connection. Specifically, the byte sequence is encoded using [Consistent Overhead Byte Stuffing](https://en.wikipedia.org/wiki/Consistent_Overhead_Byte_Stuffing) ([Cheshire S, & Baker M, IEEE/ACM Trans. Netw. 1999](http://www.stuartcheshire.org/papers/COBSforToN.pdf)). In the OSI model, a frame link placed on top of a serial link (layer 1) would be in layer 2 (data link layer, media access control sublayer). Each frame is always assumed to contain exactly one datagram (see next).

#### Protocol Multiplexing
Phyllo provides a connectionless way to transport byte payloads in __datagrams__ within frames, such as what is exposed by the frame link, or within other datagrams, such as what might be exposed by a UDP link. Regular datagrams merely provide a way to multiplex independent protocols on top, so that a single frame link can support multiple communication protocols. In the OSI model, a datagram link placed on top of a frame link running over a physical (layer 1) link would be in layer 2 (data link layer, logical link control control sublayer).
 
#### Reliability
Phyllo provides a connectionless way to transport error-checked byte payloads in __validated datagrams__ within datagrams, such as what is exposed by the datagram link. Validated datagrams can also multiplex many independent protocols on top. In the OSI model, validated datagram link placed on top of a datagram link running over a frame link running over a physical (layer 1) link would be in layer 2 (data link layer, logical link control control sublayer).

Phyllo provides a connection-oriented way to reliably transport byte payloads in __reliable buffers__ within error-checked datagrams, such as what is exposed by the validated datagram link. This provides [automatic repeat query](https://en.wikipedia.org/wiki/Automatic_repeat_request) (ARQ) for error correction, as well as a guarantee of same-order delivery. Reliable buffers can also multiplex many independent protocols on top. In the OSI model, a datagram link or validated datagram link placed on top of a validated datagram link running over a datagram link running over a frame link running over a physical link would be in layer 2 (data link layer, logical link control control sublayer).

#### Application Multiplexing
Phyllo provides an optional way to multiplex communication between different peer applications over a pair of __ports__ on the two hosts at the ends of the port link. The port for the application at one host may have a different value from the port for the application at the other host.


### Presentation

Phyllo provides ways to use various presentation layer protocols for serialization and deserialization of application data as __documents__ over any phyllo transport layer. Canonically, [MessagePack](https://msgpack.org/index.html) is used for efficient encoding and transmission of small schema-less documents. Alternative serialization schemes can also be used.

### Application

#### Publish/Subscribe Messaging
Phyllo provides a way to pass __messages__ between peers in independent channels multiplexed in a hierarchical namespace over a document link, acting as a pub-sub message bus between workers at the ends of the message link.

#### Request/Response Transaction
Phyllo provides frameworks to write client/server application programs with request/reply message transactions over a message link. The [remote procedure call](https://en.wikipedia.org/wiki/Remote_procedure_call) (RPC) framework uses the message link's namespace to name procedures, and adds a channel ID and sequence number for sequential and reliable execution of multiple transactions concurrently. The resource ([RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer)) framework uses the message link's namespace as a uniform resource locator to uniquely name resources, and adds a verb code to name different types of requests and responses.


## Implementations

Currently, the following implementations exist:

- A C++ library providing a standard protocol implementation and application framework for embedded devices, along with an I/O layer for serial communication in the Arduino framework. This implementation heavily uses the [Embedded Template Library](https://www.etlcpp.com/).
- A Python library providing a standard protocol implementation and application framework for computers, along with an I/O layer for various communication I/O implementations. The protocol implementation has a [Sans I/O](https://sans-io.readthedocs.io/)-based design.


## Scope and Limitations

Currently, phyllo does:

- Specify a way (ChunkedStreamLink) to transport up to 255-byte chunks of non-null bytes over a stream.
- Specify a way (FrameLink) to transport up to 254-byte frames of arbitrary bytes over a stream.
- Specify a way (DatagramLink) to transport up to 252-byte arbitrary datagram payloads over a FrameLink.
- Specify a way (ValidatedDatagramLink) to transport up to 247-byte arbitrary validated datagram payloads over a DatagramLink with an exceptionally fast 32-bit CRC using polynomial 0x000001ED ([Ray32sub8](https://users.ece.cmu.edu/~koopman/pubs/ray06_crcalgorithms.pdf), with computational cost similar to a normal CRC16 computation) guaranteeing that any data transmission errors of up to 5 bits per datagram (i.e. HD=6) will be caught, and that any data transmission errors of up to 7 bits per datagram (i.e. HD=8) will be caught for datagram payloads of up to 99 bytes.
- Partially specify a way (ReliableBufferLink) to reliably transmit up to 243-byte arbitrary byte buffer payloads over a ValidatedDatagramLink.
- Specify a way (DocumentLink) to efficiently encode and transmit small unstructured documents, with [MessagePack](https://msgpack.org/index.html). A document is equivalent to a JSON number/string/array/object or a Python number/string/list/tuple/dict.
- Specify a structured command interface (CommandLink) for issuing and service arbitrary requests and responses between an application client and server over a DocumentLink.

Currently, phyllo does not yet:

- Specify most of the application layer links or framework functionality (NameRouter, ResourceLink, and/or ServiceLink).
- Specify ReliableBufferLink handshaking for reliable setup of reliable data transmission.
- Specify a connection hand-shaking method to negotiate the version and capabilities of the protocol (in ValidatedDatagramLink or above) or the application.
- Provide documentation of protocol specification APIs.
- Specify a way to make unreliable buffer transmission accessible to layers above ReliableBufferLink.
- Specify a way to negotiate the serialization scheme for ObjectLink, to allow for JSON serialization instead of MessagePack serialization, and to allow fast and lightweight transmission of structured data following a schema.

Phyllo does not yet (and will not, unless you'd like to make a feature request and start a discussion):

- Support efficient out-of-order delivery of reliable buffers (such as with [Selective Repeat ARQ](https://en.wikipedia.org/wiki/Selective_Repeat_ARQ)).
- Provide a way to send buffers longer than 246 bytes over a ReliableBufferLink (i.e. with segmentation).

Phyllo does not (and will not, unless you'd like to develop a specification and/or extension):

- Specify a way to extend the functionality available in the data transport layers.
- Specify a way (such as with selective acknowledgements) to reduce the amount of data sent in reliable buffer layer retransmissions needed for reliable transmission.
- Specify a means of direct communication between more than two peers on a bus or network. The roadmap includes a plan to specify a DistributedServiceLink which acts as a middle-man to delegate servicing of client requests to another connected device. If you want direct communication between more than two peers at a lower level, consider using PJON (see the Related Projects section), either standalone or combined with phyllo.
- Provide an adapter to combine PJON with phyllo.
- Specify a way to encrypt data or authenticate peer identity for transport security and privacy. This would be compute-intensive with a significant performance cost and is not needed for the projects currently supporting phyllo's development. If you need security features, you'll need to do the specification and implementation.
- Support full protocol capabilities on low-memory microcontrollers such as the Arduino Micro - to deploy the full protocol stack, message payload sizes must be drastically reduced (to ~32-64 bytes).
- Specify a way to transport datagrams and reliable buffers in a human-readable format for regular serial monitors. While this is technically possible, it adds complexity and is not needed for the projects currently supporting phyllo's development.


## Related Projects

### Transport

The following libraries specify network protocols to enable reliable data exchange on embedded devices:

- [PJON](https://www.pjon.org) ([Github](https://github.com/gioblu/PJON)) is a protocol designed for network communication between two or more hosts, potentially on a network, over *any of a variety of physical media and transports*. PJON specifies a dynamic header for different functionalities; this dynamic extensibility is important for interoperability on multi-host networks but adds overhead unnecessary for a point-to-point communication protocol between two hosts. PJON's [acknowledgement specification](https://github.com/gioblu/PJON/blob/master/specification/PJON-protocol-acknowledge-specification-v1.0.md) specifies [stop-and-wait ARQ](https://en.wikipedia.org/wiki/Stop-and-wait_ARQ) for reliable data transmission on physical media with multiple hosts, but it does not provide [general sliding window ARQ](https://en.wikipedia.org/wiki/Sliding_window_protocol) which can attain much higher performance on a point-to-point link between two hosts. These aspects of the protocol design make it more suitable for communication over networks but less suitable for high-data-rate performance on a point-to-point link.
- [RadioHead](https://www.airspayce.com/mikem/arduino/RadioHead/) species a network protocol to provide to enable communication between two or more devices, potentially on a network, over *any of a variety of data radios* and other transports such as serial.
- There are too many libraries to list which enable reliable data exchange via TCP/IP, with all the added complexities and hardware capabilities required for these protocols.

There are too many libraries to list which enable data exchange on embedded devices without guaranteed reliability of data exchange.

### Application

The following libraries also specify application frameworks based on message exchange on embedded devices:

- [ModuleInterface](https://github.com/fredilarsen/ModuleInterface) provides a framework to synchronize variable values over a PJON connection.
- [Telemetry](https://github.com/Overdrivr/Telemetry/wiki) provides a pub-sub messaging framework over serial or Bluetooth to send simple messages from computers or embedded devices or to send updated values to variables on embedded devices. However, the project is currently inactive, and there does not appear to be an easy way to guarantee reliable transmission over the communication link.
- [Arduino-SerialCommand](https://github.com/kroimon/Arduino-SerialCommand) and its forks are very simple libraries which allow registering callback functions to string commands.
