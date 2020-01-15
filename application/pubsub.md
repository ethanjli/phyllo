# Publish-Subscribe Messaging

The phyllo __pub-sub messaging framework__ enables exchange of byte buffers as payloads over independent named logical channels (called _topics_) multiplexed over a the underlying link by the [publish-subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern). Each data unit combining a topic and a payload is called a __pub-sub message__. Various components of an application can exchange data over application-specified topics with an application running on the other peer, even without specific knowledge about the other application. This allows applications to specify an interface for independent channels of communication by message passing.


## Basic Description

- Framework name: Pub-Sub
- Purpose: Publish-subscribe pattern of data exchange.


## Pub-Sub Message Link

The __pub-sub message link__ handles representation of messages.

### Basic Description

- Data unit name: Pub-Sub Message
- Payload type: byte buffer
- Layer name: Pub-Sub Message Link
- Protocol type: `application/pubsub`
- Protocol type code: `0x60`
- Purpose: Multiplexing of application data over named-topic logical channels.
- Payload demultiplexing keys: topic
- Length overhead: between 3 bytes and 18 bytes, depending on the length of the topic name
- Payload maximum length: determined by payload length limits of layers underneath and length of topic name
- Services required from below:
    - Byte buffer exchange
- Services provided for above:
    - Byte buffer exchange
    - Payload named-topic multiplexing

### Service Interface

Named-topic structured data interface:

- __send__ a byte buffer with a specified topic to the other peer.
- __has receive__ to check whether a message has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received message's payload and topic from the other peer.

### Peer Interface

Each byte buffer data exchanged due to the service interface is together with a topic:

| __Name__     | Type                       | Topic Length       | Topic               | Payload      |
| ------------ | -------------------------- | ------------------ | ------------------- | ------------ |
| __Length__   | 1                          | 1                  | ...                 | ...          |
| __Contents__ | Payload Protocol Type Code | Topic Field Length | Topic Byte Sequence | Buffer bytes |

If the type field is an 8-bit unsigned integer between `0x00` and `0x0f`, then the message is actually a layer protocol message used internally by the pub-sub messaging framework. Messages with such topics do not contain application data and thus should not be exposed by the service interface to higher layers as normal data.

The topic and payload fields are of variable length; the topic length field specifies the boundary between the two fields and is represented as the lower 4 bits of a single byte (1 - 18). The topic length must be between 1 and 16 - the topic may not be empty. The upper 4 bits are reserved for future use.

Normally, the topic field specifies the topic associated with the payload and is represented as a sequence of raw bytes. For communication between applications running on computers, it is recommended that the bytes be strings encoded by utf-8 into bytes. Topics may be specified by the application either as a monolithic string (e.g. `"echo"` or `"ping"`) or as a hierarchical name path where each byte is a name in the namespace specified by the previous byte (e.g. `(0x00)`, `(0x00, 0x00)`, `(0x00, 0x01)`, `(0x01)`, `(0x01, 0x00)`, `(0x01, 0x01)`, `(0x01, 0x02)`, `(0x01, 0x03)`, `(0x02)`); the way topics are used is determined by the application or application framework layered over the pub-sub messaging framework.


#### Sending
When a peer sends a topic, payload type, and payload byte buffer, the framework will put them into a message and send the serialized representation of that document.

#### Receiving
When a peer receives a message, it will expose the topic, payload type, and payload byte buffer.

#### Example


## Router

The __message router__ provides an event-driven observer interface on top of a message link.

### Service Interface

Named-topic observer interface:

- __subscribe__ an observer callback to handle any byte buffers (and their associated topics) on all topics.
- __subscribe__ an observer callback to handle any byte buffers (and their associated topics) on the specified topic.
- __subscribe__ an observer callback to handle any byte buffers (and their associated topics) on all topics with the specified prefix.
