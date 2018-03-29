---
bodyclass: docs
layout: docs
headline: gRPC Basics - Rust
sidenav: doc-side-tutorial-nav.html
type: markdown
---
<p class="lead">This tutorial provides a basic Rust programmer's introduction to
working with gRPC.</p>

By walking through this example you'll learn how to:

- Define a service in a .proto file.
- Generate server and client code using the protocol buffer compiler.
- Use the Rust gRPC API to write a simple client and server for your service.

It assumes that you have read the [Overview](/docs/index.html) and are familiar
with [protocol buffers](https://developers.google.com/protocol-buffers/docs/overview). Note that the
example in this tutorial uses the proto3 version of the protocol buffers
language, which is currently in beta release: you can find out more in the
[proto3 language
guide](https://developers.google.com/protocol-buffers/docs/proto3) and 
see the [release notes](https://github.com/google/protobuf/releases) for the
new version in the protocol buffers Github repository.

<div id="toc"></div>

## Why use gRPC?

Our example is a simple route mapping application that lets clients get
information about features on their route, create a summary of their route, and
exchange route information such as traffic updates with the server and other
clients.

With gRPC we can define our service once in a .proto file and implement clients
and servers in any of gRPC's supported languages, which in turn can be run in
environments ranging from servers inside Google to your own tablet - all the
complexity of communication between different languages and environments is
handled for you by gRPC. We also get all the advantages of working with protocol
buffers, including efficient serialization, a simple IDL, and easy interface
updating.

## Example code and setup

The example code for our tutorial is in
[grpc-rs/examples/route_guide](https://github.com/pingcap/grpc-rs/tree/master/examples/route_guide).
To download the example, clone the `grpc-rs` repository by running the following
command:

```
$ git clone https://github.com/pingcap/grpc-rs
```

Then change your current directory to `grpc-rs`:

```
$ cd grpc-rs
```

You also should have the relevant tools installed to generate the server and client interface code - if you don't already, follow the setup instructions in [the Rust quick start guide](/docs/quickstart/rust.html).

Notice: grpc-rs is based on Rust [futures](https://github.com/rust-lang-nursery/futures-rs), you must know the basic concepts like Future,
Stream, Sink and know how to use them at first.

## Defining the service

Our first step (as you'll know from the [Overview](/docs/index.html)) is to
define the gRPC *service* and the method *request* and *response* types using
[protocol buffers](https://developers.google.com/protocol-buffers/docs/overview). You can see the
complete .proto file in
[`grpc-rs/proto/proto/example/route_guide.proto`](https://github.com/pingcap/grpc-rs/blob/master/proto/proto/example/route_guide.proto).

To define a service, you specify a named `service` in your .proto file:

```proto
service RouteGuide {
   ...
}
```

Then you define `rpc` methods inside your service definition, specifying their
request and response types. gRPC lets you define four kinds of service method,
all of which are used in the `RouteGuide` service:

- A *simple RPC* where the client sends a request to the server using the stub
  and waits for a response to come back, just like a normal function call.

```proto
// Obtains the feature at a given position.
rpc GetFeature(Point) returns (Feature) {}
```

- A *server-side streaming RPC* where the client sends a request to the server
  and gets a stream to read a sequence of messages back. The client reads from
  the returned stream until there are no more messages. As you can see in our
  example, you specify a server-side streaming method by placing the `stream`
  keyword before the *response* type.

```proto
// Obtains the Features available within the given Rectangle.  Results are
// streamed rather than returned at once (e.g. in a response message with a
// repeated field), as the rectangle may cover a large area and contain a
// huge number of features.
rpc ListFeatures(Rectangle) returns (stream Feature) {}
```

- A *client-side streaming RPC* where the client writes a sequence of messages
  and sends them to the server, again using a provided stream. Once the client
  has finished writing the messages, it waits for the server to read them all
  and return its response. You specify a client-side streaming method by placing
  the `stream` keyword before the *request* type.

```proto
// Accepts a stream of Points on a route being traversed, returning a
// RouteSummary when traversal is completed.
rpc RecordRoute(stream Point) returns (RouteSummary) {}
```

- A *bidirectional streaming RPC* where both sides send a sequence of messages
  using a read-write stream. The two streams operate independently, so clients
  and servers can read and write in whatever order they like: for example, the
  server could wait to receive all the client messages before writing its
  responses, or it could alternately read a message then write a message, or
  some other combination of reads and writes. The order of messages in each
  stream is preserved. You specify this type of method by placing the `stream`
  keyword before both the request and the response.

```proto
// Accepts a stream of RouteNotes sent while a route is being traversed,
// while receiving other RouteNotes (e.g. from other users).
rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
```

Our .proto file also contains protocol buffer message type definitions for all the request and response types used in our service methods - for example, here's the `Point` message type:

```proto
// Points are represented as latitude-longitude pairs in the E7 representation
// (degrees multiplied by 10**7 and rounded to the nearest integer).
// Latitudes should be in the range +/- 90 degrees and longitude should be in
// the range +/- 180 degrees (inclusive).
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}
```

## Generating client and server code

Next we need to generate the gRPC client and server interfaces from our .proto
service definition. We do this using the protocol buffer compiler `protoc` with
a special gRPC Go plugin. 
This is similar to what we did in the [quickstart guide](../../quickstart/rust.html)

From the `grpc-rs` directory run:

```sh
 protoc --rust_out=examples/route_guide --grpc_out=examples/route_guide --plugin=protoc-gen-grpc=`which grpc_rust_plugin` proto/proto/example/route_guide.proto
```

Running this command generates `route_guide_grpc.rs` and `route_guide.rs` files under the `route_guide` example directory.

This contains:
- All the protocol buffer code to populate, serialize, and retrieve our request
  and response message types
- An interface type (or *stub*) for clients to call with the methods defined in
  the `RouteGuide` service.
- An interface type for servers to implement, also with the methods defined in
  the `RouteGuide` service.

Notice: In grpc-rs project, we generate all the files and encode them into a `.desc` file which Rust can use directly. For `route_guide`, it is in the [grpc-rs/proto/example.desc](https://github.com/pingcap/grpc-rs/blob/master/proto/example.desc).

<a name="server"></a>

## Creating the server

First let's look at how we create a `RouteGuide` server. If you're only
interested in creating gRPC clients, you can skip this section and go straight
to [Creating the client](#client) (though you might find it interesting
anyway!).

There are two parts to making our `RouteGuide` service do its job:

- Implementing the service interface generated from our service definition:
  doing the actual "work" of our service.
- Running a gRPC server to listen for requests from clients and dispatch them to
  the right service implementation.

You can find our example `RouteGuide` server in
[grpc-rs/examples/route_guide/server.rs](https://github.com/pingcap/grpc-rs/blob/master/examples/route_guide/server.rs).
Let's take a closer look at how it works.

### Implementing RouteGuide

As you can see, our server has a `RouteGuideService` struct type that implements the generated `RouteGuide` interface:

```rust
struct RouteGuideService {
  data: Arc<Vec<Feature>>,
}
...

fn get_feature(&self, ctx: RpcContext, point: Point, sink: UnarySink<Feature>) {
        ...
}
...

fn list_features(&self, ctx: RpcContext, rect: Rectangle, resp: ServerStreamingSink<Feature>) {
        ...
}
...

fn record_route(&self, ctx: RpcContext,points: RequestStream<Point>, resp: ClientStreamingSink<RouteSummary>) {
        ...
}
...

fn route_chat(&self, ctx: RpcContext, notes: RequestStream<RouteNote>, resp: DuplexSink<RouteNote>) {
        ...
}
...
```

#### Simple RPC
`RouteGuideService` implements all our service methods. Let's look at the
simplest type first, `get_feature`, which just gets a `Point` from the client and
returns the corresponding feature information from its database in a `Feature`.

```rust
fn get_feature(&self, ctx: RpcContext, point: Point, sink: UnarySink<Feature>) {
    let data = self.data.clone();
    let resp = data.iter()
        .find(|f| same_point(f.get_location(), &point))
        .map_or_else(Feature::new, ToOwned::to_owned);
    let f = sink.success(resp)
        .map_err(|e| error!("failed to handle getfeature request: {:?}", e));
    ctx.spawn(f)
}
```

The method is passed a context object for the RPC, the client's `Point`
protocol buffer request and a Sink with `Feature` protocol buffer oject. 
In the method we populate the `Feature` with the appropriate information, and 
then use the sink to return the `Feature` to the client.

#### Server-side streaming RPC

Now let's look at one of our streaming RPCs. `list_features` is a server-side
streaming RPC, so we need to send back multiple `Feature`s to our client.

```rust
fn list_features(&self, ctx: RpcContext, rect: Rectangle, resp: ServerStreamingSink<Feature>) {
    let data = self.data.clone();
    let features: Vec<_> = data.iter()
        .filter_map(move |f| {
            if fit_in(f.get_location(), &rect) {
                Some((f.to_owned(), WriteFlags::default()))
            } else {
                None
            }
        })
        .collect();
    let f = resp.send_all(stream::iter_ok::<_, Error>(features))
        .map(|_| {})
        .map_err(|e| error!("failed to handle listfeatures request: {:?}", e));
    ctx.spawn(f)
}
```

As you can see, instead of getting simple request and response objects in our
method parameters, this time we get a request object (the `Rectangle` in which
our client wants to find `Feature`s) and a special
`ServerStreamingSink` object to write our responses.

In the method, we populate as many `Feature` objects as we need to return,
writing them to the `ServerStreamingSink` using its `send_all()` method.
Should any error happen in this call, the `ServerStreamingSink` returns an error; 
the gRPC layer will translate it into an appropriate RPC status to be sent on the wire.

#### Client-side streaming RPC

Now let's look at something a little more complicated: the client-side
streaming method `record_route`, where we get a stream of `Point`s from the
client and return a single `RouteSummary` with information about their trip. As
you can see, this time the method doesn't have a request parameter at all.
Instead, it gets a `RequestStream` stream, which the server can
use to read messages, and gets a `ClientStreamingSink` sink, which the server can
return its single response.

```rust
fn record_route(&self, ctx: RpcContext, points: RequestStream<Point>, resp: ClientStreamingSink<RouteSummary>,
) {
    let data = self.data.clone();
    let timer = Instant::now();
    let f = points
        .fold(
            (None, 0f64, RouteSummary::new()),
            move |(last, mut dis, mut summary), point| {
                let total_count = summary.get_point_count();
                summary.set_point_count(total_count + 1);
                let valid_point = data.iter()
                    .any(|f| !f.get_name().is_empty() && same_point(f.get_location(), &point));
                if valid_point {
                    let feature_count = summary.get_feature_count();
                    summary.set_feature_count(feature_count + 1);
                }
                if let Some(last_point) = last {
                    dis += cal_distance(&last_point, &point);
                }
                Ok((Some(point), dis, summary)) as Result<_>
            },
        )
        .and_then(move |(_, dis, mut s)| {
            s.set_distance(dis as i32);
            let dur = timer.elapsed();
            s.set_elapsed_time(dur.as_secs() as i32);
            resp.success(s)
        })
        .map_err(|e| error!("failed to record route: {:?}", e));
    ctx.spawn(f)
}
```

In the method body we use the `RequestStream`'s `fold()` method to
repeatedly read in our client's requests to a request object (in this case a
`Point`) until there are no more messages: the server needs to check the
error when consumes the stream. After the stream is finished, the server can return its `RouteSummary`. 
Any error be translated to an RPC status by the gRPC layer.

#### Bidirectional streaming RPC
Finally, let's look at our bidirectional streaming RPC `route_chat`.

```rust
fn route_chat(&self, ctx: RpcContext, notes: RequestStream<RouteNote>, resp: DuplexSink<RouteNote>,
) {
    let mut buffer: Vec<RouteNote> = Vec::new();
    let to_send = notes
        .map(move |note| {
            let to_prints: Vec<_> = buffer
                .iter()
                .filter_map(|n| {
                    if same_point(n.get_location(), note.get_location()) {
                        Some((n.to_owned(), WriteFlags::default()))
                    } else {
                        None
                    }
                })
                .collect();
            buffer.push(note);
            stream::iter_ok::<_, Error>(to_prints)
        })
        .flatten();
    let f = resp.send_all(to_send)
        .map(|_| {})
        .map_err(|e| error!("failed to route chat: {:?}", e));
    ctx.spawn(f)
}
```

This time we get a `RequestStream` stream that, as in our
client-side streaming example, can be used to read messages, 
and a `DuplexSink`, can be used to write messages. 

The syntax for reading and writing here is very similar to our client-streaming
method, except the server uses the sink's `send_allSend()` method rather than
`success()` because it's writing multiple responses. Although each side
will always get the other's messages in the order they were written, both the
client and server can read and write in any order — the streams operate
completely independently.

### Starting the server

Once we've implemented all our methods, we also need to start up a gRPC server
so that clients can actually use our service. The following snippet shows how we
do this for our `RouteGuide` service:

```rust
let env = Arc::new(Environment::new(2));
let instance = RouteGuideService {
    data: Arc::new(load_db()),
};
let service = route_guide_grpc::create_route_guide(instance);
let mut server = ServerBuilder::new(env)
    .register_service(service)
    .bind("127.0.0.1", 50051)
    .build()
    .unwrap();
server.start();
```

To build and start a server, we:

1. Create a `Environment`.
2. Cretae a `RouteGuideService` and pass to the gRPC service with `route_guide_grpc::create_route_guide`.
3. Create the server with the Environment, register the service to the server, bind the listening host and port, then build the server.
4. Start the server.

<a name="client"></a>

## Creating the client

In this section, we'll look at creating a Rust client for our `RouteGuide`
service. You can see our complete example client code in
[grpc-rs/examples/route_guide/client.rs](https://github.com/pingcap/grpc-rs/blob/master/examples/route_guide/client.rs).

### Creating a stub

To call service methods, we first need to create a gRPC *channel* to communicate
with the server. We create this as follows:

```rust
let env = Arc::new(Environment::new(2));
let channel = ChannelBuilder::new(env).connect("127.0.0.1:50051");
```

We first create an Environment then use a ChannelBuilder to create a channel with server address.

Once the gRPC *channel* is setup, we need a client *stub* to perform RPCs. We
get this using the `RouteGuideClient::new` method provided in the `route_guide_grpc.rs` file we
generated from our .proto.

```rust
let client = RouteGuideClient::new(channel);
```

### Calling service methods

Now let's look at how we call our service methods. 

#### Simple RPC

Calling the simple RPC `get_feature` is nearly as straightforward as calling a
local method.

```rust
let get_feature = client.get_feature_async(&new_point(409_146_138, -746_188_906)).unwrap();
match get_feature.wait() {
        ...
}
```

As you can see, we call the method on the stub we got earlier. In our method
parameters we create and populate a request protocol buffer object (in our case
`Point`). This method returns a Future, we wait the Future finished and get the response.


#### Server-side streaming RPC

Here's where we call the server-side streaming method `list_features`, which
returns a stream of geographical `Feature`s. If you've already read [Creating
the server](#server) some of this may look very familiar - streaming RPCs are
implemented in a similar way on both sides.

```rust
let rect = new_rect(400_000_000, -750_000_000, 420_000_000, -730_000_000);
info!("Looking for features between 40, -75 and 42, -73");
let mut list_features = client.list_features(&rect).unwrap();
loop {
    let f = list_features.into_future();
    match f.wait() {
        Ok((Some(feature), s)) => {
            list_features = s;
            let loc = feature.get_location();
            info!(
                "Found feature {} at {}",
                feature.get_name(),
                util::format_point(loc)
            );
        }
        Ok((None, _)) => break,
        Err((e, _)) => panic!("List features failed: {:?}", e),
    }
}
```

As in the simple RPC, we pass the method a request. However,
instead of getting a Future object back, we get back an instance of
`ClientSStreamReceiver`. The client can use the
`ClientSStreamReceiver` stream to read the server's responses.

We convert the stream into future and wait the response repeatedly from the server.
If the stream is finished, we will get a None and break the loop.

#### Client-side streaming RPC

The client-side streaming method `record_route` returns a `ClientCStreamSender` sink to send messages to the server
and a `ClientCStreamReceiver` future to receive message from the server.

```rust
let features = util::load_db();
let mut rng = rand::thread_rng();
let (mut sink, receiver) = client.record_route().unwrap();
for _ in 0..10 {
    let f = rng.choose(&features).unwrap();
    let point = f.get_location();
    info!("Visiting {}", util::format_point(point));
    sink = sink.send((point.to_owned(), WriteFlags::default()))
        .wait()
        .unwrap();
    thread::sleep(Duration::from_millis(rng.gen_range(500, 1500)));
}
// flush
future::poll_fn(|| sink.close()).wait().unwrap();
let sumary = receiver.wait().unwrap();
```

The `ClientCStreamSender` has a `send()` method that we can use to send
requests to the server. Once we've finished writing our client's requests to the
sink using `send()`, we need to call `close()` on the sink to let
gRPC know that we've finished writing and are expecting to receive a response.
THen we use future wait to get the response. 

#### Bidirectional streaming RPC

Finally, let's look at our bidirectional streaming RPC `route_chat()`. It returns a 
`ClientDuplexSender` to send messages to the server and a `ClientDuplexReceiver` to receive 
messages from the server. 

```rust
let (mut sink, mut receiver) = client.route_chat().unwrap();
let h = thread::spawn(move || {
    let notes = vec![
        ("First message", 0, 0),
        ("Second message", 0, 1),
        ("Third message", 1, 0),
        ("Fourth message", 0, 0),
    ];

    for (msg, lat, lon) in notes {
        let note = new_note(lat, lon, msg);
        info!("Sending message {} at {}, {}", msg, lat, lon);
        sink = sink.send((note, WriteFlags::default())).wait().unwrap();
    }
    future::poll_fn(|| sink.close()).wait().unwrap();
});

loop {
    match receiver.into_future().wait() {
        Ok((Some(note), r)) => {
            let location = note.get_location();
            info!(
                "Got message {} at {}, {}",
                note.get_message(),
                location.get_latitude(),
                location.get_longitude()
            );
            receiver = r;
        }
        Ok((None, _)) => break,
        Err((e, _)) => panic!("RouteChat RPC failed: {:?}", e),
    }
}

h.join().unwrap();
```

The syntax for reading here is very similar to our server-side stream method and 
for writing here is very similar to our client-side sink method.
Although each side will always get the other's messages in
the order they were written, both the client and server can read and write in
any order — the streams operate completely independently.

## Try it out!

To compile and run the server, assuming you are in the folder
`grpc-rs`, simply:

```sh
$ cargo run --example route_guide_server 
```

Likewise, to run the client:

```sh
$ cargo run --example route_guide_client
```
