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

This demo requires Docker Compose file
[version 3](https://docs.docker.com/compose/compose-file/). Please refer to
[Docker website][] on how to install Docker.

### On Windows
- Install docker desktop and switch to linux containers
- [Login](https://docs.docker.com/engine/reference/commandline/login/) into docker before you run `docker-compose pull` 

## Run an Echo example from the browser!

```sh
$ git clone https://github.com/grpc/grpc-web
$ cd grpc-web
$ docker-compose pull
$ docker-compose up -d node-server envoy commonjs-client
```

Open a browser tab, and go to:

```
http://localhost:8081/echotest.html
```

To shutdown, run `docker-compose down`.


## What is Happening?

In this demo, there are three key components:

 1. `node-server`: This is a standard gRPC Server, implemented in Node.
 This server listens at port `:9090` and implements the service's business
 logic.
 
 2. `envoy`: This is the Envoy proxy. It listens at `:8080` and forwards the
 browser's gRPC-Web requests to port `:9090`. This is done via a config file
 `envoy.yaml`.
 
 3. `commonjs-client`: This component generates the client stub class using
 the `protoc-gen-grpc-web` protoc plugin, compiles all the JS dependencies
 using `webpack`, and hosts the static content `echotest.html` and
 `dist/main.js` using a simple web server at port `:8081`. Once the user
 interacts with the webpage, it sends a gRPC-Web request to the Envoy proxy
 endpoint at `:8080`.


## What's next

- Work through a more detailed tutorial in [gRPC Basics: Web][]

[Docker website]:https://docs.docker.com/compose/install/#install-compose
[gRPC Basics: Web]:../tutorials/basic/web.html
