---
bodyclass: docs
headline: What is gRPC?
layout: docs
sidenav: doc-side-guides-nav.html
title: Guides
type: markdown
---
<p class="lead">
This document introduces you to gRPC (and its friend, protocol buffers).  If you’re new to gRPC and/or protocol buffers, read this! If you just want to dive in and see gRPC in action first, see our [Quick Starts].
</p>

<div id="toc" class="toc mobile-toc"></div>

### Overview
In gRPC a client application can directly call methods on a server application on a different machine as if it was a local object, making it easier for you to create distributed applications and services. As in many RPC systems, gRPC is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. On the server side, the server implements this interface and runs a gRPC server to handle client calls. On the client side, the client has a stub (referred to as just a client in some languages) that provides the same methods as the server.

![Concept Diagram]({{site.baseurl}}/img/landing-2.svg)

gRPC clients and servers can run and talk to each other in a variety of environments - from servers inside Google to your own desktop - and can be written in any of gRPC's supported languages. So, for example, you can easily create a gRPC server in Java with clients in Go, Python, or Ruby. In addition, the latest Google APIs will have gRPC versions of their interfaces, letting you easily build Google functionality into your applications.

### Working with Protocol Buffers
By default gRPC uses protocol buffers, Google’s mature open source mechanism for serializing structured data (although it can be used with other data formats such as JSON). As you'll see in our examples, you define gRPC services using proto files, with method parameters and return types specified as protocol buffer message types. You can find out lots more about protocol buffers in the Protocol Buffers documentation.


### Protocol buffer versions
While protocol buffers have been available for open source users for some time, our examples use a new flavor of protocol buffers called proto3, which has a slightly simplified syntax, some useful new features, and supports lots more languages. This is currently available as a beta release in Java, C++, Python, Objective-C, and C#, with an alpha release for JavaNano (Android Java), Ruby, and JavaScript fromthe protocol buffers Github repo, as well as a Go language generator from the golang/protobuf Github repo, with more languages in development. You can find out more in the proto3 language guide and the reference documentation for each language (where available), and see the major differences from the current default version in the release notes. Even more proto3 documentation is coming soon.

In general, while you can use proto2 (the current default protocol buffers version), we recommend that you use proto3 with gRPC as it lets you use the full range of gRPC-supported languages, as well as avoiding compatibility issues with proto2 clients talking to proto3 servers and vice versa.
