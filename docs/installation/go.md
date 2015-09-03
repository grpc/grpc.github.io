---
layout: docs
title: Go Quick Start
---

<h1 class="page-header">gRPC in 3 minutes (Go)</h1>

<p class="lead">This document will get you started with gRPC in Go, including a quick example.</p>

Prerequisites
-------------

- gRPC requires Go 1.4
- It also requires that [GOPATH is set](https://golang.org/doc/code.html#GOPATH)

```sh
$ go help gopath
$ # ensure the PATH contains $GOPATH/bin
$ export PATH=$PATH:$GOPATH/bin
```

Install
-------

-  First [install protoc](https://github.com/google/protobuf/blob/master/INSTALL.txt).
  - For now, this needs to be installed from source
  - This is will change once proto3 is officially released

- Install the protoc Go plugin.

```sh
$ go get -a github.com/golang/protobuf/protoc-gen-go
```

- Get the example.

```sh
$ go get -u github.com/grpc/grpc-go/examples/helloworld/greeter_client
$ go get -u github.com/grpc/grpc-go/examples/helloworld/greeter_server
```

Try it!
-------

For this sample, we've already generated the server and client stubs from [helloworld.proto](https://github.com/grpc/grpc-go/blob/master/examples/helloworld/proto/helloworld.proto). 

- Run the server
```sh
$ greeter_server &
```

- Run the client
```sh
$ greeter_client
```

OPTIONAL - Rebuilding the generated code
----------------------------------------

You must have protoc and the Go protoc plugin installed to do this:

```sh
$ # from the examples/helloworld dir; invoke protoc
$ protoc -I ../proto ../proto/helloworld.proto --go_out=plugins=grpc:helloworld
```

