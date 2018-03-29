---
bodyclass: docs
headline: Rust Quick Start
layout: docs
sidenav: doc-side-quickstart-nav.html
type: markdown
---

<p class="lead">This guide gets you started with gRPC in Rust with a simple
working example.</p>

<div id="toc"></div>

## Before you begin

### Prerequisites

#### Rust version

gRPC requires Rust 1.19 or higher.

```sh
$ rustc -V
```

For installation instructions, follow this guide: [Install Rust](https://www.rust-lang.org/install.html)

You also need to install:

+ CMake >= 3.8.0
+ Go >= 1.7 if you want to enable secure feature
+ GCC or Clang for Linux and Mac OS

Under Windows, you still need to install:

+ Active State Perl
+ yasm
+ Visual Studio 2015+

You can follow the [INSTALL](https://github.com/grpc/grpc/blob/master/INSTALL.md) instruction to see how to install them.

#### Install Protocol Buffers v3

Install the protoc compiler that is used to generate gRPC service code. The simplest way to do this is to download pre-compiled binaries for your platform(`protoc-<version>-<platform>.zip`) from here: [https://github.com/google/protobuf/releases](https://github.com/google/protobuf/releases)

  * Unzip this file.
  * Update the environment variable `PATH` to include the path to the protoc binary file.

Next, install the protoc plugin for Rust

```
$ cargo install protobuf
```

Then, install the protoc gRPC plugin for Rust

```
$ cargo install grpcio-compiler
```

## Download the example

You'll need a local copy of the example code to work through this quickstart.
Download the example code from our Github repository (the following command
clones the entire repository, but you just need the examples for this quickstart
and other tutorials):

```sh
$ # Clone the repository to get the example code:
$ git clone https://github.com/pingcap/grpc-rs
$ # Navigate to the grpc-rs:
$ cd grpc-rs
```

## Run a gRPC application

From the `grpc-rs` directory:

1. Compile the client and server

   ```sh
   $ git submodule update --init --recursive
   $ cargo build --example greeter_client
   $ cargo build --example greeter_server
   ```

2. Run the server

   ```sh
   $ ./target/debug/examples/greeter_server
   ```

3. In another terminal, run the client

   ```sh
   $ ./target/debug/examples/greeter_client
   ```

Congratulations! You've just run a client-server application with gRPC.

## Update a gRPC service

Now let's look at how to update the application with an extra method on the
server for the client to call. Our gRPC service is defined using protocol
buffers; you can find out lots more about how to define a service in a `.proto`
file in [What is gRPC?](/docs/#what-is-grpc) and [gRPC Basics:
Rust][]. For now all you need to know is that both the server and the client
"stub" have a `SayHello` RPC method that takes a `HelloRequest` parameter from
the client and returns a `HelloReply` from the server, and that this method
is defined like this:

```protobuf
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

Let's update this so that the `Greeter` service has two methods. Make sure you are in the grpc-rs dir. 

Edit `proto/proto/example/helloworld.proto` and update it with a new `SayHelloAgain` method, with the same request and response
types:

```protobuf
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // Sends another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

## Generate gRPC code

Next we need to update the gRPC code used by our application to use the new
service definition.

From the `grpc-rs` directory run:

```sh
 protoc --rust_out=examples/hello_world --grpc_out=examples/hello_world --plugin=protoc-gen-grpc=`which grpc_rust_plugin` proto/proto/example/hello_world.proto
```

Running this command generates `hello_world_grpc.rs` and `hello_world.rs` files under the `hello_world` example directory.

Notice: In grpc-rs project, we generate all the files and encode them into a `.desc` file which Rust can use directly. For `hello_world`, it is in the [grpc-rs/proto/example.desc](https://github.com/pingcap/grpc-rs/blob/master/proto/example.desc).

For simpplicity, after you change the proto file, you need to rebuild the `.desc` file.

```sh
$ ./proto/build.sh
```

## Update and run the application

We now have new generated server and client code, but we still need to implement
and call the new method in the human-written parts of our example application.

### Update the server

Edit `examples/hello_world/server.rs` and add the following function to it:

```rust
fn say_hello_again(&self, ctx: RpcContext, req: HelloRequest, sink: UnarySink<HelloReply>) {
    let msg = format!("Hello {} again", req.get_name());
    let mut resp = HelloReply::new();
    resp.set_message(msg);
    let f = sink.success(resp)
        .map_err(move |e| error!("failed to reply {:?}: {:?}", req, e));
    ctx.spawn(f)
}
```

### Update the client

Edit `examples/hello_world/client.rs` to add the following code to the main function.

```rust
let reply = client.say_hello_again(&req).expect("rpc");
info!("Greeter received: {}", reply.get_message());
```

### Run!

Run the server 

```
$ cargo run --example greeter_server
```

On a different terminal, run the client 

```
$ cargo run --example greeter_client
```

You should see the updated output:

```
$ cargo run --example greeter_client
Greeter received: Hello world
received: Hello world again
```

## What's next

- Read a full explanation of how gRPC works in [What is gRPC?](../guides/)
  and [gRPC Concepts](../guides/concepts.html)
- Work through a more detailed tutorial in [gRPC Basics: Rust][]
- Explore the gRPC Rust core API in its [reference
  documentation](https://docs.rs/grpcio/)

[gRPC Basics: Rust]:../tutorials/basic/rust.html
