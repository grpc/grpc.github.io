---
bodyclass: docs
headline: C# Quick Start
layout: docs
sidenav: doc-side-quickstart-nav.html
type: markdown
---

<h1 class="page-header">C# Quickstart</h1>

<p class="lead">This guide gets you started with gRPC in C# with a simple
working example.</p>

<div id="toc"></div>

## Before you begin

### Prerequisites

Whether you're using Windows, OS X, or Linux, you can follow this
example by using either an IDE and its build tools,
or by using the the .NET Core SDK command line tools.

First, make sure you have installed the
[gRPC C# prerequisites](https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_tag }}/src/csharp/README.md#prerequisites).
You will also need Git to download the sample code.

## Download the example

You'll need a local copy of the example code to work through this quickstart.
Download the example code from our Github repository (the following command
clones the entire repository, but you just need the examples for this quickstart
and other tutorials):

```sh
$ # Clone the repository to get the example code:
$ git clone -b {{ site.data.config.grpc_release_tag }} https://github.com/grpc/grpc 
$ cd grpc
```

This document will walk you through the "Hello World" example.
The projects and source files can be found in the `examples/csharp/Helloworld` directory.

The example in this walkthrough already adds the necessary
dependencies for you (Grpc, Grpc.Tools and Google.Protobuf NuGet packages).

## Build the example

### Using Visual Studio (or Visual Studio for Mac)
* Open the solution `Greeter.sln` with Visual Studio
* Build the solution

### Using .NET Core SDK from the command line
From the `examples/csharp/Helloworld` directory:

```
> dotnet build Greeter.sln
```

*NOTE: If you want to use gRPC C# from a project that uses the old-style .csproj files (supported by Visual Studio 2013, 2015 and older versions of Mono), please refer to the
[Greeter using legacy .csproj](https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_tag }}/examples/csharp/HelloworldLegacyCsproj/README.md) example.*

## Run a gRPC application

From the `examples/csharp/Helloworld` directory:

* Run the server

```
> cd GreeterServer
> dotnet run -f netcoreapp2.1
```

* In another terminal, run the client

```
> cd GreeterClient
> dotnet run -f netcoreapp2.1
```

Congratulations! You've just run a client-server application with gRPC.

## Update a gRPC service

Now let's look at how to update the application with an extra method on the
server for the client to call. Our gRPC service is defined using protocol
buffers; you can find out lots more about how to define a service in a `.proto`
file in [gRPC Basics: C#][]. For now all you need to know is that both the
server and the client "stub" have a `SayHello` RPC method that takes a
`HelloRequest` parameter from the client and returns a `HelloResponse` from the
server, and that this method is defined like this:


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

Let's update this so that the `Greeter` service has two methods. Edit
`examples/protos/helloworld.proto` and update it with a new `SayHelloAgain`
method, with the same request and response types:

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

Next we need to update the gRPC code used by our application to use the new service definition. 

The `Grpc.Tools` NuGet package contains the protoc and protobuf C# plugin binaries you will need to generate the code. 

### Obtaining the Grpc.Tools NuGet package

This example project already depends on the `Grpc.Tools.{{ site.data.config.grpc_release_tag | remove_first: "v" }}` NuGet package
and the package will be downloaded to your local NuGet cache as soon as you restore the nuget packages by clicking "Restore NuGet Packages" in Visual Studio or running `dotnet restore RouteGuide.sln` from the `examples/csharp/RouteGuide` directory.

### Commands to generate the gRPC code
Note that you may have to change the `platform_architecture` directory names (e.g. windows_x86, linux_x64) in the commands below based on your environment.

From the `examples/csharp/Helloworld` directory:

**Windows**

```
@rem Local nuget cache on Windows is located in %UserProfile%\.nuget\packages
> %UserProfile%\.nuget\packages\Grpc.Tools.{{ site.data.config.grpc_release_tag | remove_first: "v" }}\tools\windows_x86\protoc.exe -I../../protos --csharp_out Greeter --grpc_out Greeter ../../protos/helloworld.proto --plugin=protoc-gen-grpc=%UserProfile%\.nuget\packages\packages\Grpc.Tools.{{ site.data.config.grpc_release_tag | remove_first: "v" }}\tools\windows_x86\grpc_csharp_plugin.exe
```

**Linux (or OS X by using macosx_x64 directory)**

```
# Local nuget cache on Linux and Mac is located in ~/.nuget/packages
$ ~/.nuget/packages/packages/grpc.tools.{{ site.data.config.grpc_release_tag | remove_first: "v" }}/tools/linux_x64/protoc -I../../protos --csharp_out Greeter --grpc_out Greeter ../../protos/helloworld.proto --plugin=protoc-gen-grpc=~/.nuget/packages/grpc.tools.{{ site.data.config.grpc_release_tag | remove_first: "v" }}/tools/linux_x64/grpc_csharp_plugin
```

Running the appropriate command for your OS regenerates the following files in
the directory:

* Greeter/Helloworld.cs contains all the protocol buffer code to populate,
  serialize, and retrieve our request and response message types
* Greeter/HelloworldGrpc.cs provides generated client and server classes,
  including:
    * an abstract class Greeter.GreeterBase to inherit from when defining
      Greeter service implementations
    * a class Greeter.GreeterClient that can be used to access remote Greeter
      instances
    
## Update and run the application

We now have new generated server and client code, but we still need to implement
and call the new method in the human-written parts of our example application.

### Update the server

With the `Greeter.sln` open in your IDE, open `GreeterServer/Program.cs`.
Implement the new method by editing the GreeterImpl class like this:

```
class GreeterImpl : Greeter.GreeterBase
{
    // Server side handler of the SayHello RPC
    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        return Task.FromResult(new HelloReply { Message = "Hello " + request.Name });
    }

    // Server side handler for the SayHelloAgain RPC
    public override Task<HelloReply> SayHelloAgain(HelloRequest request, ServerCallContext context)
    {
        return Task.FromResult(new HelloReply { Message = "Hello again " + request.Name });
    }
}
```

### Update the client

With the same `Greeter.sln` open in your IDE, open `GreeterClient/Program.cs`.
Call the new method like this:

```
public static void Main(string[] args)
{
    Channel channel = new Channel("127.0.0.1:50051", ChannelCredentials.Insecure);

    var client = new Greeter.GreeterClient(channel);
    String user = "you";

    var reply = client.SayHello(new HelloRequest { Name = user });
    Console.WriteLine("Greeting: " + reply.Message);
    
    var secondReply = client.SayHelloAgain(new HelloRequest { Name = user });
    Console.WriteLine("Greeting: " + secondReply.Message);

    channel.ShutdownAsync().Wait();
    Console.WriteLine("Press any key to exit...");
    Console.ReadKey();
}
```

### Rebuild the modified example

Rebuild the newly modified example just like we first built the original
example by running `dotnet build Greeter.sln`.

### Run!

Just like we did before, from the `examples/csharp/Helloworld` directory:

* Run the server

```
> cd GreeterServer
> dotnet run -f netcoreapp2.1
```

* In another terminal, run the client

```
> cd GreeterClient
> dotnet run -f netcoreapp2.1
```

## What's next

- Read a full explanation of how gRPC works in [What is gRPC?](../guides/)
  and [gRPC Concepts](../guides/concepts.html)
- Work through a more detailed tutorial in [gRPC Basics: C#][]
- Explore the gRPC C# core API in its [reference
  documentation](/grpc/csharp/api/Grpc.Core.html)

[helloworld.proto]:../protos/helloworld.proto
[gRPC Basics: C#]:../tutorials/basic/csharp.html
