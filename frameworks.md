# Application Frameworks

Application functionality is structured into layers by various frameworks.


## Pub-Sub Messaging

The phyllo pub-sub messaging framework enables exchange of documents over independent named logical channels (called _topics_) multiplexed over a link providing structured data exchange, such as a document link, by the [publish-subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern). Various components of an application can exchange messages over application-specified topics with an application running on the other peer, even without specific knowledge about the other application. This allows applications to specify an interface for independent channels of communication by document exchange.

### Basic Description

- Data unit name: Message
- Payload type: application data
- Framework name: Pub-Sub
- Document schema type: `framework/pubsub/phyllo`
- Document schema code: `0x41`
- Purpose: Publish-subscribe pattern of message exchange.
- Payload demultiplexing keys: topic
- Services required from below:
    - Structured data exchange
- Services provided for above:
    - Byte buffer exchange
    - Payload topic-based multiplexing

### Service Interface

Named-topic byte buffer interface:

- __send__ a byte buffer with a specified topic to the other peer.
- __has receive__ to check whether a byte buffer has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received byte buffer (and the associated topic) from the other peer.

Event-driven observer interface:

- __subscribe__ an observer callback to catch any byte buffers (and their associated topics) on all topics.
- __subscribe__ an observer callback to catch any byte buffers (and their associated topics) on the specified topic.
- __subscribe__ an observer callback to catch any byte buffers (and their associated topics) on all topics with the specified prefix.

### Peer Interface

Each byte buffer exchanged due to the service interface is exchanged as a sequence of raw bytes in the pub-sub messaging framework's serialization format, together with a topic:

- Message: two-element array
    - Topic: byte sequence represented as raw bytes, length up to 16 bytes; OR an 8-bit unsigned integer (`uint8_t`) between `0x00` and `0x0f`.
    - Payload: byte buffer represented as raw bytes, arbitrary length (limited by payload size limit of lower layers).

Normally, the topic field specifies the topic associated with the payload and is represented as a sequence of raw bytes. For communication between applications running on computers, it is recommended that the bytes be utf-8 encoded strings. Topics may be specified by the application either as a monolithic string (e.g. `"echo"` or `"ping"`) or as a hierarchical name path where each byte is a name in the namespace specified by the previous byte (e.g. `(0x00)`, `(0x00, 0x00)`, `(0x00, 0x01)`, `(0x01)`, `(0x01, 0x00)`, `(0x01, 0x01)`, `(0x01, 0x02)`, `(0x01, 0x03)`, `(0x02)`); the way topics are used is determined by the application or application framework layered over the pub-sub messaging framework.

If the topic of the payload is an 8-bit unsigned integer between `0x00` and `0x0f` instead of a byte sequence, then the message is actually a layer protocol message used internally by the pub-sub messaging framework. Messages with such topics do not contain application data and thus should not be exposed by the service interface to higher layers as normal data; then the structure of the body's internal layer data is implicitly defined by the pub-sub messaging framework.

#### Sending
When a peer sends a topic and a byte buffer, the framework will just send a document consisting of the topic and the byte buffer.

#### Receiving
When a peer receives a message, it will expose the topic and the payload.

#### Example
A byte buffer payload of `(0x01, 0x02, 0x03, 0x04)` will be sent on the topic `(0x00, 0xff)` as the following message:

- Message:
    - Topic: raw byte sequence of length 2, body `(0x00, 0xff)`
    - Payload: raw byte sequence of length 4, body `(0x01, 0x02, 0x03)`

A byte buffer payload of `(0x01, 0x02, 0x03, 0x04)` will be sent on the topic `"hello"` (which has a utf-8 encoded representation of `(0x68, 0x65, 0x6c, 0x6c, 0x6f)`) as the following message:

- Message:
    - Topic: raw byte sequence of length 5, body `(0x68, 0x65, 0x6c, 0x6c, 0x6f)`
    - Payload: raw byte sequence of length 4, body `(0x01, 0x02, 0x03)`
