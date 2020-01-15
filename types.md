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


### Reserved

`0x60` - `0xff` are reserved for future specification/allocation by phyllo.


## Document Serialization Formats

Each document serialization format has an associated serialization format code. When a format code is given with a document body received by a peer, the format code specifies which format should be used to deserialize the body into a payload.

### Binary

`0x00` - `0x3f` are reserved for binary formats. Binary formats store data for compact transmission.

#### Dynamically Typed
`0x00` - `0x0f` are reserved for phyllo-specified binary dynamically-typed formats:

| Protocol Type            | Format Code | Name        |
| ------------------------ | ----------- | ----------- |
| `binary/dynamic/msgpack` | `0x00`      | MessagePack |
| `binary/dynamic/cbor`    | `0x01`      | CBOR        |
| `binary/dynamic/bson`    | `0x02`      | BSON        |
| `binary/dynamic/avro`    | `0x03`      | Avro        |

`0x04` - `0x0f` are reserved for future specification by phyllo.

`0x10` - `0x1f` are allocated for binary dynamically-typed serialization formats specified by programmers/applications on an ad hoc basis.

#### Statically Typed
`0x20` - `0x2f` are reserved for phyllo-specified binary statically typed formats:

| Protocol Type               | Format Code | Name             | ASCII Type Code Equivalent |
| --------------------------- | ----------- | ---------------- | -------------------------- |
| `binary/static/protobuf`    | `0x20`      | Protocol Buffers | (space)                    |
| `binary/static/thrift`      | `0x21`      | Thrift           | `!`                        |
| `binary/static/flatbuffers` | `0x22`      | FlatBuffers      | `"`                        |
| `binary/static/capnproto`   | `0x23`      | Cap'n Proto      | `#`                        |

`0x24` - `0x2f` are reserved for future specification by phyllo.

`0x30` - `0x3f` are allocated for binary schema-based serialization formats specified by programmers/applications on an ad hoc basis.

### Text

`0x40` - `0x4f` are reserved for phyllo-specified text-based formats:

| Protocol Type | Format Code | Name | ASCII Type Code Equivalent |
| ------------- | ----------- | ---- | -------------------------- |
| `text/json`   | `0x40`      | JSON | `@`                        |
| `text/csv`    | `0x41`      | CSV  | `A`                        |

`0x42` - `0x4f` are reserved for future specification by phyllo.

`0x50` - `0x5f` are allocated for text-based serialization formats specified by programmers/applications on an ad hoc basis.

### Reserved

`0x60` - `0xff` are reserved for future specification/allocation by phyllo.


## Document Schemas

Each document may have an associated schema code. When a schema code is given with a document body received by a peer, it specifies which schema should be used to parse the payload into a data structure.

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

### Generic

Generic schemas are common reusable types corresponding to singleton data.

| Schema Type          | Schema Code | Description |
| -------------------- | ----------- | ----------- |
| `generic/schemaless` | `0x10`      | No schema   |

#### Primitive
`0x01` - `0x0f` are reserved for phyllo-specified generic types of singleton data:

| Schema Type                 | Schema Code | Description         |
| --------------------------- | ----------- | ------------------- |
| `generic/primitive/none`    | `0x11`      | Nil/null/none       |
| `generic/primitive/boolean` | `0x12`      | boolean             |
| `generic/primitive/uint`    | `0x13`      | uint8 number        |
| `generic/primitive/uint8`   | `0x14`      | uint8 number        |
| `generic/primitive/uint16`  | `0x15`      | uint16 number       |
| `generic/primitive/uint32`  | `0x16`      | uint32 number       |
| `generic/primitive/uint64`  | `0x17`      | uint64 number       |
| `generic/primitive/int`     | `0x18`      | uint8 number        |
| `generic/primitive/int8`    | `0x19`      | int8 number         |
| `generic/primitive/int16`   | `0x1a`      | int16 number        |
| `generic/primitive/int32`   | `0x1b`      | int32 number        |
| `generic/primitive/int64`   | `0x1c`      | int64 number        |
| `generic/primitive/float32` | `0x1d`      | 32-bit float number |
| `generic/primitive/float64` | `0x1e`      | 64-bit float number |

`0x1f` is reserved for future specification by phyllo.

#### Sequence
`0x20` - `0x2f` are reserved for phyllo-specified generic types of sequence data:

| Schema Type                 | Schema Code | Description                    |
| --------------------------- | ----------- | ------------------------------ |
| `generic/sequence/string`   | `0x20`      | utf-8 string, arbitrary length |
| `generic/sequence/string8`  | `0x21`      | utf-8 string, up to 8 chars    |
| `generic/sequence/string16` | `0x22`      | utf-8 string, up to 16 chars   |
| `generic/sequence/string32` | `0x23`      | utf-8 string, up to 32 chars   |
| `generic/sequence/string64` | `0x24`      | utf-8 string, up to 64 chars   |
| `generic/sequence/binary`   | `0x25`      | byte array, arbitrary length   |
| `generic/sequence/binary8`  | `0x26`      | byte array, up to 8 bytes      |
| `generic/sequence/binary16` | `0x27`      | byte array, up to 16 bytes     |
| `generic/sequence/binary32` | `0x28`      | byte array, up to 32 bytes     |
| `generic/sequence/binary64` | `0x29`      | byte array, up to 64 bytes     |

`0x2a` - `0x2f` are reserved for future specification by phyllo.

#### Other
`0x30` - `0x3f` are allocated for generic types specified by programmers/applications on an ad hoc basis.

### Application Frameworks

`0x40` - `0x6f` are reserved for application framework messages. Such messages are used internally by application frameworks.

#### Pub-Sub Messaging
`0x40` - `0x4f` are reserved for phyllo-specified schemas of pub-sub application framework messages:

| Schema Type                | Schema Code | Description             | ASCII Type Code Equivalent |
| -------------------------- | ----------- | ----------------------- | -------------------------- |
| `framework/pubsub/generic` | `0x40`      | Generic pub-sub message | `@`                        |
| `framework/pubsub/phyllo`  | `0x41`      | Phyllo pub-sub message  | `A`                        |

`0x42` - `0x4f` are reserved for future specification by phyllo.

#### RPC
| Schema Type             | Schema Code | Description             | ASCII Type Code Equivalent |
| ----------------------- | ----------- | ----------------------- | -------------------------- |
| `framework/rpc/generic` | `0x50`      | Generic RPC message     | `P`                        |
| `framework/rpc/phyllo`  | `0x51`      | Phyllo RPC message      | `Q`                        |
| `framework/rpc/msgpack` | `0x52`      | MessagePack-RPC message | `R`                        |

`0x53` - `0x57` are reserved for future specification by phyllo.

#### REST
| Schema Type              | Schema Code | Description             | ASCII Type Code Equivalent |
| ------------------------ | ----------- | ----------------------- | -------------------------- |
| `framework/rest/generic` | `0x58`      | Generic RESTful message | `X`                        |
| `framework/rest/phyllo`  | `0x59`      | Phyllo RESTful message  | `Y`                        |
| `framework/rest/coap`    | `0x5a`      | CoAP RESTful message    | `Z`                        |

`0x5b` - `0x5f` are reserved for future specification by phyllo.

#### Other
`0x60` - `0x6f` are reserved for future specification by phyllo.

### Applications

`0x70` - `0xff` are allocated for custom application schemas specified by programmers/applications on an ad hoc basis. The following convention is recommended for embedded systems but does not have to be strictly followed:

| Schema Type                  | Schema Code Range | Description                      |
| ---------------------------- | ----------------- | -------------------------------- |
| `application/generic/tuples` | `0x70` - `0x7f`   | Generic (reusable) tuples        |
| `application/generic/arrays` | `0x80` - `0x8f`   | Generic (reusable) arrays        |
| `application/generic/maps`   | `0x90` - `0x9f`   | Generic (reusable) maps          |
| `application/debug`          | `0xa0` - `0xaf`   | Debugging-related functionality  |
| `application/system`         | `0xb0` - `0xbf`   | System management functionality  |
| `application/services`       | `0xc0` - `0xcf`   | Service management functionality |
| `application/computation`    | `0xd0` - `0xdf`   | Intensive computation            |
| `application/devices`        | `0xe0` - `0xef`   | Device/hardware control          |
| `application/miscellaneous`  | `0xf0` - `0xff`   | Miscellaneous functionality      |
