---
bodyclass: docs
headline: Go Generated Code Guide
layout: docs
sidenav: doc-side-quickstart-nav.html
type: markdown
---

<p class="lead">This guide gives an overview of code generated with the protoc grpc go plugin.</p>

<div id="toc"></div>

## Compiler Invocation and Results of Code Generation
This section provides instructions for using the `grpc` plugin to `protoc-gen-go`, when compiling `.proto` files with `protoc`.
See https://developers.google.com/protocol-buffers/docs/reference/go-generated for detailed instructions on 
how to use the `protoc` compiler the `protoc-gen-go` plugin.

When `protoc` is invoked with a `--go_out=plugins=grpc:` argument, gRPC stub and service interface code gets generated
from the `.proto` files specified in the rest of the command. The names of the output files are computed in the same way as though the "plain" Go
`protoc-gen-go` was used (as per the description in https://developers.google.com/protocol-buffers/docs/reference/go-generated). But additional content
is added to the generated `.pb.go` files as follows:

Each `service Foo` in the `.proto` file results in a `FooClient interface` and a `FooServer interface` in the `.pb.go` file.
  * For example, a declaration of `service Foo` in `bar.proto` results in a `type FooServer interface` and a `type FooClient interface` in `bar.pb.go`.
  
Each `rpc` declaration within a `service` definition in the `.proto` files results in a corresponding method on the `Server` and `Client` interfaces.
Method arguments and return values differ depending on whether the method is on the client or server interface, and on the type of RPC 
(e.g. unary, client-streaming, server-streaming, bidi-streaming).

### Different RPC types and their RPC signatures in proto files
The different types RPC's (unary, client-streaming, server-streaming, and bidi-streaming),
are declared in the `service` declarations in `.proto` files as follows:

Note that type message types `MsgA` and `MsgB` are just placeholders for arbitrary protobuf message types. Also note that the descriptions
further on this document refer to the example RPC signatures in this subsection.

#### Unary Method signature
`rpc Foo(MsgA) returns (MsgB)`

#### Server-streaming Method signature
`rpc Foo(stream MsgA)`

#### Client-streaming Method signature
`rpc Foo(MsgA) returns (stream MsgB)`

#### Bidi-streaming Method signatures
`rpc Foo(stream MsgA) returns (stream MsgB)`
    
## Using the generated service interfaces:

#### Implementation and Registration of a service definition
On server side, each `service Bar` in the `.proto` file results in the function: `func RegisterFooServer(s *grpc.Server, srv BarServer)`.
The application can define a concrete implementation of the `BarServer` interface and register it with a `grpc.Server` instance 
(before starting the server instance) by using this function.

#### Unary methods
These methods have the following signature on the generated service interface:
`Foo(context.Context, *MsgA) (*MsgB, error)`

In this context, `MsgA` is the protobuf message sent from the client, and `MsgB` is the protobuf message sent back from the server.

#### Server-streaming methods
These methods have the following signature on the generated service interface:
`Foo(*MsgA, <ServiceName>_FooServer) error`

In this context, `MsgA` is the single request from the client, and the `<ServiceName>_FooServer` parameter represents the server-to-client stream
of `MsgB` messages.

`<ServiceName>_FooServer` has an embedded `grpc.ServerStream` and the following interface:

```
type <ServiceName>_FooServer interface {
	Send(*MsgB) error
	grpc.ServerStream
}
```

The server-side handler can send a stream of protobuf messages to the client through this parameters' `Send` method. End-of-stream for the server-to-client
stream is caused by the `return` of the handler method.
  
#### Client-streaming methods
These methods have the following signature on the generated service interface:
`Foo(<ServiceName>_FooServer) error`

In this context, `<ServiceName>_FooServer` can be used both to read the client-to-server message stream and to send the single server response message.

`<ServiceName>_FooServer` has an embedded `grpc.ServerStream` and the following interface:

```
type <ServiceName>_FooServer interface {
	SendAndClose(*MsgA) error
	Recv() (*MsgB, error)
	grpc.ServerStream
}
```

The server-side handler can repeatedly call `Recv` on this parameter in order to receive the full stream of
messages from the client. `Recv` returns `(nil, io.EOF)` once it has reached the end of the stream.
The single response message from the server is sent by calling the `SendAndClose` method on this `<ServiceName>_FooServer` parameter. 
Note that `SendAndClose` must be called once and only once.
  
#### Bidi-streaming methods
These methods have the following signature on the generated service interface:
`Foo(<ServiceName>_FooServer) error`

In this context, `<ServiceName>_FooServer` can be used to access both the client-to-server message stream and the server-to-client message stream.
`<ServiceName>_FooServer` has an embedded `grpc.ServerStream` and the following interface:

```
type <ServiceName>_FooServer interface {
	Send(*MsgA) error
	Recv() (*MsgB, error)
	grpc.ServerStream
}
```

The server-side handler can repeatedly call `Recv` on this parameter in order to read the client-to-server message stream.
`Recv` returns `(nil, io.EOF)` once it has reached the end of the client-to-server stream.
The response server-to-client message stream is sent by repeatedly calling the `Send` method of on this `ServiceName>_FooServer` parameter.
End-of-stream for the server-to-client stream is indicated by the `return` of the bidi method handler.

### Methods on generated client interfaces:
For client side usage, each `service Bar` in the `.proto` file also results in the function: `func BarClient(cc *grpc.ClientConn) BarClient`, which
returns a concrete implementation of the `BarClient` interface (this concrete implementation also lives in the generated `.pb.go` file).

#### Unary Methods 
These methods have the following signature on the generated client stub:
`(ctx context.Context, in *MsgA, opts ...grpc.CallOption) (*MsgB, error)`.

In this context, `MsgA` is the single request from client to server, and `MsgB` contains the response sent back from the server.

#### Server-Streaming methods
These methods have the following signature on the generated client stub:
`Foo(ctx context.Context, in *MsgA, opts ...grpc.CallOption) (<ServiceName>_FooClient, error)`.

In this context, `<ServiceName>_FooClient` represents the server-to-client `stream` of `MsgB` messages.

This stream has an embedded `grpc.ClientStream` and the following interface:

```
type <ServiceName>_FooClient interface {
	Recv() (*MsgB, error)
	grpc.ClientStream
}
```

The stream begins when the client calls the `Foo` method on the stub. 
The client can then repeatedly call the `Recv` method on the returned `<ServiceName>_FooClient` return value in order to read the server-to-client response stream. 
This `Recv` method returns `(nil, io.EOF)` once the server-to-client stream has been completely read through.

#### Client-Streaming methods
These methods have the following signature on the generated client stub:
`Foo(ctx context.Context, opts ...grpc.CallOption) (<ServiceName>_FooClient, error)`.

In this context, `<ServiceName>_FooClient` represents the client-to-server `stream` of `MsgA` messages.

`<ServiceName>_FooClient` has an embedded `grpc.ClientStream` and the following interface:

```
type <ServiceName>_FooClient interface {
	Send(*MsgA) error
	CloseAndRecv() (*MsgA, error)
	grpc.ClientStream
}
```

The stream begins when the client calls the `Foo` method on the stub. 
The client can then repeatedly call the `Send` method on the returned `<ServiceName>_FooClient` stream in order to send the client-to-server message stream.
The `CloseAndRecv` method on this stream must be called once and only once, in order to both close the client-to-server stream
and receive the single response message from the server.

#### Bidi-Streaming methods:
These methods have the following signature on the generated client stub:
`Foo(ctx context.Context, opts ...grpc.CallOption) (<ServiceName>_FooClient, error)`.

In this context, `<ServiceName>_FooClient` represents both the client-to-server and server-to-client message streams.

`<ServiceName>_FooClient` has an embedded `grpc.ClientStream` and the following interface:

```
type <ServiceName>_FooClient interface {
	Send(*MsgA) error
	Recv() (*MsgB, error)
	grpc.ClientStream
}
```

The stream begins when the client calls the `Foo` method on the stub.
The client can then repeatedly call the `Send` method on the returned `<SericeName>_FooClient` stream in order to send the 
client-to-server message stream. The client can also repeatedly call `Recv` on this stream in order to
receive the full server-to-client message stream.

End-of-stream for the server-to-client stream is indicated by a return value of `(nil, io.EOF)` on the `Recv` method of the stream.
End-of-stream for the client-to-server stream can be indicated from the client by calling the `CloseSend` method on the stream.

### Packages and Namespaces
When the `prococ` compiler is invoked with `--go_out=plugins=grpc:`, the `proto package` to `Go package` "translation"
works the same as when the `protoc-gen-go` plugin is used without the `grpc` plugin (as described in https://developers.google.com/protocol-buffers/docs/reference/go-generated).

So, for example, if `foo.proto` declares itself to be in `package foo`, then the generated `foo.pb.go` file will also be in
the "Go package" `foo`.