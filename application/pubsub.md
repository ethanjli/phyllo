# Publish-Subscribe Messaging

The phyllo __pub-sub messaging framework__ enables exchange of byte buffers or structured data as payloads over independent named logical channels (called _topics_) multiplexed over the underlying link by the [publish-subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern). Each data unit combining a topic and a payload is called a __pub-sub message__. Various components of an application can exchange data over application-specified topics with an application running on the other peer, even without specific knowledge about the other application. This allows applications to specify an interface for independent channels of communication by message passing.


## Basic Description

- Framework name: Pub-Sub
- Purpose: Publish-subscribe pattern of data exchange.


## Pub-Sub Message Link

The __pub-sub message link__ handles multiplexing of byte buffer payloads over logically independent channels named by topics.

### Basic Description

- Data unit name: Pub-Sub Message
- Payload type: byte buffer
- Layer name: Pub-Sub Message Link
- Protocol type: `application/pubsub`
- Protocol type code: `0x60`
- Purpose: Multiplexing of application data over named-topic logical channels.
- Payload demultiplexing keys: topic
- Length overhead: between 2 bytes and 18 bytes, depending on the length of the topic name
- Payload maximum length: determined by payload length limits of layers underneath and length of topic name
- Services required from below:
    - Byte buffer exchange
- Services provided for above:
    - Byte buffer exchange
    - Payload named-topic multiplexing

### Service Interface

Named-topic byte buffer interface:

- __send__ a byte buffer with a specified topic to the other peer.
- __has receive__ to check whether a message has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received message's payload and topic from the other peer.

### Peer Interface

Each byte buffer data exchanged due to the service interface is exchanged together with a type code, a topic, and a topic length:

| __Name__     | Type                       | Topic Length       | Topic               | Payload      |
| ------------ | -------------------------- | ------------------ | ------------------- | ------------ |
| __Length__   | 1                          | 1                  | ...                 | ...          |
| __Contents__ | Payload Protocol Type Code | Topic Field Length | Topic Byte Sequence | Buffer bytes |

If the type field is an 8-bit unsigned integer between `0x00` and `0x0f`, then the message is actually a layer protocol message used internally by the pub-sub messaging framework. Messages with such topics do not contain application data and thus should not be exposed by the service interface to higher layers as normal data.

The topic and payload fields are of variable length; the topic length field specifies the boundary between the two fields and is represented as the lower 4 bits of a single byte (0 - 15). The topic length must be between 0 and 15 - the topic may be empty. The upper 4 bits of the topic field are reserved for future use.

Normally, the topic field specifies the topic associated with the payload and is represented as a sequence of raw bytes. For communication between applications running on computers (where conversion to and from JSON representations may be needed), it is recommended that the bytes be strings encoded by utf-8 into bytes. Topics may be specified by the application either as a monolithic string (e.g. `"echo"` or `"ping"`) or as a hierarchical name path where each byte is a name in the namespace specified by the previous byte (e.g. `(0x00)`, `(0x00, 0x00)`, `(0x00, 0x01)`, `(0x01)`, `(0x01, 0x00)`, `(0x01, 0x01)`, `(0x01, 0x02)`, `(0x01, 0x03)`, `(0x02)`); the way topics are used is determined by the application layered over the pub-sub messaging framework.


#### Sending
When a peer sends a topic, payload type, and payload byte buffer, the framework will put them into a message and send the serialized representation of that document.

#### Receiving
When a peer receives a message, it will expose the topic, payload type, and payload byte buffer.

#### Example
A generic byte buffer payload of `0x01 0x02 0x03 0x04 0x00` will be sent with topic `0x30 0x31` as the following byte buffer (as a generic byte buffer of type `bytes/buffer` has type code `0x10` and the topic has 2 bytes):

| __Type__     | Type   | Topic Length | Topic       | Byte   | Byte   | Byte   | Byte   | Byte   |
| ------------ | ------ | ------------ | ----------- | ------ | ------ | ------ | ------ | ------ |
| __Length__   | 1      | 1            | 2           | 1      | 1      | 1      | 1      | 1      |
| __Contents__ | `0x10` | `0x02`       | `0x30 0x31` | `0x01` | `0x02` | `0x03` | `0x04` | `0x00` |

The receiver of this byte buffer body will receive the same generic byte buffer payload `0x01 0x02 0x03 0x04 0x00` and topic `0x30 0x31`.


## Pub-Sub Document Link

The __pub-sub document link__ handles multiplexing of structured data payloads over logically independent channels named by topics. It is just a Document Link, but with a service interface exposing the topic from the underlying pub-sub message link.

### Basic Description

- Data unit name: Document
- Payload type: application data (serialized body or specific data or generic document tree)
- Layer name: Pub-Sub Document Link
- Protocol type: `presentation/document`
- Protocol type code: `0x40`
- Purpose: Multiplexing of arbitrary structued data over named-topic logical channels.
- Payload demultiplexing keys: topic
- Length overhead: 2 bytes plus any overhead from the serialization format
- Payload maximum length: determined by payload length limits of layers underneath and by the serialization format (and possibly the serialization schema)
- Services required from below:
    - Byte buffer exchange
    - Payload named-topic multiplexing
- Services provided for above:
    - Structured data exchange
    - Payload named-topic multiplexing

### Service Interface

Named-topic structured data interface:

- __send__ a byte buffer representing serialized structured data with a specified serialization format and schema and topic to the other peer.
- __send__ a unit of structured data with a specified format and schema (or, for implementations which support it, a specified schema type) and topic to the other peer.
- __send__ a document tree containing structured data with a specified format and schema (or, for implementations which support it, a specified schema type) and topic to the other peer, for serialization format implementations which support it.
- __has receive__ to check whether a unit of structured data and its topic has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received unit of structured data and its topic from the other peer. In some implementations, the serialized data may be immediately deserialized into a document tree or a structured representation, while in other implementations deserialization may be deferred until required by the application through an interface provided with the data.

### Peer Interface

The peer interface is exactly the same as that of the presentation document link. The topic is transparently passed to/from the underlying pub-sub message link.


## Endpoint

An __endpoint__ defines a filter which sits on top of a pub-sub document link and passes documents which match a topic name filter.

### Service Interface

Endpoint interface:

- __has receive__ to check whether a pub-sub document has been received at the endpoint from the other peer with a matching topic name and is ready to be consumed.
- __receive__ to consume the received pub-sub document.
- __send__ a document on the topic name associated with the endpoint.


## Endpoint Handler

The __endpoint handler__ interface defines a pub-sub document interface which sits on top of a pub-sub document link and can send and receive documents on certain endpoints.

### Service Interface

Endpoint handler interface:

- __receive__ to handle a received pub-sub document arbitarily.


## Single Endpoint Handler

The __single-endpoint handler__ inteface defines a Endpoint Handler event-driven observer interface which sits on top of a pub-sub document link and handles all documents on exactly one Endpoint.

### Service Interface

Single-endpoint handler interface:

- __on_receive__ to handle a document received on the associated endpoint.
- __send__ a document on the associated endpoint.


## Router

The __message router__ provides an event-driven observer interface on top of a pub-sub document link.

### Service Interface

Named-topic observer interface:

- __subscribe__ an Endpoint Handler to receive pub-sub documents.
