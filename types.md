# Types

Data types are specified by a one-byte numerical type code associated with the data.


## Protocol Types

Each protocol has a one-byte numerical type code associated with it. When a type code is given with a data unit payload received by a peer, the type code specifies which protocol should be used to handle the payload.

### Layer

`0x00` - `0x0f` are reserved for data units and payloads to be used internally by the layer which transmitted the data unit. Payloads of these types should not be exposed by the service interface to higher layers as normal data.

| Protocol Type        | Type Code | Description                     |
| -------------------- | --------- | ------------------------------- |
| `layer/control`      | `0x00`    | Control signal                  |
| `layer/version`      | `0x01`    | Version negotiation signal      |
| `layer/capabilities` | `0x02`    | Capabilities negotiation signal |
| `layer/error`        | `0x03`    | Error signal                    |
| `layer/warn`         | `0x04`    | Warning signal                  |
| `layer/info`         | `0x05`    | Info signal                     |
| `layer/debug`        | `0x06`    | Debugging signal                |
| `layer/trace`        | `0x07`    | Tracing signal                  |
| `layer/metrics`      | `0x08`    | Metrics signal                  |

`0x09` - `0x0f` are reserved for definition by the layer which transmitted the data unit.

### Bytes

`0x10` - `0x1f` are reserved for byte or byte buffer payloads representing generic bytes.

| Protocol Type  | Type Code | Description                         |
| -------------- | --------- | ----------------------------------- |
| `bytes/buffer` | `0x10`    | Byte buffer of arbitrary bytes      |
| `bytes/stream` | `0x11`    | Byte stream of arbitrary bytes      |
| `bytes/chunk`  | `0x12`    | Byte stream chunk of non-null bytes |

`0x13` - `0x1f` are reserved for future specification by phyllo.

### Transport

`0x20` - `0x2f` are reserved for byte buffer payloads representing phyllo-specified transport-level data units:

| Protocol Type                 | Type Code | Description        | ASCII Type Code Equivalent |
| ----------------------------- | --------- | ------------------ | -------------------------- |
| `transport/frame`             | `0x20`    | COBS-encoded Frame | (space)                    |
| `transport/datagram`          | `0x21`    | Datagram           | `!`                        |
| `transport/validatedDatagram` | `0x22`    | Validated Datagram | `"`                        |
| `transport/reliableBuffer`    | `0x23`    | Reliable Buffer    | `#`                        |
| `transport/portedBuffer`      | `0x24`    | Ported Buffer      | `$`                        |

`0x25` - `0x2f` are reserved for future specification by phyllo.

`0x30` - `0x3f` are allocated for byte buffer payloads representing transport-level data units specified by programmers on an ad hoc basis.

### Presentation

`0x40` - `0x4f` are reserved for byte buffer payloads representing phyllo-specified presentation-level serialized documents:

| Protocol Type           | Type Code | Description                                      | ASCII Type Code Equivalent |
| ----------------------- | --------- | -------------------------------------------------| -------------------------- |
| `presentation/document` | `0x40`    | Generic (format unspecified) serialized document | `@`                        |

`0x41` - `0x4f` are reserved for future specification by phyllo.

`0x50` - `0x5f` are allocated for presentation-level messages specified by programmers/applications on an ad hoc basis.

### Application Frameworks

`0x60` - `0x6f` are reserved for byte buffer payloads representing phyllo-specified application framework data units:

| Protocol Type        | Type Code | Description              | ASCII Type Code Equivalent |
| -------------------- | --------- | ------------------------ | -------------------------- |
| `application/pubsub` | `0x60`    | Pub-sub message          | `` ` ``                    |
| `application/rpc`    | `0x61`    | RPC transaction message  | `a`                        |
| `application/rest`   | `0x62`    | REST transaction message | `b`                        |

`0x63` - `0x6f` are reserved for future specification by phyllo.

### Reserved

`0x70` - `0xff` are reserved for future specification/allocation by phyllo.


## Document Serialization Formats

Each document serialization format has an associated serialization format code. When a format code is given with a document body received by a peer, the format code specifies which format should be used to deserialize the body into a payload.

### Layer

`0x00` - `0x0f` are reserved for messages and payloads to be used internally by the document layer. Documents of these formats do not contain application data and thus should not be exposed by the service interface to higher layers as normal data.

| Protocol Type        | Format Code | Description                     |
| -------------------- | ----------- | ------------------------------- |
| `layer/control`      | `0x00`      | Control signal                  |
| `layer/version`      | `0x01`      | Version negotiation signal      |
| `layer/capabilities` | `0x02`      | Capabilities negotiation signal |
| `layer/error`        | `0x03`      | Error signal                    |
| `layer/warn`         | `0x04`      | Warning signal                  |
| `layer/info`         | `0x05`      | Info signal                     |
| `layer/debug`        | `0x06`      | Debugging signal                |
| `layer/trace`        | `0x07`      | Tracing signal                  |
| `layer/metrics`      | `0x08`      | Metrics signal                  |

`0x09` - `0x0f` are reserved for future specification or allocation by phyllo.

### Binary

`0x00` - `0x3f` are reserved for binary formats. Binary formats store data for compact transmission.

#### Dynamically Typed
`0x00` - `0x0f` are reserved for phyllo-specified binary dynamically-typed formats:

| Protocol Type            | Format Code | Name                       |
| ------------------------ | ----------- | -------------------------- |
| `binary/dynamic/unknown` | `0x10`      | Unknown (undefined format) |
| `binary/dynamic/msgpack` | `0x11`      | MessagePack                |
| `binary/dynamic/cbor`    | `0x12`      | CBOR                       |
| `binary/dynamic/bson`    | `0x13`      | BSON                       |
| `binary/dynamic/avro`    | `0x14`      | Avro                       |

`0x14` - `0x1f` are reserved for future specification by phyllo.

`0x20` - `0x2f` are allocated for binary dynamically-typed serialization formats specified by programmers/applications on an ad hoc basis.

#### Statically Typed
`0x30` - `0x3f` are reserved for phyllo-specified binary statically typed formats:

| Protocol Type               | Format Code | Name             | ASCII Type Code Equivalent |
| --------------------------- | ----------- | ---------------- | -------------------------- |
| `binary/static/protobuf`    | `0x30`      | Protocol Buffers | `0`                        |
| `binary/static/thrift`      | `0x31`      | Thrift           | `1`                        |
| `binary/static/flatbuffers` | `0x32`      | FlatBuffers      | `2`                        |
| `binary/static/capnproto`   | `0x33`      | Cap'n Proto      | `3`                        |

`0x34` - `0x3f` are reserved for future specification by phyllo.

`0x40` - `0x4f` are allocated for binary schema-based serialization formats specified by programmers/applications on an ad hoc basis.

### Text

`0x50` - `0x5f` are reserved for phyllo-specified text-based formats:

| Protocol Type | Format Code | Name | ASCII Type Code Equivalent |
| ------------- | ----------- | ---- | -------------------------- |
| `text/json`   | `0x50`      | JSON | `P`                        |
| `text/csv`    | `0x51`      | CSV  | `Q`                        |

`0x52` - `0x5f` are reserved for future specification by phyllo.

`0x60` - `0x6f` are allocated for text-based serialization formats specified by programmers/applications on an ad hoc basis.

### Reserved

`0x70` - `0xff` are reserved for future specification/allocation by phyllo.


## Document Schemas

Each document may have an associated schema code. When a schema code is given with a document body received by a peer, it specifies which schema should be used to parse the payload into a data structure.

### Generic

Generic schemas are common reusable types corresponding to singleton data.

| Schema Type          | Schema Code | Description |
| -------------------- | ----------- | ----------- |
| `generic/schemaless` | `0x00`      | No schema   |

#### Primitive
`0x01` - `0x0f` are reserved for phyllo-specified generic types of singleton data:

| Schema Type                 | Schema Code | Description         |
| --------------------------- | ----------- | ------------------- |
| `generic/primitive/none`    | `0x01`      | Nil/null/none       |
| `generic/primitive/boolean` | `0x02`      | boolean             |
| `generic/primitive/uint`    | `0x03`      | uint8 number        |
| `generic/primitive/uint8`   | `0x04`      | uint8 number        |
| `generic/primitive/uint16`  | `0x05`      | uint16 number       |
| `generic/primitive/uint32`  | `0x06`      | uint32 number       |
| `generic/primitive/uint64`  | `0x07`      | uint64 number       |
| `generic/primitive/int`     | `0x08`      | uint8 number        |
| `generic/primitive/int8`    | `0x09`      | int8 number         |
| `generic/primitive/int16`   | `0x0a`      | int16 number        |
| `generic/primitive/int32`   | `0x0b`      | int32 number        |
| `generic/primitive/int64`   | `0x0c`      | int64 number        |
| `generic/primitive/float32` | `0x0d`      | 32-bit float number |
| `generic/primitive/float64` | `0x0e`      | 64-bit float number |

`0x0f` is reserved for future specification by phyllo.

#### Sequence
`0x10` - `0x1f` are reserved for phyllo-specified generic types of sequence data:

| Schema Type                 | Schema Code | Description                    |
| --------------------------- | ----------- | ------------------------------ |
| `generic/sequence/string`   | `0x10`      | utf-8 string, arbitrary length |
| `generic/sequence/string8`  | `0x11`      | utf-8 string, up to 8 chars    |
| `generic/sequence/string16` | `0x12`      | utf-8 string, up to 16 chars   |
| `generic/sequence/string32` | `0x13`      | utf-8 string, up to 32 chars   |
| `generic/sequence/string64` | `0x14`      | utf-8 string, up to 64 chars   |
| `generic/sequence/binary`   | `0x15`      | byte array, arbitrary length   |
| `generic/sequence/binary8`  | `0x16`      | byte array, up to 8 bytes      |
| `generic/sequence/binary16` | `0x17`      | byte array, up to 16 bytes     |
| `generic/sequence/binary32` | `0x18`      | byte array, up to 32 bytes     |
| `generic/sequence/binary64` | `0x19`      | byte array, up to 64 bytes     |

`0x1a` - `0x1f` are reserved for future specification by phyllo.

#### Other
`0x20` - `0x2f` are allocated for generic types specified by programmers/applications on an ad hoc basis.

### Application Frameworks

`0x30` - `0x4f` are reserved for schemas specified by phyllo-specified application frameworks. Such schemas are used internally by application frameworks.

### Applications

`0x50` - `0xff` are allocated for custom application schemas specified by programmers/applications on an ad hoc basis. The following convention is recommended for embedded systems but does not have to be strictly followed:

| Schema Type                  | Schema Code Range| Description                      |
| ---------------------------- | ---------------- | -------------------------------- |
| `application/generic/tuples` | `0x50` - `0x5f`  | Generic (reusable) tuples        |
| `application/generic/arrays` | `0x60` - `0x6f`  | Generic (reusable) arrays        |
| `application/generic/maps`   | `0x70` - `0x7f`  | Generic (reusable) maps          |
| `application/debug`          | `0x80` - `0x8f`  | Debugging-related functionality  |
| `application/system`         | `0x90` - `0x9f`  | System management functionality  |
| `application/services`       | `0xa0` - `0xaf`  | Service management functionality |
| `application/computation`    | `0xb0` - `0xbf`  | Intensive computation            |
| `application/hardware`       | `0xc0` - `0xcf`  | Low-level hardware control       |
| `application/devices`        | `0xd0` - `0xdf`  | High-level device control        |
| `application/data`           | `0xe0` - `0xef`  | Data management functionality    |
| `application/miscellaneous`  | `0xf0` - `0xff`  | Miscellaneous functionality      |
