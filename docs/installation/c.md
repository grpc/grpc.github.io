---
layout: docs
title: C++ Quick Start
---

<h1 class="page-header">gRPC in 3 minutes (C++)</h1>
<p class="lead">This document will get you started with gRPC in C++, including a quick example.</p>

## Installation

To install gRPC on your system, follow the instructions here:
[https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/INSTALL](https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/INSTALL).

## Hello C++ gRPC!

Here's how to build and run the C++ implementation of the [Hello World](https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/protos/helloworld.proto) example used in the [Overview](/docs/index.shtml).

The example code for this and our other examples lives in the `examples`
directory. Clone this repository to your local machine by running the
following command:


```sh
$ git clone https://github.com/grpc/grpc.git
```

Change your current directory to examples/cpp/helloworld

```sh
$ cd examples/cpp/helloworld/
```


### Generating gRPC code

To generate the client and server side interfaces:

```sh
$ make helloworld.grpc.pb.cc helloworld.pb.cc
```
Which internally invokes the proto-compiler as:

```sh
$ protoc -I ../../protos/ --grpc_out=. --plugin=protoc-gen-grpc=grpc_cpp_plugin ../../protos/helloworld.proto
$ protoc -I ../../protos/ --cpp_out=. ../../protos/helloworld.proto
```

### Client and server implementations

The client implementation is at [greeter_client.cc](https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/cpp/helloworld/greeter_client.cc).

The server implementation is at [greeter_server.cc](https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/cpp/helloworld/greeter_server.cc).

### Try it!
Build client and server:

```sh
$ make
```
Run the server, which will listen on port 50051:

```sh
$ ./greeter_server
```
Run the client (in a different terminal):

```sh
$ ./greeter_client
```
If things go smoothly, you will see the "Greeter received: Hello world" in the client side output.

## Tutorial

You can find a more detailed tutorial in [gRPC Basics: C++](/docs/tutorials/basic/c.html)
