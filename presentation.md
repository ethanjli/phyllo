# Presentation

Data presentation protocol functionality is decomposed into various serialization strategies in a single document layer. The document layer exchanges structured data payloads serialized as blobs of bytes, together with a serialization format identifier and a schema identifier to specify how the body should be deserialized and parsed by the application running over the document layer.


## Document

The __document link__ enables exchange of structured data with arbitrary contents over a link providing byte buffer exchange, by serializing payloads with an application protocol-specified schema. Each document link is initialized to support exactly one serialization format - applications which need to communicate over multiple serialization formats should use multiple document links multiplexed with a Ported Buffer Link.

### Basic Description

- Data unit name: Document
- Payload type: application data (serialized body or document tree)
- Layer name: Document Link
- Protocol type: `presentation/document`
- Protocol type code: `0x40`
- Purpose: Serialization of arbitrary structured data.
- Payload demultiplexing keys: none
- Services required from below:
    - Byte buffer exchange
- Services provided for above:
    - Structured data exchange

### Service Interface

Structured data interface:

- __send__ a serialized body of structured data with a specified schema to the other peer.
- __send__ a unit of structured data with a specified schema (or, for implementations which support it, a specified schema type).
- __send__ a document tree containing structured data with a specified schema (or, for implementations which support it, a specified schema type) to the other peer, for serialization format implementations which support it.
- __has receive__ to check whether a unit of structured data has been received from the other peer and is ready to be consumed.
- __receive__ to consume the received unit of structured data from the other peer. In some implementations, the serialized data may be immediately deserialized into a document tree or a structured representation, while in other implementations deserialization may be deferred until required by the application through an interface provided with the data.

Error reporting:

- When received data cannot be deserialized into a valid unit of data, an error is reported to the layer above.

### Peer Interface

Each byte buffer exchanged due to the service interface is exchanged as a body serialized in the document link's serialization format, together with a schema code specifying the schema for parsing the payload:

| __Name__     | Schema              | Body                          |
| ------------ | ------------------- | ----------------------------- |
| __Length__   | 1                   | ...                           |
| __Contents__ | Payload Schema Code | Serialized payload data bytes |

The schema field specifies which schema should be used to parse the payload into a data structure and is represented as a single byte (`0x00` - `0xff`). If the schema is between `0x00` and `0x0f`, it is actually a layer protocol message used internally by the document layer. Documents of those formats do not contain application data and thus should not be exposed by the service interface to higher layers as normal data; then the schema for the body's internal layer data is implicitly defined by the document layer.

#### Sending
When a peer sends a byte buffer as a body, it will first prepend the appropriate header and then send the resulting byte buffer as a body.

When a peer sends a unit of structured data, it will first serialize the data, prepend the appropriate header, and then send the resulting body.

When a peer sends a document tree containing structured data, it will first serialize the data, prepend the appropriate header, and then send the resulting body.

#### Receiving
When a peer receives a document with a body, either it will first deserialize it and then expose the full payload for consumption in the service interface as a document tree or a unit of structured data, or it will expose the body and provide methods for deserializing the data.

#### Example
A string payload of `"abcd"` will be sent as the following messagepack-serialized document with schema code `0x21` corresponding to schema type `generic/sequence/string8`:

| __Type__     | Schema | Byte   | Byte   | Byte   | Byte   | Byte   |
| ------------ | ------ | ------ | ------ | ------ | ------ | ------ |
| __Length__   | 1      | 1      | 1      | 1      | 1      | 1      |
| __Contents__ | `0x21` | `0xa4` | `0x61` | `0x62` | `0x63` | `0x64` |

The receiver of this document body will receive the same string `"abcd"`.

A tuple payload of `(true, "abcd")` which is specified by some programmer-defined schema `0x70` will be sent as the following messagepack-serialized document:

| __Type__     | Schema | Byte   | Byte   | Byte   | Byte   | Byte   | Byte   | Byte   |
| ------------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| __Length__   | 1      | 1      | 1      | 1      | 1      | 1      | 1      | 1      |
| __Contents__ | `0x70` | `0x92` | `0xc3` | `0xa4` | `0x61` | `0x62` | `0x63` | `0x64` |

The receiver of this document body will receive the same tuple `(true, "abcd")`.


## Serialization Formats

Every document has a body serialized in a particular format. The conventional Document Link serialization format for binary communication (such as with embedded systems) is [MessagePack](https://msgpack.org/), while the conventional serialization format for text-based communication between computers (such as over HTTP) is JSON.

### MessagePack

The particular version of MessagePack used as the standard format for Phyllo communication with embedded systems is the v5/2.0 specification, which distinguishes between strings and binary data. Furthermore, all implementations and applications are expected to obey the following conventions for interoperability:

- Binary data needs to be supported.
- All strings should be UTF-8 encoded.
- Extension types are not allowed.
- Map keys must be unique: no duplicate keys are allowed.
- All keys of any given map should have the same type. Strings can always be used as map keys. In implementations which allow integers (signed or unsigned) as map keys, all keys of any given map should be either pure strings, pure signed integers, or pure unsigned integers. No other types can be used as map keys.
- The order of key-value pairs in maps should not be expected to be stable or preserved.

MessagePack is used as the conventional binary serialization format for applications, rather than a more compact (and perhaps faster) compiled format such as Protocol Buffers or FlatBuffers, so that application programmers do not have to compile schema files into code files.

In the future, application frameworks may use Protocol Buffers for serializing messages within the application framework layer(s), but every framework should expose a MessagePack document link at the top for serializing any application data payloads carried by the framework.
