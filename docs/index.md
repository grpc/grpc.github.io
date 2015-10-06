---
layout: docs
title: Documentation
---

<h1>Getting started</h1>

<p class="lead">Welcome to the developer documentation for gRPC, a language-neutral, platform-neutral, open source, remote procedure call (RPC) system initially developed at Google.</p>

This document introduces you to gRPC with a quick overview and a simple
Hello World example. You'll find more tutorials and reference docs in this site - more documentation is coming soon!

<div id="toc"></div>

<!--=========================================================================-->
<a name="quickstart"></a>
## Quick start
To get up and running with gRPC straight away, see the quick start for your chosen language, which provides links to installation instructions, quick instructions for building the example used in this guide, and more:

* [C++](https://github.com/grpc/grpc/tree/master/examples/cpp)
* [Java](https://github.com/grpc/grpc-java/tree/master/examples)
* [Go](https://github.com/grpc/grpc-go/tree/master/examples)
* [Python](https://github.com/grpc/grpc/tree/master/examples/python/helloworld)
* [Ruby](https://github.com/grpc/grpc/tree/master/examples/ruby)
* [Node.js](https://github.com/grpc/grpc/tree/master/examples/node)
* [Android Java](https://github.com/grpc/grpc-java/tree/master/examples/android)
* [C#](https://github.com/grpc/grpc/tree/master/examples/csharp/helloworld)
* [Objective-C](https://github.com/grpc/grpc/tree/master/examples/objective-c/helloworld)
* [PHP](https://github.com/grpc/grpc/tree/master/examples/php)

You can find out about the gRPC source code repositories in
[grpc](https://github.com/grpc/grpc). Most of our example code lives in the [examples](https://github.com/grpc/grpc/tree/{{ site.data.config.grpc_release_branch }}/examples) directory.

<!--=========================================================================-->
## What is gRPC?

In gRPC a *client* application can directly call
methods on a *server* application on a different machine as if it was a
local object, making it easier for you to create distributed applications and
services. As in many RPC systems, gRPC is based around the idea of defining
a *service*, specifying the methods that can be called remotely with their
parameters and return types. On the server side, the server implements this
interface and runs a gRPC server to handle client calls. On the client side,
the client has a *stub* that provides exactly the same methods as the server.

<img src="../img/grpc_concept_diagram_00.png" class="img-responsive" alt="gRPC diagram">

gRPC clients and servers can run and talk to each other in a variety of
environments - from servers inside Google to your own desktop - and can
be written in any of gRPC's [supported languages](#quickstart). So, for
example, you can easily create a gRPC server in Java with clients in Go,
Python, or Ruby. In addition, the latest Google APIs will have gRPC versions
of their interfaces, letting you easily build Google functionality into
your applications.

<a name="protocolbuffers"></a>
### Working with protocol buffers

By default gRPC uses *protocol buffers*, Google’s
mature open source mechanism for serializing structured data (although it
can be used with other data formats such as JSON). As you'll
see in our example below, you define gRPC services using *proto files*,
with method parameters and return types specified as protocol buffer message
types. You
can find out lots more about protocol buffers in the [Protocol Buffers
documentation](https://developers.google.com/protocol-buffers/docs/overview).

#### Protocol buffer versions

While protocol buffers have been available for open source users for some
time, our examples use a new flavor of protocol buffers called proto3,
which has a slightly simplified syntax, some useful new features, and supports
lots more languages. This is currently available as an beta release in
Java and C++, with an alpha release for JavaNano (Android Java), Python, and
Ruby from [the protocol buffers Github
repo](https://github.com/google/protobuf/releases), as well as a Go language
generator from [the golang/protobuf Github repo](https://github.com/golang/protobuf), with more languages in development. You can find out more in the [proto3 language guide](https://developers.google.com/protocol-buffers/docs/proto3), and see
the major differences from the current default version in the [release notes](https://github.com/google/protobuf/releases). More proto3 documentation is coming soon.

In general, while you *can* use proto2 (the current default protocol buffers version), we recommend that you use proto3 with gRPC as it lets you use the full range of gRPC-supported languages, as well as avoiding compatibility
issues with proto2 clients talking to proto3 servers and vice versa.

<!--=========================================================================-->
<a name="hello"></a>
## Hello gRPC!

Now that you know a bit more about gRPC, the easiest way to see how it
works is to look at a simple example. Our Hello World walks you through the
construction of a simple gRPC client-server application, showing you how to:

- Create a protocol buffers schema that defines a simple RPC service with
a single
Hello World method.
- Create a server that implements this interface in your favourite language (where available).
- Create a client in your favourite language (or any other one you like!) that accesses your server.

The complete code for the examples is available in the `examples` directory in our
GitHub repositories. We use the Git versioning system for source code management:
however, you don't need to know anything about Git to follow along other
than how to install and run a few git commands.

Note that server code for our example isn't available in all gRPC languages, as gRPC PHP and Objective-C only support creating clients.

This is an introductory example rather than a comprehensive tutorial for any particular language. You can find more in-depth tutorials in this site, and
reference documentation for all gRPC languages is coming soon.

<!--=================================-->
<a name="setup"></a>
### Setup

This section explains how to set up your local machine to work with
the example code. If you just want to read the examples, you can go straight
to the [next step](#servicedef).

#### Install Git

You can download and install Git from http://git-scm.com/download. Once
installed you should have access to the git command line tool. The main
commands that you will need to use are:

- git clone ... : clone a remote repository onto your local machine
- git checkout ... : check out a particular branch or a tagged version of
the code to hack on

#### Install gRPC

To build and install gRPC plugins and related tools, see the [Quickstart](#quickstart) for your chosen language(s). Java gRPC does not require any installation other than the JDK.

#### Get the source code

<div class="tabs">
  <ul>
    <li><a href="#java_source">Java</a></li>
    <li><a href="#cpp_source">C++</a></li>
    <li><a href="#python_source">Python</a></li>
    <li><a href="#go_source">Go</a></li>
    <li><a href="#ruby_source">Ruby</a></li>
    <li><a href="#node_source">Node.js</a></li>
    <li><a href="#csharp_source">C#</a></li>
    <li><a href="#objective-c_source">Objective-C</a></li>
    <li><a href="#php_source">PHP</a></li>
  </ul>
  <div id="java_source">
The example code for our Java example lives in the <code>grpc-java</code>
GitHub repository. Clone this repository to your local machine by running the
following command:
<pre>
git clone https://github.com/grpc/grpc-java.git
</pre>

Change your current directory to <code>grpc-java/examples</code>.
<pre>
cd grpc-java/examples
</pre>
  </div>
  <div id="cpp_source">
<p>The example code for this lives in the <code>examples</code> directory in our GitHub repositories. Clone this repository to your local machine by running the following command:

<pre>
$ git clone https://github.com/grpc/grpc.git
</pre>

<p>Change your current directory to <code>examples/cpp/helloworld</code>

<pre>
$ cd examples/cpp/helloworld/
</pre>
  </div>
  <div id="python_source">
<p>The example code for this lives in the <code>examples</code> directory in our GitHub repositories. Clone this repository to your local machine by running the following command:

<pre>
$ git clone https://github.com/grpc/grpc.git
</pre>

<p>Change your current directory to <code>examples/python/helloworld</code>

<pre>
$ cd examples/python/helloworld/
</pre>
</div>
  <div id="go_source">
<p>Get the example:
<pre>
$ go get -u github.com/grpc/grpc-go/examples/helloworld/greeter&lowbar;client
$ go get -u github.com/grpc/grpc-go/examples/helloworld/greeter&lowbar;server
</pre>
<p>Change your current directory to <code>examples/helloworld</code>
  </div>
  <div id="ruby_source">
<p>The example code for this lives in the <code>examples</code> directory in our GitHub repositories. Clone this repository to your local machine by running the following command:</p>

<pre>
$ git clone https://github.com/grpc/grpc.git
</pre>

<p>Change your current directory to <code>examples/ruby</code>. Then use bundler to install the example package's dependencies:</p>

<pre>
$ gem install bundler # if you don't already have bundler available
$ bundle install
</pre>

  </div>
  <div id="node_source">
<p>The example code for this lives in the <code>examples</code> directory in our GitHub repositories. Clone this repository to your local machine by running the following command:</p>

<pre>
$ git clone https://github.com/grpc/grpc.git
</pre>

<p>Change your current directory to <code>examples/node</code>, then install this package's dependencies:</p>
<pre>
$ cd examples/node
$ npm install
</pre>

  </div>
  <div id="csharp_source">
<p>The example code for this lives in the <code>examples</code> directory in our GitHub repositories. Clone this repository to your local machine by running the following command:</p>

<pre>
$ git clone https://github.com/grpc/grpc.git
</pre>

<p>Open <code>Greeter.sln</code> from Visual Studio (or Monodevelop on Linux). See the <a href="/docs/installation/csharp.html">C# Quickstart</a> for platform-specific setup.</p>
  </div>
<div id="objective-c_source">
<p>The example code for this lives in the <code>examples</code> directory in our GitHub repositories. Clone this repository to your local machine by running the following command:</p>

<pre>
$ git clone https://github.com/grpc/grpc.git
</pre>

<p>Change your current directory to <code>examples/objective-c/helloworld</code>.</p>

</div>
  <div id="php_source">
<p>The example code for this lives in the <code>examples</code> in our GitHub repositories. Clone this repository to your local machine by running the following command:</p>

<pre>
$ git clone https://github.com/grpc/grpc.git
</pre>

<p>Change your current directory to <code>examples/php</code>.</p>

<p class="note">While most of our Hello World examples use the same .proto file, the PHP example has its own copy of <code>helloworld.proto</code> because it currently depends on some proto2 syntax. There is no proto3 support for PHP yet.</p>
  </div>
</div>


<!--=================================-->
<a name="servicedef"></a>
### Defining a service

The first step in creating our example is to define a *service*: an RPC
service specifies the methods that can be called remotely with their parameters
and return types. As you saw in the
[overview](#protocolbuffers) above, gRPC does this using [protocol
buffers](https://developers.google.com/protocol-buffers/docs/overview). We
use the protocol buffers interface definition language (IDL) to define our
service methods, and define the parameters and return
types as protocol buffer message types. Both the client and the
server use interface code generated from the service definition.

Here's our example service definition, defined using protocol buffers IDL in
[helloworld.proto](https://github.com/grpc/grpc-java/tree/master/examples/src/main/proto). The `Greeter`
service has one method, `SayHello`, that lets the server receive a single
`HelloRequest`
message from the remote client containing the user's name, then send back
a greeting in a single `HelloReply`. This is the simplest type of RPC you
can specify in gRPC - you can find out about other types in the tutorial for your chosen language.

```proto
syntax = "proto3";

option java_package = "io.grpc.examples";

package helloworld;

// The greeter service definition.
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

<!--=================================-->
<a name="generating"></a>
### Generating gRPC code

Once we've defined our service, we use the protocol buffer compiler
`protoc` to generate the special client and server code we need to create
our application - you
can generate gRPC code in any gRPC-supported language, although PHP and Objective-C only support creating clients. The generated code contains both stub code for clients to
use and an abstract interface for servers to implement, both with the method
defined in our `Greeter` service.

(If you didn't install the gRPC plugins and protoc on your system and are just reading along with
the example, you can skip this step and move
onto the next one where we examine the generated code.)

<div class="tabs">
  <ul>
    <li><a href="#java_generate">Java</a></li>
    <li><a href="#cpp_generate">C++</a></li>
    <li><a href="#python_generate">Python</a></li>
    <li><a href="#go_generate">Go</a></li>
    <li><a href="#ruby_generate">Ruby</a></li>
    <li><a href="#node_generate">Node.js</a></li>
    <li><a href="#csharp_generate">C#</a></li>
    <li><a href="#objective-c_generate">Objective-C</a></li>
    <li><a href="#php_generate">PHP</a></li>
  </ul>
  <div id="java_generate">
<p>The build system for this example is also part of Java gRPC itself's build —
for simplicity we recommend using our pre-generated code for the example. You
can refer to the <a
href="https://github.com/grpc/grpc-java/blob/master/README.md">README</a> for
how to generate code from your own .proto files.

<p>Pre-generated code for the examples is available in <a
href="https://github.com/grpc/grpc-java/tree/master/examples/src/generated/main">src/generated/main</a>.
The following classes contain all the generated code we need to create our
example:

<ul><li><code>HelloRequest.java</code>, <code>HelloResponse.java</code>, and
  others which have all the protocol buffer code to populate, serialize, and
  retrieve our <code>HelloRequest</code> and <code>HelloReply</code> message types
<li><code>GreeterGrpc.java</code>, which contains (along with some other useful code):
    <ul><li>an interface for <code>Greeter</code> servers to implement

    <pre>
public static interface Greeter {
    public void sayHello(Helloworld.HelloRequest request,
        StreamObserver&lt;Helloworld.HelloReply> responseObserver);
}
    </pre>

    <li> <em>stub</em> classes that clients can use to talk to a <code>Greeter</code> server. As you can see, the async stub also implements the <code>Greeter</code> interface.

  <pre>
public static class GreeterStub extends AbstractStub&lt;GreeterStub>
    implements Greeter {
  ...
}
  </pre>
</ul>
</ul>
  </div>
  <div id="cpp_generate">
<p>To generate the client and server side interfaces, run:

<pre>
$ make helloworld.grpc.pb.cc helloworld.pb.cc
</pre>

<p>Which internally invokes the protocol buffer compiler as:

<pre>
$ protoc -I ../../protos/ --grpc_out=. --plugin=protoc-gen-grpc=grpc_cpp_plugin ../../protos/helloworld.proto
$ protoc -I ../../protos/ --cpp_out=. ../../protos/helloworld.proto
</pre>

<p>This generates:
<ul><li><code>helloworld.pb.h</code>, which declares classes for populating, serializing, and retrieving our <code>HelloRequest</code> and <code>HelloResponse</code> message types, and its implementation <code>helloworld.pb.cc</code>.
<li><code>helloworld.grpc.pb.h</code>, which declares our generated service classes, and its implementation <code>helloworld.grpc.pb.cc</code>.
</ul>
  </div>
  <div id="python_generate">
<p>To generate the client and server side interfaces:

<pre>
$ ./run_codegen.sh
</pre>

<p>Which internally invokes the protocol buffer compiler as:

<pre>$ protoc -I ../../protos --python_out=. --grpc_out=. --plugin=protoc-gen-grpc=&#96;which grpc&lowbar;python&lowbar;plugin&#96; ../../protos/helloworld.proto</pre>

<p>This generates <code>helloworld_pb2.py</code>, which contains our generated client and server classes, as well as classes for populating, serializing, and retrieving our <code>HelloRequest</code> and <code>HelloResponse</code> message types.

</div>
  <div id="go_generate">
To generate the client and server side interfaces, run the protocol buffer compiler:

```
protoc -I ../protos ../protos/helloworld.proto --go_out=plugins=grpc:helloworld
```

This generates `helloworld.pb.go`, which contains our generated client and server code, as well as code for populating, serializing, and retrieving our `HelloRequest` and `HelloResponse` message types.

  </div>
  <div id="ruby_generate">
To generate the client and server side interfaces, run the protocol buffer compiler:</p>

```
protoc -I ../protos --ruby_out=lib --grpc_out=lib --plugin=protoc-gen-grpc=`which grpc_ruby_plugin` ../protos/helloworld.proto
```

This generates the following files in the <code>lib</code> directory:

- `lib/helloworld.rb` defines a module `Helloworld`, which provides all the protocol buffer code to populate, serialize, and retrieve our request and response message types.
- `lib/helloworld_services.rb` extends the `Helloworld` module with our generated client and server classes.

  </div>
  <div id="node_generate">
The Node.js library dynamically generates service descriptors and client stub definitions from .proto files loaded at runtime, so there's no need to generate any special code when using this language. Instead, in our example server and client we `require` the gRPC library, then use its `load()` method:

```
var grpc = require('grpc');
var hello_proto = grpc.load(PROTO_PATH).helloworld;
```
  </div>
  <div id="csharp_generate">

- To generate the code on Windows, we use `protoc.exe` from the `Google.Protobuf` NuGet package and `grpc_csharp_plugin.exe` from the `Grpc.Tools` NuGet package (both under the `tools` directory).
Normally you would need to add the `Grpc.Tools` package to the solution yourself, but in this tutorial it has been already done for you. The following command should be run from the `examples/csharp/helloworld` directory: 

  ```
  > packages\Google.Protobuf.3.0.0-alpha4\tools\protoc.exe -I../../protos --csharp_out Greeter --grpc_out Greeter --plugin=protoc-gen-grpc=packages\Grpc.Tools.0.7.0\tools\grpc_csharp_plugin.exe ../../protos/helloworld.proto
  ```

- On Linux or OS X, we rely on `protoc` and `grpc_csharp_plugin` being installed by Linuxbrew/Homebrew. Run this command from the route_guide directory:

  ```
  $ protoc -I../../protos --csharp_out Greeter --grpc_out Greeter --plugin=`which grpc_csharp_plugin` ../../protos/helloworld.proto
  ```

Running the appropriate command for your OS regenerates the following files in the Greeter directory:

- `Greeter/Helloworld.cs` defines a namespace `Helloworld`
  - This contains all the protocol buffer code to populate, serialize, and retrieve our request and response message types
- `Greeter/HelloworldGrpc.cs`, provides stub and service classes, including:
   - an interface `Greeter.IGreeter` to inherit from when defining RouteGuide service implementations
   - a class `Greeter.GreeterClient` that can be used to access remote RouteGuide instances


  </div>
<div id="objective-c_generate">
For simplicity, we've provided a [Podspec file](https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/objective-c/helloworld/HelloWorld.podspec) that runs protoc for you with the appropriate plugin, input, and output, and describes how to compile the generated files. You just need to run in `examples/objective-c/route_guide`:

```
$ pod install
```

You can then open the XCode workspace created by Cocoapods to see the generated code. Running the command generates:

- `Helloworld.pbobjc.h`, the header which declares your generated message classes.
- `Helloworld.pbobjc.m`, which contains the implementation of your message classes.
- `Helloworld.pbrpc.h`, the header which declares your generated service classes.
- `Helloworld.pbrpc.m`, which contains the implementation of your service classes.

</div>

  <div id="php_generate">
gRPC PHP uses the [protoc-gen-php](https://github.com/datto/protobuf-php) tool to generate code from .proto files. You can find out how to install this in the [PHP Quickstart](/docs/installation/php.html). To generate the code for our Greeter service, run:

```
protoc-gen-php -i . -o . ./helloworld.proto
```

This generates `helloworld.php`, which contains:

- All the protocol buffer code to populate, serialize, and retrieve our request and response message types.
- A class called `GreeterClient` that lets clients call the methods defined in the `Greeter` service.


  </div>
</div>


<!--=================================-->
<a name="server"></a>
### Writing a server

Now let's write some code! First we'll create a server application to implement
our service (which, you'll remember, we can do in all gRPC languages except Objective-C and PHP). We're not going to go into a lot of detail about how
to create a server in this section - more detailed information will be in the
tutorial for your chosen language.

#### Service implementation

<div class="tabs">
  <ul>
    <li><a href="#java_service">Java</a></li>
    <li><a href="#cpp_service">C++</a></li>
    <li><a href="#python_service">Python</a></li>
    <li><a href="#go_service">Go</a></li>
    <li><a href="#ruby_service">Ruby</a></li>
    <li><a href="#node_service">Node.js</a></li>
    <li><a href="#csharp_service">C#</a></li>
  </ul>
  <div id="java_service">
<p><a href="https://github.com/grpc/grpc-java/blob/master/examples/src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java#L51">GreeterImpl.java</a>
actually implements our <code>Greeter</code> service's required behaviour.</p>

<p>As you can see, the class <code>GreeterImpl</code> implements the interface
<code>GreeterGrpc.Greeter</code> that we <a href="#generating">generated</a> from our proto
<a href="https://github.com/grpc/grpc-java/tree/master/examples/src/main/proto">IDL</a> by implementing the method <code>sayHello</code>:</p>
<pre>
@Override
public void sayHello(HelloRequest req, StreamObserver&lt;HelloReply&gt; responseObserver) {
  HelloReply reply = HelloReply.newBuilder().setMessage("Hello " + req.getName()).build();
  responseObserver.onNext(reply);
  responseObserver.onCompleted();
}
</pre>
<ul>
<li><code>sayHello</code> takes two parameters:

<ul>
<li><code>HelloRequest</code>: the request</li>
<li><code>StreamObserver&lt;HelloReply&gt;</code>: a response observer, which is
a special interface for the server to call with its response</li>
</ul></li>
</ul>

<p>To return our response to the client and complete the call:</p>

<ol>
<li>We construct and populate a <code>HelloReply</code> response object with our exciting
message, as specified in our interface definition.</li>
<li>We return the <code>HelloReply</code> to the client and then specify that we've finished dealing with the RPC.</li>
</ol>
  </div>
  <div id="cpp_service">
<p><a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/cpp/helloworld/greeter_server.cc">greeter_server.cc</a>
implements our <code>Greeter</code> service's required behaviour.</p>

<p>As you can see, the class <code>GreeterServiceImpl</code> implements the interface
<code>Greeter::Service</code> that we <a href="#generating">generated</a> from our proto
service definition by implementing the method <code>sayHello</code>:</p>
<pre>
class GreeterServiceImpl final : public Greeter::Service {
  Status SayHello(ServerContext* context, const HelloRequest* request,
                  HelloReply* reply) override {
    std::string prefix("Hello ");
    reply->set_message(prefix + request->name());
    return Status::OK;
  }
};
</pre>

<p>In this case we're implementing the synchronous version of <code>Greeter</code>, which provides our default gRPC server behaviour (there's also an asynchronous interface, <code>Greeter::AsyncService</code>).
<ul>
<li><code>sayHello</code> takes three parameters:

<ul>
<li><code>ServerContext</code>: a context object for this RPC.
<li><code>HelloRequest</code>: the request</li>
<li><code>HelloReply</code>: the response</li>
</ul></li>
</ul>

<p>To return our response to the client and complete the call:</p>

<ol>
<li>We populate the provided <code>HelloReply</code> response object with our exciting
message, as specified in our interface definition.</li>
<li>We return <code>Status::OK</code> to specify that we've finished dealing with the RPC.</li>
</ol>
  </div>
  <div id="python_service">
<p><a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/python/helloworld/greeter_server.py">greeter_server.py</a> implements our <code>Greeter</code> service's required behaviour.

<p>As you can see, the class <code>Greeter</code> implements the interface
<code>helloworld_pb2.BetaGreeterServicer</code> that we <a href="#generating">generated</a> from our proto
service definition by implementing the method <code>SayHello</code>:</p>
<pre>class Greeter(helloworld_pb2.BetaGreeterServicer):

  def SayHello(self, request, context):
    return helloworld_pb2.HelloReply(message='Hello, %s!' % request.name)
</pre>
<p>To return our response to the client and complete the call:
<ul>
<li>We construct and populate a <code>HelloReply</code> response object with our exciting
message, as specified in our interface definition.</li>
<li>We return the <code>HelloReply</code> to the client.</li>
</ul>
</div>
  <div id="go_service">
<p><a href="https://github.com/grpc/grpc-go/blob/master/examples/helloworld/greeter_server/main.go">greeter_server/main.go</a> implements our <code>Greeter</code> service's required behaviour.
<p>As you can see, our server has a <code>server</code> struct type. This implements the <code>GreeterServer</code> interface that we <a href="#generating">generated</a> from our proto
service definition by implementing the method <code>SayHello</code>:
<pre>// server is used to implement helloworld.GreeterServer.
type server struct{}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	return &pb.HelloReply{Message: "Hello " + in.Name}, nil
}
</pre>
<p>To return our response to the client and complete the call:
<ul>
<li>We construct and populate a <code>HelloReply</code> response object with our exciting
message, as specified in our interface definition.</li>
<li>We return the <code>HelloReply</code> to the client.</li>
</ul>

  </div>
  <div id="ruby_service">
<p><a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/ruby/greeter_server.rb">greeter&lowbar;server.rb</a> implements our <code>Greeter</code> service's required behaviour.
<p>Our server has a <code>GreeterServer</code> class, which implements the <code>GreeterServer</code> interface that we <a href="#generating">generated</a> from our proto
service definition by implementing the method <code>SayHello</code>:</p>
<pre>class GreeterServer < Helloworld::Greeter::Service
  # say_hello implements the SayHello rpc method.
  def say_hello(hello_req, _unused_call)
    Helloworld::HelloReply.new(message: "Hello #{hello_req.name}")
  end
</pre>
<p>To return our response to the client and complete the call, we construct and populate a <code>HelloReply</code> response object with our exciting
message, as specified in our interface definition, then return.</p>

  </div>
  <div id="node_service">
<p><a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/node/greeter_server.js">greeter&lowbar;server.js</a> implements our <code>Greeter</code> service's required behaviour.
<p>Our server implements the <code>Greeter</code>service from our
service definition by implementing the method <code>SayHello</code>:</p>
<pre>
function sayHello(call, callback) {
  callback(null, {message: 'Hello ' + call.request.name});
}
</pre>
<p>To return our response to the client and complete the call, we populate our response and pass it to the provided callback, with a null first parameter to indicate that there is no error.</p>

  </div>
  <div id="csharp_service">
<p><a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/csharp/helloworld/GreeterServer/Program.cs">GreeterServer/Program.cs</a> implements our <code>Greeter</code> service's required behaviour.
<p>Our server has a <code>GreeterImpl</code> class, which implements the <code>IGreeter</code> interface that we <a href="#generating">generated</a> from our proto
service definition by implementing the method <code>SayHello</code>:</p>
<pre>
class GreeterImpl : Greeter.IGreeter
{
    public Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        return Task.FromResult(new HelloReply { Message = "Hello " + request.Name });
    }
}
</pre>

<p>To return our response to the client and complete the call:</p>
<ul>
<li>We construct and populate a <code>HelloReply</code> response object with our exciting
message, as specified in our interface definition.</li>
<li>We return the <code>HelloReply</code> to the client.</li>
</ul>

</div>
</div>


#### Server implementation

The other main feature required to provide a gRPC service is making the service
implementation available from the network.

<div class="tabs">
  <ul>
    <li><a href="#java_server">Java</a></li>
    <li><a href="#cpp_server">C++</a></li>
    <li><a href="#python_server">Python</a></li>
    <li><a href="#go_server">Go</a></li>
    <li><a href="#ruby_server">Ruby</a></li>
    <li><a href="#node_server">Node.js</a></li>
    <li><a href="#csharp_server">C#</a></li>
  </ul>
  <div id="java_server">
<p><a href="https://github.com/grpc/grpc-java/blob/master/examples/src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java">HelloWorldServer.java</a>
provides this for our Java example.</p>
<pre>/* The port on which the server should run */
private int port = 50051;
private Server server;

private void start() throws Exception {
  server = ServerBuilder.forPort(port)
      .addService(GreeterGrpc.bindService(new GreeterImpl()))
      .build()
      .start();
  logger.info("Server started, listening on " + port);
  Runtime.getRuntime().addShutdownHook(new Thread() {
    @Override
    public void run() {
      // Use stderr here since the logger may has been reset by its JVM shutdown hook.
      System.err.println("*** shutting down gRPC server since JVM is shutting down");
      HelloWorldServer.this.stop();
      System.err.println("*** server shut down");
    }
  });
}</pre>

  </div>
  <div id="cpp_server">
<p><a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/cpp/helloworld/greeter_server.cc">greeter_server.cc</a>
also provides this for our C++ example.</p>
<pre>void RunServer() {
  std::string server_address("0.0.0.0:50051");
  GreeterServiceImpl service;

  ServerBuilder builder;
  builder.AddListeningPort(server_address, grpc::InsecureServerCredentials());
  builder.RegisterService(&service);
  std::unique_ptr<Server> server(builder.BuildAndStart());
  std::cout << "Server listening on " << server_address << std::endl;
  server->Wait();
}

</pre>
  </div>
  <div id="python_server">
<p><a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/python/helloworld/greeter_server.py">greeter_server.py</a>
also provides this for our Python example.</p>
<pre>  server = helloworld_pb2.beta_create_Greeter_server(Greeter())
  server.add_insecure_port('[::]:50051')
  server.start()
  try:
    while True:
      time.sleep(_ONE_DAY_IN_SECONDS)
  except KeyboardInterrupt:
    server.stop()
</pre>
</div>
  <div id="go_server">
<p><a href="https://github.com/grpc/grpc-go/blob/master/examples/helloworld/greeter_server/main.go">greeter_server/main.go</a> also provides this for our Go example.
<pre>
const (
	port = ":50051"
)
...

func main() {
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	pb.RegisterGreeterServer(s, &server{})
	s.Serve(lis)
}
</pre>

  </div>
  <div id="ruby_server">
<p><a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/ruby/greeter_server.rb">greeter&lowbar;server.rb</a> also provides this for our Ruby example.
<pre>
def main
  s = GRPC::RpcServer.new
  s.add&lowbar;http2&lowbar;port('0.0.0.0:50051')
  s.handle(GreeterServer)
  s.run
end
</pre>

  </div>
  <div id="node_server">
<p><a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/node/greeter_server.js">greeter&lowbar;server.js</a> also provides this for our Node.js example.
<pre>
function main() {
  var server = new Server({
    "helloworld.Greeter": {
      sayHello: sayHello
    }
  });
  server.bind('0.0.0.0:50051');
  server.listen();
}
</pre>

  </div>
  <div id="csharp_server">
<p><a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/csharp/helloworld/GreeterServer/Program.cs">GreeterServer/Program.cs</a> also provides this for our C# example.
<pre>Server server = new Server
{
    Services = { Greeter.BindService(new GreeterImpl()) },
    Ports = { new ServerPort("localhost", 50051, ServerCredentials.Insecure) }
};
server.Start();
</pre>
  </div>
</div>

Here we create an appropriate gRPC server, binding the `Greeter` service
implementation that we created to a port. Then we start the server running: the server is now ready to receive
requests from `Greeter` service clients on our specified port. We'll cover
how all this works in a bit more detail in our language-specific documentation.

<!--=================================-->
<a name="client"></a>
### Writing a client

Client-side gRPC is pretty simple. In this step, we'll use the generated code
to write a simple client that can access the `Greeter` server we created
in the [previous section](#server).

Again, we're not going to go into much detail about how to implement a client;
we'll leave that for the tutorial.

#### Connecting to the service

First let's look at how we connect to the `Greeter` server. First we need
to create a gRPC channel, specifying the hostname and port of the server we
want to connect to. Then we use the channel to construct the stub instance.


<div class="tabs">
  <ul>
    <li><a href="#java_connect">Java</a></li>
    <li><a href="#cpp_connect">C++</a></li>
    <li><a href="#python_connect">Python</a></li>
    <li><a href="#go_connect">Go</a></li>
    <li><a href="#ruby_connect">Ruby</a></li>
    <li><a href="#node_connect">Node.js</a></li>
    <li><a href="#csharp_connect">C#</a></li>
    <li><a href="#objective-c_connect">Objective-C</a></li>
    <li><a href="#php_connect">PHP</a></li>
  </ul>
  <div id="java_connect">
<pre>
private final ManagedChannel channel;
private final GreeterGrpc.GreeterBlockingStub blockingStub;

public HelloWorldClient(String host, int port) {
  channel = ManagedChannelBuilder.forAddress(host, port)
      .usePlaintext(true)
      .build();
  blockingStub = GreeterGrpc.newBlockingStub(channel);
}
</pre>

<p>In this case, we create a blocking stub. This means that the RPC call waits
for the server to respond, and will either return a response or raise an
exception. gRPC Java has other kinds of stubs that make non-blocking calls
to the server, where the response is returned asynchronously.
  </div>
  <div id="cpp_connect">
<pre>int main(int argc, char** argv) {
  GreeterClient greeter(
      grpc::CreateChannel("localhost:50051", grpc::InsecureCredentials(),
                          ChannelArguments()));
...
}

...

class GreeterClient {
 public:
  GreeterClient(std::shared_ptr<ChannelInterface> channel)
      : stub_(Greeter::NewStub(channel)) {}
...
 private:
  std::unique&lowbar;ptr<Greeter::Stub> stub_;
};

  </div>
  <div id="python_connect">
<p>The generated Python code has a helper function that creates a stub given a channel.
<pre>channel = implementations.insecure_channel('localhost', 50051)
stub = helloworld_pb2.beta_create_Greeter_stub(channel)
...
</pre>
  </div>
  <div id="go_connect">
<pre>const (
	address     = "localhost:50051"
	defaultName = "world"
)

func main() {
	// Set up a connection to the server.
	conn, err := grpc.Dial(address)
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := pb.NewGreeterClient(conn)
...
}</pre>

<p>In gRPC Go you use a special <code>Dial()</code> function to create the channel.</p>

  </div>
  <div id="ruby_connect">
<pre>stub = Helloworld::Greeter::Stub.new('localhost:50051')</pre>
<p>In Ruby, we can do this in a single method call using the <code>Stub</code> class generated from our .proto.</p>
  </div>
  <div id="node_connect">
<pre>var client = new hello_proto.Greeter('localhost:50051');</pre>
<p>In Node.js, we can do this in a single step by calling the <code>Greeter</code> stub constructor.</p>

  </div>
  <div id="csharp_connect">
<pre>Channel channel = new Channel("127.0.0.1:50051", Credentials.Insecure);
var client = Greeter.NewClient(channel);
...
</pre>
  </div>
  <div id="objective-c_connect">

```
#import <HelloWorld/Helloworld.pbrpc.h>

static NSString * const kHostAddress = @"http://localhost:50051";
HLWGreeter *client = [[HLWGreeter alloc] initWithHost:kHostAddress];
```

In Objective-C, we can do this in a single step using our generated `HLWGreeter` class's designated initializer, which expects a `NSString *` with the server address and port.

  </div>
  <div id="php_connect">

```
$client = new helloworld\GreeterClient(
      new Grpc\BaseStub('localhost:50051', []));
```
In PHP, we can do this in a single step using the `GreeterClient` class's constructor.

  </div>
</div>


#### Calling an RPC

Now we can contact the service and obtain a greeting:

1. We construct and fill in a `HelloRequest` to send to the service.
2. We call the stub's `SayHello()` RPC with our request and get a populated `HelloReply` if the RPC is successful, from which we can get our greeting.

<div class="tabs">
  <ul>
    <li><a href="#java_call">Java</a></li>
    <li><a href="#cpp_call">C++</a></li>
    <li><a href="#python_call">Python</a></li>
    <li><a href="#go_call">Go</a></li>
    <li><a href="#ruby_call">Ruby</a></li>
    <li><a href="#node_call">Node.js</a></li>
    <li><a href="#csharp_call">C#</a></li>
    <li><a href="#objective-c_call">Objective-C</a></li>
    <li><a href="#php_call">PHP</a></li>
  </ul>
  <div id="java_call">
<pre>
HelloRequest req = HelloRequest.newBuilder().setName(name).build();
HelloReply reply = blockingStub.sayHello(req);
</pre>

<p>You can see the complete client code in
<a href="https://github.com/grpc/grpc-java/blob/master/examples/src/main/java/io/grpc/examples/helloworld/HelloWorldClient.java">HelloWorldClient.java</a>.
</div>
  <div id="cpp_call">
<pre>  std::string SayHello(const std::string& user) {
    HelloRequest request;
    request.set&lowbar;name(user);
    HelloReply reply;
    ClientContext context;
    Status status = stub_->SayHello(&context, request, &reply);
    if (status.ok()) {
      return reply.message();
    } else {
      return "Rpc failed";
    }
  }</pre>

<p>You can see the complete client code in <a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/cpp/helloworld/greeter_client.cc">greeter_client.cc</a>.</p>
  </div>
  <div id="python_call">
<pre>response = stub.SayHello(helloworld_pb2.HelloRequest(name='you'), _TIMEOUT_SECONDS)
print "Greeter client received: " + response.message
</pre>
<p>You can see the complete client code in <a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/python/helloworld/greeter_client.py">greeter_client.py</a>.</p>

</div>
  <div id="go_call">
<pre>r, err := c.SayHello(context.Background(), &pb.HelloRequest{Name: name})
if err != nil {
		log.Fatalf("could not greet: %v", err)
}
log.Printf("Greeting: %s", r.Message)</pre>
<p>You can see the complete client code in <a href="https://github.com/grpc/grpc-go/blob/master/examples/helloworld/greeter_client/main.go">greeter_client/main.go</a>.</p>

  </div>
  <div id="ruby_call">
<pre>
  message = stub.say&lowbar;hello(Helloworld::HelloRequest.new(name: user)).message
  p "Greeting: #{message}"
</pre>
<p>You can see the complete client code in <a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/ruby/greeter_client.rb">greeter_client.rb</a>.</p>

  </div>
  <div id="node_call">
<pre>  client.sayHello({name: user}, function(err, response) {
    console.log('Greeting:', response.message);
  });</pre>
<p>You can see the complete client code in <a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/node/greeter_client.js">greeter_client.js</a>.</p>

  </div>
  <div id="csharp_call">
<pre>var reply = client.SayHello(new HelloRequest { Name = user });
Console.WriteLine("Greeting: " + reply.Message);</pre>
<p>You can see the complete example code in <a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/csharp/helloworld/GreeterClient/Program.cs">GreeterClient/Program.cs</a>.</p>

  </div>
  <div id="objective-c_call">
<pre>    HLWHelloRequest *request = [HLWHelloRequest message];
    request.name = @"Objective-C";
    [client sayHelloWithRequest:request handler:^(HLWHelloReply *response, NSError *error) {
      NSLog(@"%@", response.message);
    }];</pre>
<p>You can see the complete example code in <a href="https://github.com/grpc/grpc/tree/{{ site.data.config.grpc_release_branch }}/examples/objective-c/helloworld">examples/objective-c/helloworld</a>.</p>

  </div>
  <div id="php_call">
<pre>  $request = new helloworld\HelloRequest();
  $request->setName($name);
  list($reply, $status) = $client->SayHello($request)->wait();
  $message = $reply->getMessage();</pre>

<p>You can see the complete client code in <a href="https://github.com/grpc/grpc/blob/{{ site.data.config.grpc_release_branch }}/examples/php/greeter_client.php">greeter_client.php</a>.</p>

  </div>
</div>

<!--=================================-->
<a name="run"></a>
### Try it out!

You can try building and running our example using the same language on both the client and server side. Or you can try out one of gRPC's most useful features - interoperability
between code in different languages - and run a server and client in different languages. Each service
and client uses interface code generated from the same proto, which means that any `Greeter` client can talk to any `Greeter` server.

First run the server:

<div class="tabs">
  <ul>
    <li><a href="#java_runserver">Java</a></li>
    <li><a href="#cpp_runserver">C++</a></li>
    <li><a href="#python_runserver">Python</a></li>
    <li><a href="#go_runserver">Go</a></li>
    <li><a href="#ruby_runserver">Ruby</a></li>
    <li><a href="#node_runserver">Node.js</a></li>
    <li><a href="#csharp_runserver">C#</a></li>
  </ul>
  <div id="java_runserver">
<p>You can build and run the server from the <code>examples</code> folder. First
build the client and server.

<pre>
$ ../gradlew -PskipCodegen=true installDist
</pre>

Then run the server, which will listen on port 50051:
<pre>
$ ./build/install/grpc-examples/bin/hello-world-server
</pre>

  </div>
  <div id="cpp_runserver">
<p>You can build and run the server from the <code>examples/cpp/helloworld</code> folder. First build the client and server.
<pre>$ make</pre>
Then run the server, which will listen on port 50051:
<pre>$ ./greeter_server</pre>
  </div>
  <div id="python_runserver">
<p>You can run the server from <code>examples/python/helloworld</code> using:
<pre>$ ./run_server.sh</pre>

</div>
  <div id="go_runserver">
<p>You can run the server from <code>examples/helloworld</code> using:
<pre>$ greeter_server &</pre>

  </div>
  <div id="ruby_runserver">
<p>You can run the server from <code>examples/ruby</code> using:
<pre>$ bundle exec ./greeter_server.rb &</pre>

  </div>
  <div id="node_runserver">
<p>You can run the server from <code>examples/node</code> using:
<pre>$ node ./greeter_server.js &</pre>

  </div>
  <div id="csharp_runserver">
<p>Build the solution. Then from <code>examples/csharp</code>:

```
> cd GreeterServer/bin/Debug
> GreeterServer.exe
```

  </div>
</div>

Once the server is running, in another terminal window run the client and confirm that it receives a message.

<div class="tabs">
  <ul>
    <li><a href="#java_runclient">Java</a></li>
    <li><a href="#cpp_runclient">C++</a></li>
    <li><a href="#python_runclient">Python</a></li>
    <li><a href="#go_runclient">Go</a></li>
    <li><a href="#ruby_runclient">Ruby</a></li>
    <li><a href="#node_runclient">Node.js</a></li>
    <li><a href="#csharp_runclient">C#</a></li>
    <li><a href="#objective-c_runclient">Objective-C</a></li>
    <li><a href="#php_runclient">PHP</a></li>
  </ul>
  <div id="java_runclient">
<p>You can build and run the client from the <code>examples</code> folder. If
you haven't already built the client, build it using:

<pre>
$ ../gradlew -PskipCodegen=true installDist
</pre>

Then run the client:
<pre>
$ ./build/install/grpc-examples/bin/hello-world-client
</pre>
  </div>
  <div id="cpp_runclient">
<p>You can build and run the client from the <code>examples/cpp/helloworld</code> folder. If you haven't already built the client, build it using:
<pre>$ make</pre>
Then run the client:
<pre>$ ./greeter_client</pre>
  </div>
  <div id="python_runclient">
<p>You can run the client from <code>examples/python/helloworld</code> using:
<pre>$ ./run_client.sh</pre>

</div>
  <div id="go_runclient">
<p>You can run the client from <code>examples/helloworld</code> using:
<pre>$ greeter_client</pre>

  </div>
  <div id="ruby_runclient">
<p>You can run the client from <code>examples/ruby</code> using:
<pre>$ bundle exec ./greeter_client.rb</pre>

  </div>
  <div id="node_runclient">
<p>You can run the client from <code>examples/node</code> using:
<pre>$ node ./greeter_client.js</pre>

</div>
  <div id="csharp_runclient">
<p>Build the solution. Then from <code>examples/csharp</code>:

```
> cd GreeterClient/bin/Debug
> GreeterClient.exe
```

</div>
  <div id="objective-c_runclient">
<p>Open the XCode workspace created by Cocoapods, and run the app. You can see the results in XCode's log console.</p>

</div>
  <div id="php_runclient">
<p>You can run the client from <code>examples/php</code> using:

```
$ ./run_greeter_client.sh
```

</div>
</div>

<!--=========================================================================-->
## Read more!

- Find out how to install gRPC and get started in each language's [quick start](#quickstart).
- Follow the tutorial(s) for your favorite language(s).
- Discover more about [gRPC concepts](/docs/guides/concepts.html), including RPC life-cycle, synchronous and asynchronous calls, deadlines, and more.
- Read a detailed description of the [gRPC over HTTP2 protocol](/docs/guides/wire.html).
- [gRPC Authentication Support](/docs/guides/auth.html) introduces authentication support in gRPC with supported mechanisms and examples.
