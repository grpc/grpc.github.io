---
layout: post
title: 2017-08-31 Community Meeting Update
published: true
author: Jaye Pitzeruse
company: Indeed
company-link: https://www.indeed.com
--- 

**Next Community Meeting:** Thursday, September 14, 2017 11am Pacific Time (US and Canada)

<!--more-->

## General Announcements

Mark Roth gave a deep dive into service configuration.
The goal is to allow service owners to specify default parameters for their service.
These parameters are resolved during name resolution.
A service configuration can contain the load balancer policy and a list of method configurations.
You can provide a wildcard configuration which will apply to all methods of the given service.
If your method needs to be tuned, you can override the global with a method level configuration.
In these configurations, you can specify the request timeout, maximum request size, and maximum response size.
For more information about service configuration, see the [gRFC document](https://github.com/grpc/proposal/blob/master/A2-service-configs-in-dns.md).
For information about language support, see the [working document notes](https://docs.google.com/document/d/1DTMEbBNmzNbZBh8nOivsnnw3CwUr1Q7WGRe7rNxyHOU/edit#bookmark=id.fl7x041naw19).

## Release Updates

1.6 is available.
Look for the release in your corresponding language.

## Platform Updates

gRPC RPM now available for Fedora Linux (26 and up).

## Language Updates

C-Core (C++, Node.js, Python, Ruby, Objective-C, PHP, and C#) 1.6 release is available.
Release notes available [here](https://github.com/grpc/grpc/releases/tag/v1.6.0).

Golang 1.6 release is available.
Release notes available [here](https://github.com/grpc/grpc-go/releases/tag/v1.6.0).

Java's 1.6.0 and 1.6.1 is available.
The code in these two versions are identical.
1.6.0 tag was deleted due to an invalid tag.
Release notes will be available today [here](https://github.com/grpc/grpc-java/releases/tag/v1.6.1).

## Resources

* [Working Document Notes](https://docs.google.com/document/d/1DTMEbBNmzNbZBh8nOivsnnw3CwUr1Q7WGRe7rNxyHOU/edit#bookmark=id.fl7x041naw19)
