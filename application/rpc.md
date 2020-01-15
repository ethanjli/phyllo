# Remote Procedure Calls

The phyllo __RPC framework__ enables an application structured as a client/server pair to communicate by request/response transactions over a publish-subscribe messaging framework stack. The _client_ calls a _method_ running on a _server_, which in return sends some data to the client.


## Basic Description

- Framework name: RPC
- Purpose: Service interface for client-server communication based on requests and responses.


## Request/Response Transactions

The __transaction link__ handles representation of RPC request-response transactions in pub-sub messages. A pub-sub message's topic name is the endpoint which specifies the transaction method, while the message's payload specifies other transaction data. An RPC service may control multiple endpoints, and an RPC client may access multiple endpoints. Each service is a logical grouping of related endpoints.

### Basic Description

- Data unit name: Transaction
- Data unit type: Message document
- Payload type: request/response data (serialized body or specific data structure or generic document tree)
- Document serialization format name: 'binary/dynamic/msgpack'
- Document serialization format code: `0x00`
- Document schema type: `framework/rpc/phyllo`
- Document schema code: `0x51`
- Purpose: Representation of request/response data.
- Services required from below:
    - Structured data exchange
    - Name-based multiplexing
- Services provided for above:
    - Request-response structured data exchange

Messages are serialized on embedded devices in the MessagePack format and on computers in the JSON format.

### Service Interface


### Peer Interface

Each unit of structured data exchanged due to the service interface is exchanged as the serialized payload of a pub-sub message, together with a topic:

- Transaction Request message: 4-element array
    - Type: transaction message type code, `0x10` (the request is completed with the message) or `0x2f` (more messages are forthcoming as part of the request)
    - Channel ID
    - Transaction ID
    - Metadata: `None` by default or else defined by the RPC service.
    - Arguments: serialized document of client application data represented as raw bytes, arbitrary length (limited by payload size limit of lower layers).

- Transaction Response message: 4-element array
    - Type: transaction message type code, `0x20` (the response is successful and is completed with the message) or `0x21` - `0x30` (an error was encountered and the message is the final one of the response) or `0x3f` (more messages are forthcoming as part of the response)
    - Channel ID
    - Transaction ID
    - Metadata: `None` by default or else defined by the RPC service.
    - Result: serialized document of client application data represented as raw bytes, arbitrary length (limited by payload size limit of lower layers). If the return status is not `ok`, then this may contain data about the error which happened.

- Transaction Acknowledgement message:
    - Type: transaction message type code, `0x09`
    - Transaction ID

- Transaction Keepalive message:
    - Type: transaction message type code, `0x0a`
    - Transaction ID

- Transaction Cancellation message:
    - Type: transaction message type code, `0x0b`
    - Transaction ID


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
| `request/partial`         | `0x2f`    | Transaction Request message                                                |
| `response/complete`       | `0x30`    | No error, returned on success.                                             |
| `response/cancelled`      | `0x31`    | Transaction was cancelled, typically by the client.                        |
| `response/unknown`        | `0x32`    | Unknown error.                                                             |
| `response/argument`       | `0x33`    | Client specified an invalid argument.                                      |
| `response/deadline`       | `0x34`    | Deadline expired before the transaction could complete.                    |
| `response/missing`        | `0x35`    | Some requested entity was not found.                                       |
| `response/exists`         | `0x36`    | The entity that a client attempted to create already exists.               |
| `response/permission`     | `0x37`    | Client does not have permission to execute the transaction.                |
| `response/exhausted`      | `0x38`    | Some resource has been exhausted.                                          |
| `response/precondition`   | `0x39`    | Transaction rejected because system is not in state needed for it.         |
| `response/aborted`        | `0x3a`    | Transaction aborted, typically due to concurrency conflict.                |
| `response/range`          | `0x3b`    | Transaction was attempteed outside the valid range.                        |
| `response/unimplemented`  | `0x3c`    | Transaction is not implemented/supported/enabled by the server.            |
| `response/internal`       | `0x3d`    | Some invariant expected by the underlying system has been broken.          |
| `response/unavailable`    | `0x3e`    | Service is currently unavailable.                                          |
| `response/data`           | `0x3f`    | Unrecoverable data loss or corruption.                                     |
| `response/authentication` | `0x30`    | Client does not have valid authentication credentials for the transaction. |
| `response/partial`        | `0x4f`    | Transaction Response partial message                                       |


The response type codes between `0x20` and `0x30` correspond exactly to the [status codes defined by gRPC](https://github.com/grpc/grpc/blob/master/doc/statuscodes.md) but with `0x20` added to the numerical values of the gRPC status codes, and the semantics of the gRPC error codes should be the same semantics here.

#### Channel ID
The channel ID is a sequential unsigned integer, of length up to 4 bits (`0x00` - `0x0f`), which assigns a logical channel in the endpoint within which request/response transactions are sequential, such that at most one transaction is active per channel per endpoint at any given time.

#### Transaction ID
The transaction ID is a sequential unsigned integer, of length up to 32 bits (0 - 2^32-1), which uniquely identifies the transaction which the message belongs to in the given channel.

#### End of Stream
This is a boolean flag specifying whether the peer should expect more messages forthcoming as part of the request/response.


#### Sending


#### Receiving


#### Example


## RPC Endpoint

An __RPC Endpoint__ handles requests received for a particular method on the server; recall that the method is specified by the topic name of the underlying Pub-Sub Message Link.

### Service Interface
