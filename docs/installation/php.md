---
layout: docs
title: PHP Quick Start
---

<h1 class="page-header">gRPC in 3 minutes (PHP)</h1>

This document shows you how to install and build a PHP client for a simple Hello World example, as used in the [Overview](/docs/index.shtml).

PREREQUISITES
-------------

This requires PHP 5.5 or greater.

INSTALL
-------
 - On Mac OS X, install [homebrew][]. On Linux, install [linuxbrew][]. Run the following command to install gRPC.

   ```sh
   $ curl -fsSL https://goo.gl/getgrpc | bash -s php
   ```
   This will download and run the [gRPC install script][] and compile the gRPC PHP extension.

 - Clone this repository

   ```sh
   $ git clone https://github.com/grpc/grpc-common.git
   ```

 - Install composer

   ```
   $ cd grpc-common/php
   $ curl -sS https://getcomposer.org/installer | php
   $ php composer.phar install
   ```

TRY IT!
-------

 - Run a gRPC server. Follow the instruction in [Node][] to run an example server (we only support writing clients in PHP):

   ```
   $ cd grpc-common/node
   $ nodejs greeter_server.js
   ```

 - Run the client:

   ```
   $ cd grpc-common/php
   $ ./run_greeter_client.sh
   ```

NOTE
----

While most of our Hello World examples use the same .proto file, the PHP example has its own copy of `helloworld.proto` because it currently depends on
some proto2 syntax. There is no proto3 support for PHP yet.

TUTORIAL
--------

You can find a more detailed tutorial in [gRPC Basics: PHP](/docs/tutorials/basic/php.html).

[homebrew]:http://brew.sh
[linuxbrew]:https://github.com/Homebrew/linuxbrew#installation
[gRPC install script]:https://raw.githubusercontent.com/grpc/homebrew-grpc/master/scripts/install
[Node]:/docs/installation/node
