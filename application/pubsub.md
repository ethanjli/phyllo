# Publish-Subscribe Messaging

The phyllo __pub-sub messaging framework__ enables exchange of documents as payloads over independent named logical channels (called _topics_) multiplexed over a link providing structured data exchange, such as a document link, by the [publish-subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern). Each document combining topic and payload document is called a __message__. Various components of an application can exchange documents over application-specified topics with an application running on the other peer, even without specific knowledge about the other application. This allows applications to specify an interface for independent channels of communication by document exchange.


## Basic Description

- Framework name: Pub-Sub
- Purpose: Publish-subscribe pattern of document exchange.


## Message Link

The __message link__ handles representation of messages in documents.

### Basic Description

- Data unit name: Message
- Data unit type: document
- Payload type: application data (serialized body or specific data structure or generic document tree)
- Document serialization format name: 'binary/dynamic/msgpack'
- Document serialization format code: `0x00`
- Document schema type: `framework/pubsub/phyllo`
- Document schema code: `0x41`
- Purpose: Representation of application data in named-topic channels.
- Payload demultiplexing keys: topic
- Services required from below:
    - Structured data exchange
- Services provided for above:
    - Structured data exchange
    - Payload name-based multiplexing

Messages are serialized on embedded devices in the MessagePack format and on computers in the JSON format.

### Service Interface

Named-topic structured data interface:

- __send__ a buffer representing serialized structured data with a specified schema and topic to the other peer.
- __send__ a unit of structured data with a specified schema and topic to the other peer.
- __send__ a document tree containing structured data with a specified schema and topic to the other peer, for serialization format implementations which support it.
- __has receive__ to check whether a unit of structured data has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received unit of structured data and its topic from the other peer. In some implementations, the serialized data may be immediately deserialized into a document tree or a structured representation. In some implementations, deserialization may be deferred until required by the application through an interface provided with the data.

### Peer Interface

Each unit of structured data exchanged due to the service interface is exchanged as the serialized payload of a message, together with a topic:

- Message document: two-element array
    - Topic: byte sequence represented as raw bytes, length up to 16 bytes; OR an 8-bit unsigned integer (`uint8_t`) between `0x00` and `0x0f`.
    - Payload: serialized document of application data represented as raw bytes, arbitrary length (limited by payload size limit of lower layers).

Normally, the topic field specifies the topic associated with the payload and is represented as a sequence of raw bytes. For communication between applications running on computers, it is recommended that the bytes be strings encoded by utf-8 into bytes. Topics may be specified by the application either as a monolithic string (e.g. `"echo"` or `"ping"`) or as a hierarchical name path where each byte is a name in the namespace specified by the previous byte (e.g. `(0x00)`, `(0x00, 0x00)`, `(0x00, 0x01)`, `(0x01)`, `(0x01, 0x00)`, `(0x01, 0x01)`, `(0x01, 0x02)`, `(0x01, 0x03)`, `(0x02)`); the way topics are used is determined by the application or application framework layered over the pub-sub messaging framework.

If the topic of the payload is an 8-bit unsigned integer between `0x00` and `0x0f` instead of a byte sequence, then the message is actually a layer protocol message used internally by the pub-sub messaging framework. Messages with such topics do not contain application data and thus should not be exposed by the service interface to higher layers as normal data; then the structure of the body's internal layer data is implicitly defined by the pub-sub messaging framework.

#### Sending
When a peer sends a topic and an application document, the framework will just serialize the application document, put it into a message consisting of the topic and the byte buffer, and send the serialized representation of that document.

#### Receiving
When a peer receives a message, it will expose the topic and the payload application document.

#### Example
A unit of application data `"abcd"` with schema `0x21` has a serialized MessagePack document representation of `(0xa4, 0x61, 0x62, 0x63, 0x64)`, so it will be sent on the topic `(0x00, 0xff)` as the following message:

- Message:
    - Topic: raw byte sequence of length 2, body `(0x00, 0xff)`
    - Payload: raw byte sequence of length 6, body `(0x21, 0xa4, 0x61, 0x62, 0x63, 0x64)`

A unit of application data `"abcd"` with schema `0x21` has a serialized MessagePack document representation of `(0xa4, 0x61, 0x62, 0x63, 0x64)`, so it will be sent on the topic `"hello"` (which has a utf-8 encoded representation of `(0x68, 0x65, 0x6c, 0x6c, 0x6f)`) as the following message:

- Message:
    - Topic: raw byte sequence of length 5, body `(0x68, 0x65, 0x6c, 0x6c, 0x6f)`
    - Payload: raw byte sequence of length 6, body `(0x21, 0xa4, 0x61, 0x62, 0x63, 0x64)`


## Router

The __message router__ provides an event-driven observer interface on top of a message link.

### Service Interface

Named-topic observer interface:

- __subscribe__ an observer callback to handle any documents (and their associated topics) on all topics.
- __subscribe__ an observer callback to handle any documents (and their associated topics) on the specified topic.
- __subscribe__ an observer callback to handle any documents (and their associated topics) on all topics with the specified prefix.
