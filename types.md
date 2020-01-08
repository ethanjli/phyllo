# Types

Each protocol has a one-byte numerical type code associated with it. When a type code is given with a data unit payload received by a peer, the type code specifies which protocol should be used to handle the payload.


## Layer

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


## Bytes

`0x10` - `0x1f` are reserved for byte or byte buffer payloads representing generic bytes.

| Protocol Type  | Type Code | Description                         |
| -------------- | --------- | ----------------------------------- |
| `bytes/buffer` | `0x10`    | Byte buffer of arbitrary bytes      |
| `bytes/stream` | `0x11`    | Byte stream of arbitrary bytes      |
| `bytes/chunk`  | `0x12`    | Byte stream chunk of non-null bytes |

`0x13` - `0x1f` are reserved for future specification by phyllo.


## Transport

`0x20` - `0x2f` are reserved for byte buffer payloads representing phyllo-specified transport-level data units:

| Protocol Type                | Type Code | Description        |
| ---------------------------- | --------- | ------------------ |
| `transport/frame`            | `0x20`    | COBS-encoded Frame |
| `transport/datagram`         | `0x21`    | Datagram           |
| `transport/validatedDatagram`| `0x22`    | Validated Datagram |
| `transport/reliableBuffer`   | `0x23`    | Reliable Buffer    |
| `transport/portedBuffer`     | `0x24`    | Ported Buffer      |

`0x25` - `0x2f` are reserved for future specification by phyllo.

`0x30` - `0x3f` are allocated for byte buffer payloads representing transport-level data units specified by programmers on an ad hoc basis.


## Presentation

`0x40` - `0x4f` are reserved for byte buffer payloads representing phyllo-specified presentation-level serialized documents:

| Protocol Type          | Type Code | Description          |
| ---------------------- | --------- | -------------------- |
| `presentation/msgpack` | `0x40`    | MessagePack document |
| `presentation/cbor`    | `0x41`    | CBOR data item       |
| `presentation/json`    | `0x42`    | JSON document        |

`0x43` - `0x4f` are reserved for future specification by phyllo.

`0x50` - `0x5f` are allocated for byte buffer payloads representing presentation-level serialized documents specified by programmers/applications on an ad hoc basis.


## Application

`0x60` - `0x61` are reserved for documents representing phyllo-specified application-level messages:

| Protocol Type         | Type Code | Description                           |
| --------------------- | --------- | ------------------------------------- |
| `application/generic` | `0x60`    | Generic (unspecified) application     |
| `application/pubsub`  | `0x61`    | Generic pub-sub messaging application |
| `application/rpc`     | `0x62`    | Generic RPC application               |
| `application/rest`    | `0x63`    | Generic RESTful application           |

`0x64` - `0x6f` are reserved for future specification by phyllo.

`0x70` - `0x7f` are allocated for documents representing application-level documents specified by programmers/applications on an ad hoc basis.

`0x80` - `0xff` are reserved for future specification/allocation by phyllo.
