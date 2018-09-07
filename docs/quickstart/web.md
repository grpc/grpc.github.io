---
bodyclass: docs
layout: docs
headline: Web Quickstart
sidenav: doc-side-quickstart-nav.html
type: markdown
---
<p class="lead">This guide gets you started with gRPC-Web with a simple
working example from the browser.</p>

<div id="toc"></div>

## Prerequisites

* `docker` and `docker-compose`

Please refer to [Docker website][] on how to install Docker.

## Run an Echo example from the browser!

```sh
$ git clone https://github.com/grpc/grpc-web
$ cd grpc-web
$ docker-compose pull prereqs common echo-server envoy commonjs-client
$ docker-compose up -d echo-server envoy commonjs-client
```

Open a browser tab, and go to:

```
http://localhost:8081/echotest.html
```

To shutdown, run `docker-compose down`.

## Install the gRPC-Web protoc plugin

In order to generate service stub from your protobuf defintions, you will need
the `protoc-gen-grpc-web` protoc plugin.

```sh
$ git clone https://github.com/grpc/grpc-web
$ cd grpc-web && sudo make install-plugin
```

## What's next

- Read a full explanation of how gRPC works in [What is gRPC?](../guides/)
  and [gRPC Concepts](../guides/concepts.html)
- Work through a more detailed tutorial in [gRPC Basics: Web][]

[Docker website]:https://docs.docker.com/compose/install/
[gRPC Basics: Web]:../tutorials/basic/web.html
