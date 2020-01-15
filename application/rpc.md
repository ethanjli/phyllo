# Remote Procedure Calls

The phyllo __RPC framework__ enables an application structured as a client/server pair to communicate by request/response transactions over a publish-subscribe messaging framework stack. The _client_ calls a _method_ at an _endpoint_ on a _server_, which in return sends some data to the client.


## Basic Description

- Framework name: RPC
- Purpose: Service interface for client-server communication based on requests and responses.


## RPC Messages

The __RPC message link__ handles representation of RPC request-response transaction messages. An RPC service may control multiple endpoints, each of which may have multiple methods, and an RPC client may call multiple methods at multiple endpoints. Each endpoint is a logical grouping of related methods, and each service is a logical grouping of related endpoints.

### Basic Description

- Data unit name: RPC Message
- Payload type: byte buffer
- Layer name: RPC Message Link
- Protocol type: `application/rpc`
- Protocol type code: `0x61`
- Purpose: Representation of RPC transaction request/response data.
- Payload demultiplexing keys: topic
- Length overhead: at least 3 bytes, depending on the length of the transaction ID and the RPC message options
- Payload maximum length: determined by payload length limits of layers underneath, length of transaction ID, RPC message options
- Services required from below:
    - Byte buffer exchange
- Services provided for above:
    - Byte buffer exchange
    - Request-response transactions with endpoints and methods
    - Streaming requests and responses

### Service Interface


### Peer Interface

Each byte buffer exchanged due to the service interface is exchanged as the payload of a request or a response associated with a transaction, together with an endpoint, a method, and other metadata:

- RPC Message:
    - Type: transaction message type code
    - Method/Status: transaction message method/status code
    - Transaction ID
    - Options

- RPC transaction request:
    - Type: transaction message type code, `0x10` (the request is completed with the message) or `0x2f` (more messages are forthcoming as part of the request)
    - Method: a byte of value between `0x40` and `0x7f`
    - Transaction ID
    - Options

- Transaction Response message: 4-element array
    - Type: transaction message type code, `0x20` (the response is successful and is completed with the message) or `0x21` - `0x30` (an error was encountered and the message is the final one of the response) or `0x3f` (more messages are forthcoming as part of the response)
    - Status: a byte of value between `0x00` and `0x3f`
    - Transaction ID
    - Options

- Transaction Explicit Acknowledgement message:
    - Type: transaction message type code, `0x09`
    - Method/Status: a byte of value between `0x00` and `0x7f`, should match the message being acknowledged
    - Transaction ID: ???, should match the transaction ID of the message being acknowledged

- Transaction Keepalive message:
    - Type: transaction message type code, `0x0a`
    - Method/Status: a byte of value between `0x00` and `0x7f`, should match the original request which the keepalive is associated with
    - Transaction ID: ???, should match the transaction ID of the original request which the keepalive is associated with

- Transaction Cancellation message:
    - Type: transaction message type code, `0x0b`
    - Method/Status: a byte of value between `0x00` and `0x7f`, should match the original request which is being cancelled
    - Transaction ID: ???, should match the transaction ID of the original request which is being cancelled


#### Transaction Message Type Code
Normally, the type field specifies the transaction message type and is a 7-bit unsigned integer (`0x00` - `0x8f`). If the type of the payload is between `0x00` and `0x0f`, then the message is actually a layer protocol message used internally by the RPC framework. Messages with such types do not contain application data and thus should not be exposed by the service interface to higher layers as normal data; then the structure of the body's internal layer data is implicitly defined by the pub-sub messaging framework.

| Message Type              | Type Code | Description                                                                |
| ------------------------- | --------- | -------------------------------------------------------------------------- |
| `layer/control`           | `0x00`    | Control signal                                                             |
| `layer/version`           | `0x01`    | Version negotiation signal                                                 |
| `layer/capabilities`      | `0x02`    | Capabilities negotiation signal                                            |
| `layer/error`             | `0x03`    | Error signal                                                               |
| `layer/warn`              | `0x04`    | Warning signal                                                             |
| `layer/info`              | `0x05`    | Info signal                                                                |
| `layer/debug`             | `0x06`    | Debugging signal                                                           |
| `layer/trace`             | `0x07`    | Tracing signal                                                             |
| `layer/metrics`           | `0x08`    | Metrics signal                                                             |
| `layer/acknowledge`       | `0x09`    | Transaction Acknowledge message                                            |
| `layer/keepalive`         | `0x0a`    | Transaction Keepalive message                                              |
| `layer/cancel`            | `0x0b`    | Transaction Cancellation message                                           |
| `request/complete`        | `0x10`    | Transaction Request partial message                                        |
| `request/partial`         | `0x11`    | Transaction Request message                                                |
| `response/complete`       | `0x20`    | No error, returned on success.                                             |
| `response/partial`        | `0x21`    | Transaction Response partial message                                       |

#### Method/Status Code
The method/status field is used to specify the RPC method to call in the request, or the status of the method in the response.

##### Response Statuses
`0x00` - `0x1f` are reserved for specification of response statuses by phyllo:

| Status                        | Status Code| Description                                                                |
| ----------------------------- | ---------- | -------------------------------------------------------------------------- |
| `status/ok`                   | `0x00`     | No error, returned on success.                                             |
| `status/error/cancelled`      | `0x01`     | Transaction was cancelled, typically by the client.                        |
| `status/error/unknown`        | `0x02`     | Unknown error.                                                             |
| `status/error/payload`        | `0x03`     | Client provided an invalid payload.                                        |
| `status/error/deadline`       | `0x04`     | Deadline expired before the transaction could complete.                    |
| `status/error/missing`        | `0x05`     | Some requested entity was not found.                                       |
| `status/error/exists`         | `0x06`     | The entity that a client attempted to create already exists.               |
| `status/error/permission`     | `0x07`     | Client does not have permission to execute the transaction.                |
| `status/error/exhausted`      | `0x08`     | Some resource has been exhausted.                                          |
| `status/error/precondition`   | `0x09`     | Transaction rejected because system is not in state needed for it.         |
| `status/error/aborted`        | `0x0a`     | Transaction aborted, typically due to concurrency conflict.                |
| `status/error/range`          | `0x0b`     | Transaction was attempteed outside the valid range.                        |
| `status/error/unimplemented`  | `0x0c`     | Transaction is not implemented/supported/enabled by the server.            |
| `status/error/internal`       | `0x0d`     | Some invariant expected by the underlying system has been broken.          |
| `status/error/unavailable`    | `0x0e`     | Service is currently unavailable.                                          |
| `status/error/data`           | `0x0f`     | Unrecoverable data loss or corruption.                                     |
| `status/error/authentication` | `0x10`     | Client does not have valid authentication credentials for the transaction. |

The response type codes between `0x00` and `0x10` correspond exactly to the [status codes defined by gRPC](https://github.com/grpc/grpc/blob/master/doc/statuscodes.md), and the semantics of the gRPC error codes should be the same semantics here.

`0x11` - `0x1f` are reserved for future specification or allocation by phyllo.

`0x20` - `0x3f` are available for response statuses specified by programmers/applications on an ad hoc basis.

##### Request Methods
`0x40` - `0x5f` are reserved for specification of request methods by phyllo.

| Method              | Method Code | Description                                             | REST Equivalent    | Safe | Idempotent |
| ------------------- | ----------- | ------------------------------------------------------- | ------------------ | ---- | ---------- |
| `endpoint/methods`  | `0x40`      | Query the methods supported by the endpoint.            | (none)             | Yes  | Yes        |
| `endpoint/parent`   | `0x41`      | Query the parent endpoints of the endpoint.             | (none)             | Yes  | Yes        |
| `endpoint/children` | `0x42`      | Query the child endpoints associated with the endpoint. | (none)             | Yes  | Yes        |
| `rest/read`         | `0x50`      | Query the data represented by the endpoint.             | GET                | Yes  | Yes        |
| `rest/update`       | `0x51`      | Update the data represented by the endpoint.            | PUT (for updating) | No   | Yes        |
| `rest/create`       | `0x52`      | Create a new data representation at the endpoint.       | PUT (for creating) | No   | Yes        |
| `rest/delete`       | `0x53`      | Delete the data represented by the endpoint.            | DELETE             | No   | Yes        |
| `rest/put`          | `0x54`      | Create or update the data represented by the endpoint.  | PUT                | No   | Yes        |
| `rest/post`         | `0x54`      | Perform some processing of the request at the endpoint. | POST               | No   | No         |

`0x60` - `0x7f` are reserved for request methods specified by programmers/applications on an ad hoc basis.

#### Transaction ID
The transaction ID is a sequential unsigned integer, of length up to 64 bits (0 - 2^64-1), which uniquely identifies the transaction which the message belongs to for the client. The transaction ID is represented as a variable-length MessagePack unsigned integer - that is, it can be represented as a positive fixnum, a uint8, a uint16, a uint32, or a uint32.

### Options
The options field is a MessagePack-serialized document which is exactly one map, where key-value pairs specify options. Each key is an option code represented as a 7-bit number (`0x00` - `0x7f`) and specifies an option.

| Option                    | Option Code| Description                                                               | In Request| In Response|
| ------------------------- | ---------- | ------------------------------------------------------------------------- | --------- | ---------- |
| `payload/type`            | `0x00`     | Type code of the payload, if it exists.                                   | Yes       | Yes        |
| `payload/body`            | `0x01`     | Body of the payload, if it exists.                                        | Yes       | Yes        |
| `endpoint/host`           | `0x10`     | Host of associated endpoint.                                              | Yes       | Yes        |
| `endpoint/port`           | `0x10`     | Host of associated endpoint.                                              | Yes       | Yes        |
| `endpoint/names`          | `0x10`     | Name of associated endpoint.                                              | Yes       | Yes        |
| `request/preferred/type`  | `0x20`     | Preferred type of the payload data in the response.                       | Yes       | No         |
| `request/preferred/format`| `0x21`     | Preferred format of the payload data in the response, if it's a document. | Yes       | No         |
| `request/preferred/schema`| `0x22`     | Preferred schema of the payload data in the response, if it's a document. | Yes       | No         |
| `request/preconditions`   | `0x28`     | Preconditions which must be satisfied for the request to be accepted.     | Yes       | No         |


#### Sending


#### Receiving


#### Example


## RPC Endpoint

An __RPC Endpoint__ handles requests received for a particular method on the server; recall that the method is specified by the topic name of the underlying Pub-Sub Message Link.

### Service Interface
