---
layout: docs
title: Node.js Quick Start
---

<h1 class="page-header">gRPC in 3 minutes (Node.js)</h1>

<p class="lead">This document will get you started with gRPC in Node.js, including a quick example.</p>

Prerequisites
-------------

This requires Node 10.x or greater.

Install
-------

 - Clone this repository

   ```sh
   $ git clone https://github.com/grpc/grpc.git
   ```

 - Download the grpc debian packages from the [latest grpc release](https://github.com/grpc/grpc/releases) and install them.
   - Later, it will possible to install them directly using `apt-get install`

   ```sh
   $ wget https://github.com/grpc/grpc/releases/download/release-0_5_0/libgrpc_0.5.0_amd64.deb
   $ wget https://github.com/grpc/grpc/releases/download/release-0_5_0/libgrpc-dev_0.5.0_amd64.deb
   $ sudo dpkg -i libgrpc_0.5.0_amd64.deb libgrpc-dev_0.5.0_amd64.deb
   ```

 - Install this package's dependencies

   ```sh
   $ cd examples/node
   $ npm install
   ```

Try it!
-------

 - Run the server

   ```sh
   $ # from this directory (grpc_common/node).
   $ node ./greeter_server.js &
   ```

 - Run the client

   ```sh
   $ # from this directory
   $ node ./greeter_client.js
   ```

Note
----

The <a href="https://github.com/grpc/grpc/tree/master/examples/node">examples/node</a> directory has its own copy of `helloworld.proto` because it currently depends on
some Protocol Buffer 2.0 syntax that is deprecated in Protocol Buffer 3.0.

Tutorial
--------

You can find a more detailed tutorial in [gRPC Basics: Node.js](/docs/tutorials/basic/node.html).
