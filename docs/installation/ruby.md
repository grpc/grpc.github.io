---
layout: docs
title: Ruby Quick Start
---

gRPC in 3 minutes (Ruby)
========================

This document will get you started with gRPC in Ruby, including a quick example.

Prerequisites
-------------

- **Ruby 2.x**: gRPC requires Ruby 2.x, as the gRPC API surface uses keyword args. If you don't have that installed locally, you can use [RVM][] to use Ruby 2.x for testing without upgrading the version of Ruby on your whole system. RVM is also useful if you don't have the necessary privileges to update your system's Ruby. Make sure you run `source $HOME/.rvm/scripts/rvm` as instructed to complete the set-up of RVM.

```sh
$ # RVM installation as specified at https://rvm.io/rvm/install
$ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
$ \curl -sSL https://get.rvm.io | bash -s stable --ruby=ruby-2
$
$ # follow the instructions to ensure that your're using the latest stable version of Ruby
$ # and that the rvm command is installed
```

Install
-------

- Clone the [grpc-common](https://github.com/grpc/grpc-common) repository, which contains all our examples.
- Use bundler to install the example package's dependencies

```sh
$ # from this directory
$ gem install bundler # if you don't already have bundler available
$ bundle install
```

You can find more detailed installation instructions in the [grpc repo](https://github.com/grpc/grpc/tree/master/src/ruby).

Try it!
-------

Now let's look at a Ruby implementation of the example used in the [Overview](/docs/index.shtml). For this example, we've already generated the server and client stubs from [helloworld.proto][].

- Run the server

```sh
$ # from this directory
$ bundle exec ./greeter_server.rb &
```

- Run the client

```sh
$ # from this directory
$ bundle exec ./greeter_client.rb
```

Tutorial
--------

You can find a more detailed tutorial in [gRPC Basics: Ruby](/docs/tutorials/basic/ruby.html)

[helloworld.proto]:https://github.com/grpc/grpc-common/blob/master/protos/helloworld.proto
[RVM]:https://www.rvm.io/
