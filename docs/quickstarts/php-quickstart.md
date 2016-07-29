---
layout: docs
title: PHP Quickstart
---

<h1 class="page-header">PHP Quickstart</h1>

<p class="lead">This guide gets you started with gRPC in PHP with a simple working example.</p>

<div id="toc"></div>

## Before you begin

### Prerequisites

* `php`: version 5.5 or higher, 7.0 or higher
* `pecl`: version 1.9 or higher
* `composer`

Note that currently you can only create clients in PHP for gRPC services -
you can find out how to create gRPC servers in our other tutorials,
e.g. [Node.js](node-quickstart.md).

### Install gRPC

Install gRPC:

```sh
  $ [sudo] pecl install grpc
```

### Install Protobuf-PHP

You will need to install the protocol buffer compiler `protoc`
and the special plugin for generating server and client code
from `.proto` service definitions. For the first part of our
quickstart example, we've already generated the server and client
stubs from [helloworld.proto](https://github.com/grpc/grpc/tree/{{site.data.config.grpc_release_branch}}/examples/protos/helloworld.proto),
but you'll need the tools for the rest of our quickstart, as well as later
tutorials and your own projects.

To install Protobuf-PHP, run:

```sh
  $ git clone https://github.com/stanley-cheung/Protobuf-PHP
  $ cd Protobuf-PHP
  $ rake pear:package version=1.0
  $ [sudo] pear install Protobuf-1.0.tgz
```

## Download the example

You'll need a local copy of the example code to work through this quickstart. Download the example code from our Github repository (the following command clones the entire repository, but you just need the examples for this quickstart and other tutorials):

```sh
  $ # Clone the repository to get the example code:
  $ git clone https://github.com/grpc/grpc
  $ # Navigate to the "hello, world" PHP example:
  $ cd grpc/examples/php
  $ composer install
```

## Run a gRPC application

From the `examples/node/dynamic_codegen` directory:

1. Run the server

   ```sh
   $ node greeter_server.js
   ```

In another terminal, from the `examples/php` directory:

1. Run the client

   ```sh
   $ ./run_greeter_client.sh
   ```

Congratulations! You've just run a client-server application with gRPC.

## Update a gRPC service

Now let's look at how to update the application with an extra method on the
server for the client to call. Our gRPC service is defined using protocol
buffers; you can find out lots more about how to define a service in a `.proto`
file in [gRPC Basics: PHP][]. For now all you need to know is that both the
server and the client "stub" have a `SayHello` RPC method that takes a
`HelloRequest` parameter from the client and returns a `HelloResponse` from
the server, and that this method is defined like this:


```
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
Let's update this so that the `Greeter` service has two methods. Edit `examples/proto/helloworld.proto` and update it with a new `SayHelloAgain` method, with the same request and response types:

```
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

(Don't forget to save the file!)

## Generate gRPC code

Next we need to update the gRPC code used by our application to use the new service definition. From the `examples/php` directory:

```
$ proto-gen-php -i . -o . helloworld.proto
```

This regenerates `helloworld.php`, which contains our generated client classes, as well as classes for populating, serializing, and retrieving our request and response types.

## Update and run the application

We now have new generated server and client code, but we still need to implement and call the new method in the human-written parts of our example application.

### Update the server

In the same directory, open `greeter_server.js`. Implement the new method like this:

```js
function sayHello(call, callback) {
  callback(null, {message: 'Hello ' + call.request.name});
}
  
function sayHelloAgain(call, callback) {
  callback(null, {message: 'Hello again, ' + call.request.name});
}
...
```
    
### Update the client

In the same directory, open `greeter_client.php`. Call the new method like this:

```php
    $request = new helloworld\HelloRequest();
    $request->setName($name);
    list($reply, $status) = $client->SayHello($request)->wait();
    $message = $reply->getMessage();
    list($reply, $status) = $client->SayHelloAgain($request)->wait();
    $message = $reply->getMessage();
```

### Run!

Just like we did before, from the `examples/node/dynamic_codegen` directory:

1. Run the server

   ```sh
   $ node greeter_server.js
   ```

In another terminal, from the `examples/php` directory:

2. Run the client

   ```sh
   $ ./run_greeter_client.sh
   ```

## What's next

- Read a full explanation of this example and how gRPC works in our [Overview](http://www.grpc.io/docs/)
- Work through a more detailed tutorial in [gRPC Basics: PHP][]
- Explore the gRPC PHP core API in its [reference documentation](http://www.grpc.io/grpc/php/namespaces/Grpc.html)

[helloworld.proto]:../protos/helloworld.proto
[gRPC Basics: PHP]:http://www.grpc.io/docs/tutorials/basic/php.html

