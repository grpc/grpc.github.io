---
layout: post
title: Let's gracefully clean up in gRPC JUnit tests
published: false
permalink: blog/gracefully_clean_up_in_grpc_junit_tests
author: Dapeng Zhang
author-link: https://github.com/dapengzhang0
company: Google
company-link: https://www.google.com
---

As a good programmer, it's important that you remember to always clean up gRPC resources such as client channels, servers, and previously attached Contexts whenever they are no longer needed. This is even true for JUnit tests, because otherwise leaked resource will not only linger in your machine forever in some cases but also interfere with subsequent tests. A not-so-bad case is that subsequent tests can not pass because of the leaked resource in the previous test. The worst case is that some subsequent tests pass but in fact they wouldn't have passed if the previously passed test had not leaked resource.

<!--more-->

So cleanup, cleanup, cleanup...\
and fail the test if any cleanup is not successful.

A typical example is

```java
public class MyTest {
  private Server server;
  private ManagedChannel channel;
  ...
  @After
  public void tearDown() InterruptedException {
    // assume channel and server are not null
    channel.shutdownNow();
    server.shutdownNow();
    // fail the test if cleanup is not successful
    assert channel.awaitTermination(5, TimeUnit.SECONDS) : "channel failed to shutdown";
    assert server.awaitTermination(5, TimeUnit.SECONDS) : "server failed to shutdown";
  }
  ...
}
```

or to be more graceful

```java
public class MyTest {
  private Server server;
  private ManagedChannel channel;
  ...
  @After
  public void tearDown() throws InterruptedException {
    // assume channel and server are not null
    channel.shutdown();
    server.shutdown();
    // fail the test if cannot gracefully shutdown
    try {
      assert channel.awaitTermination(5, TimeUnit.SECONDS) : "channel cannot be gracefully shutdown";
      assert server.awaitTermination(5, TimeUnit.SECONDS) : "server cannot be gracefully shutdown";
    } finally {
      channel.shutdownNow();
      server.shutdownNow();
    }
  }
  ...
}
```

But that's tedious. You need to write the shutdown boilerplate by yourself. So gRPC testing library introduced something to make it less tedious.

A JUnit rule [`GrpcServerRule`][GrpcServerRule] was first introduced to eliminate the shutdown boilerplate. With this rule, it creates an In-Process server and channel at the beginning of the test and shuts them down at the end of test automatically. But it is inflexible and too restrictive in the sense that it does not support transports other than In-Process transports, multiple channels to the server, custom channel or server builder options, and configuration inside individual test methods.

A more preferable JUnit rule [`GrpcCleanupRule`][GrpcCleanupRule] was introduced (and will be available from gRPC release v1.13.0), which also eliminates the shutdown boilerplate. Unlike `GrpcServerRule`, `GrpcCleanupRule` does not create any server or channel automatically at all. Users should create and start the server by themselves, and create channels by themselves, just as in plain tests. With this rule, users just need to register every resource (channel or server) that needs to be shut down at the end of test, and the rule will then shut them down gracefully automatically.

You can register resources either before running test methods

```java
public class MyTest {
  @Rule
  public GrpcCleanupRule grpcCleanup = new GrpcCleanupRule();
  ...
  private String serverName = InProcessServerBuilder.generateName();
  private Server server = grpcCleanup.register(InProcessServerBuilder
      .forName(serverName).directExecutor().addService(myServiceImpl).build().start());
  private ManagedChannel channel = grpcCleanup.register(InProcessChannelBuilder
      .forName(serverName).directExecutor().build());
  ...
}
```

or inside each individual test method

```java
public class MyTest {
  @Rule
  public GrpcCleanupRule grpcCleanup = new GrpcCleanupRule();
  ...
  private String serverName = InProcessServerBuilder.generateName();
  private InProcessServerBuilder serverBuilder = InProcessServerBuilder
      .forName(serverName).directExecutor();
  private InProcessChannelBuilder channelBuilder = InProcessChannelBuilder
      .forName(serverName).directExecutor();
  ...

  @Test
  public void testFooBar() {
    ...
    grpcCleanup.register(
    	serverBuilder.addService(myServiceImpl).build().start());
    ManagedChannel channel = grpcCleanup.register(
    	channelBuilder.maxInboundMessageSize(1024).build());
    ...
  }
}
```

Now with [`GrpcCleanupRule`][GrpcCleanupRule] you don't need to worry about graceful shutdown of gRPC servers and channels in JUnit test. So try it out and clean up in your tests!

[GrpcServerRule]:https://github.com/grpc/grpc-java/blob/v1.1.x/testing/src/main/java/io/grpc/testing/GrpcServerRule.java
[GrpcCleanupRule]:https://github.com/grpc/grpc-java/blob/master/testing/src/main/java/io/grpc/testing/GrpcCleanupRule.java
