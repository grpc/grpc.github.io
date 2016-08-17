---
layout: docs
title: Performance Benchmarking
---

<h1 class="page-header">Performance Benchmarking</h1>

<p class="lead">gRPC is designed to support high-performance
open-source RPCs in many languages. This document describes the
performance benchmarking tools, the scenarios considered by the tests,
and the testing infrastructure.</p>

<div id="toc"></div>

## Overview

gRPC is designed for both high-performance and high-productivity design
of distributed applications.

## Performance testing design

## Languages under test

The following languages have continuous performance testing as both
clients and servers at master:

  * C++
  * Java
  * Go
  * C#
  * node.js
  * Python
  * Ruby

PHP does not have any performance testing because our performance
tests are implemented as gRPC servers, which are not supported in
PHP. Mobile performance testing is also currently not supported.

Additionally, all languages derived from C core have limited
performance testing (smoke testing) conducted at every pull request.

In addition to running as both the client-side and server-side of
performance tests, all languages are tested as clients against a C++
server, and as servers against a C++ client. This test aims to provide
the current upper bound of performance for a given language's client or
server implementation without testing the other side.

## Scenarios under test

## Testing infrastructure

