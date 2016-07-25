---
layout: docs
title: Objective-C Quickstart
---

<h1 class="page-header">Objective-C Quickstart</h1>

<p class="lead">This guide gets you started with gRPC on iOS platform in Objective-C with a simple working example.</p>

<div id="toc"></div>

## Before you begin

### System requirement
OS X El Capitan (version 10.11) or above is required to build and run this Quickstart.

### Prerequisites

* `CocoaPods`: version 1.0 or higher

   * Check status and version of CocoaPods on your system with command `pod --version`.
   * If CocoaPods is not installed, follow install instructions on CocoaPods [website](https://cocoapods.org).

* `Xcode`: version 7.2 or higher

   * Check your Xcode version by running Xcode from Lauchpad, then select "Xcode->About Xcode" in the menu.
   * Make sure the command line developer tools are installed:
   ```sh
   [sudo] xcode-select --install
   ```

* `Homebrew`
   * Check status and version of Homebrew on your system with command `brew --version`.
   * If Homebrew is not installed, install with:
   ```sh
   /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   ```

* `autoconf`, `libtool`, `pkg-config`
   * Install with Homebrew
   ```sh
   brew install autoconf libtool pkg-config
   ```

## Download the example

You'll need a local copy of the sample app source code to work through this
quickstart. Copy the source code from Github
[repository](https://github.com/grpc/grpc):

```sh
$ git clone https://github.com/grpc/grpc.git
```

## Install gRPC plugins and libraries
```sh
$ cd grpc
$ git submodule update --init
$ make
$ [sudo] make install
```

## Install protoc compiler
```sh
$ brew tap grpc/grpc
$ brew install google-protobuf
```

## Run the server

For this sample app, we need a gRPC server running on the local machine. gRPC
Objective-C API supports gRPC client but not gRPC server. Therefore instead we
build and run the C++ server in the same repository:

```sh
$ cd examples/cpp/helloworld
$ make
$ ./greeter_server &
```

## Run the client

### Generate client libraries and dependencies

Have CocoaPods generate and install the client library from our .proto files, as
well as installing several dependencies:

```sh
$ cd ../../objective-c/helloworld
$ pod install
```

(This might have to compile OpenSSL, which takes around 15 minutes if Cocoapods
doesn't have it yet on your computer's cache.)

### Run the client app

Open the Xcode workspace created by CocoaPods:

```sh
$ open HelloWorld.xcworkspace
```

This will open the app project with Xcode. Run the app in an iOS simulator
by pressing the Run button on the top left corner of Xcode window. You can check
the calling code in `main.m` and see the resuls in Xcode's console.

The code sends a `HLWHelloRequest` containing the string "Objective-C" to a
local server. The server responds with a `HLWHelloResponse`, which contains a
string "Hello Objective-C" that is then output to the console.

Congratulations! You've just run a client-server application with gRPC.

## Update a gRPC service

Now let's look at how to update the application with an extra method on the
server for the client to call. Our gRPC service is defined using Protocol
Buffers; you can find out lots more about how to define a service in a `.proto`
file in Protocol Buffers 
[website](https://developers.google.com/protocol-buffers/). For now all you 
need to know is that both the server and the client "stub" have a `SayHello`
RPC method that takes a `HelloRequest` parameter from the client and returns a
`HelloResponse` from the server, and that this method is defined like this:

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

## Update the client and server

We now have new gRPC service definition, but we still need to implement and call the new method in the human-written parts of our example application.

### Update the server

Recall that gRPC does not provide Objective-C version server API. We update the C++ sample server for our purpose. Open `examples/cpp/helloworld/greeter_server.cc`. Implement the new method like this:

```
class GreeterServiceImpl final : public Greeter::Service {
  Status SayHello(ServerContext* context, const HelloRequest* request,
                  HelloReply* reply) override {
    std::string prefix("Hello ");
    reply->set_message(prefix + request->name());
    return Status::OK;
  }
  Status SayHelloAgain(ServerContext* context, const HelloRequest* request,
                  HelloReply* reply) override {
    std::string prefix("Hello again ");
    reply->set_message(prefix + request->name());
    return Status::OK;
  }
};
```

### Update the client

Edit `examples/objective-c/helloworld/main.m` to call the new method like this:

```
int main(int argc, char * argv[]) {
  @autoreleasepool {
    [GRPCCall useInsecureConnectionsForHost:kHostAddress];
    [GRPCCall setUserAgentPrefix:@"HelloWorld/1.0" forHost:kHostAddress];

    HLWGreeter *client = [[HLWGreeter alloc] initWithHost:kHostAddress];

    HLWHelloRequest *request = [HLWHelloRequest message];
    request.name = @"Objective-C";

    [client sayHelloWithRequest:request handler:^(HLWHelloReply *response, NSError *error) {
      NSLog(@"%@", response.message);
    }]; 
    [client sayHelloAgainWithRequest:request handler:^(HLWHelloReply *response, NSError *error) {
      NSLog(@"%@", response.message);
    }]; 
    
    return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
  }
}
```

Note that we made the second RPC call with the new gRPC service SayHelloAgain defined above.

## Build and run

We first terminate the server process already running in the background:
```sh
$ pkill greeter_server
```

With current directory being `examples/cpp/helloworld`, build and run the server with the 
following commands:

```sh
$ make
$ ./greeter_server &
```

Change directory to `examples/objective-c/helloworld`, clean up and reinstall Pods for
the client app with the following commands:

```sh
$ rm -Rf Pods
$ rm Podfile.lock
$ rm -Rf HelloWorld.xcworkspace
$ pod install
```

This regenerates files in `Pods/HelloWorld` based on the new proto file we wrote above.
Open the client Xcode project in Xcode:
```sh
$ open HelloWorld.xcworkspace
```
and run the client app. Note the effect of the new service by looking at the console messages.
You should see two RPC calls, one call on SayHello and another call on SayHelloAgain.
