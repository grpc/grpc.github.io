---
layout: docs
title: Objective-C Quickstart
---

<h1 class="page-header">Objective-C Quickstart</h1>

<p class="lead">This guide gets you started with gRPC on iOS platform in Objective-C with a simple working example.</p>

<div id="toc"></div>

## Before you begin

### Prerequisites

* `CocoaPods`: version 1.0 or higher
* `XCode`: version 7.2 or higher
* `Homebrew`

### Install Protocol Buffer compiler and gRPC plugins
Use our formula to install Protocol Buffer and gRPC plugins:

```sh
brew tap grpc/grpc
brew install --with-plugins grpc
```

## Download the example

You'll need a local copy of the sample app source code to work through this
quickstart. Copy the source code from Github
[repository](https://github.com/grpc/grpc) (the following command clones the
entire repository, but you just need the examples for this quickstart and other
tutorials):

```sh
  $ git clone https://github.com/grpc/grpc
  $ cd grpc/examples/objective-c/helloworld
```

## Generate client libraries and dependencies

Have CocoaPods generate and install the client library for our .proto files, and
obtain the protocol buffer compiler `protoc`:

```sh
  $ pod install
```

(This might have to compile OpenSSL, which takes around 15 minutes if Cocoapods
doesn't have it yet on your computer's cache.)

## Run the server

For this sample app, we need a gRPC server running on the local machine. gRPC
Objective-C API supports gRPC client but not gRPC server. Therefore insteam we
build and run the C++ server in the same repository:

```sh
$ pushd ../../cpp/helloworld
$ make
$ ./greeter_server &
$ popd
```

## Run the client app

Open the XCode workspace created by CocoaPods:

```sh
  $ open HelloWorld.xcworkspace
```

This will open the app project with XCode. Run the app in an iOS simulator
by pressing the Run button on the top left corner of XCode window. You can check
the calling code in `main.m` and see the resuls in XCode's console.

The code sends a `HLWHelloRequest` containing the string "Objective-C" to a
local server. The server responds with a `HLWHelloResponse`, which contains a
string "Hello Objective-C" that is then output to the console.

Congratulations! You've just run a client-server application with gRPC.

