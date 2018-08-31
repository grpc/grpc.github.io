---
layout: post
title: gRPC Packaging: Release Process and Nightly Builds
published: true
permalink: blog/grpc-packaging-releases-nightlies
author: Mehrdad Afshari
author-link: https://mehrdad.afshari.me
company: Google
company-link: https://www.google.com
---

# gRPC Packaging: Release Process and Nightly Builds #

gRPC is [developed in the open on GitHub](https://github.com/grpc) and the
`master` branch undergoes continuous testing to try to ensure the `HEAD` of
`master` is buildable and passes the tests.  In this post, we discuss the
release process of gRPC as an open source project to clarify it for our users,
and to serve as an example for other open source projects to consider adopting.
This article is written mostly from the perspective of the gRPC Core (C-based)
languages.  The release processes of gRPC Java and Go, although similar in
spirit, may be slightly different in technicalities.

## Official Releases ##

gRPC follows semantic versioning and we aim to ship a new minor version every
six weeks.  While `master` is quite good and heavily tested, and we could
almost always simply consider tagging its `HEAD` as a minor release whenever we
like to ship one, a more rigorous quality control process is justified for
supported gRPC releases, as it is a critical piece of infrastructure for many
users.  As a consequence, we schedule a feature freeze every six weeks.  [gRPC
Release Schedule]() highlights the actual dates.

### Feature Freeze: Release Branch Cut ###

The feature window for each minor release closes when a release branch is cut
from `master`.  The release manager of that particular release takes a commit
close to the `HEAD`, often the `HEAD` itself, where the tests are green, and
creates a new release branch at that point.  Starting at the branch cut point,
additional feature development continues on the `master` branch without
affecting the release branch and changes to the release branch are kept
minimal, *i.e.* mostly bug fixes.  The version number on the `master` branch is
promptly bumped to reflect that new features will land starting in the followup
release.  The `master` branch for gRPC Core repository always carries a `-dev`
suffix to indicate it is under active development.

### Prereleases and Release Candidates ###

Once the release branch is cut, its version is updated to reflect its
prerelease status, *e.g.* on the Core repository, the suffix will be changed to
`-pre1` indicating it is the first release candidate of that version
(subsequent RCs, if any, will get `-pre2`, *etc.* suffixes.)

A package build job is then triggered on the prerelease branch to create binary
packages for the supported languages and platforms.  The binary packages
undergo *distribution tests*, which are tests that verify correct installation
and loading of the binary packages on supported platforms.  If everything looks
fine as expected, the release manager then tags `HEAD` of the release branch as
a "prerelease" on GitHub and the prerelease package builds will be uploaded to
the common package managers of the supported languages.

Mission-critical users depending on the latest version of gRPC in their
application should run their continuous tests against the latest prerelease, as
well as the latest release, to see possible regressions early and file issues
and/or temporarily pin the dependency version to guard against breakages.

After the branch is cut and the prerelease is evaluated in the wild for a week
or two, the release candidate gets promoted to the final version.  Bug fixes
landing in the release branch after the first prerelease induce subsequent
prerelease and qualification processes (albeit often with a shorter timescale)
to decrease the risk of regressions the moment a new stable package is uploaded
to the language package managers.  Strictly adhering to this ritual is critical
as even small bug fixes may have unexpected side-effects, wherefore this policy
should only be short-circuited in extreme cases--*yes, we've learned it the
hard way*.

### Final Release ###

Once a prerelease is qualified, it is promoted to the final version, ideally
intact except for the version bump, removing the prerelease suffix, tagging the
final minor version on GitHub, and shipping the final binaries on the common
package managers.

Subsequent bug fixes that warrant a patch release for that minor version
trigger the prerelease and qualification process again.

## Nightlies: Daily Package Builds ##

gRPC supports many language and platform combinations.  While build systems
like [Bazel](https://bazel.build) improve the build from source and
distribution experience for scenarios where it is supported, setting up the
infrastructure to build gRPC from source can be a hurdle for some people who
like to experiement with the bleeding edge gRPC code on the `master` branch.
To alleviate this concern for the users who want to be on the bleeding edge, or
just cannot wait for a particular bug fix or feature on `master` to land in an
official release, as well as facilitating bisection while debugging by
providing prebuilt binaries at regular intervals, [we produce
packages](https://github.com/grpc/grpc/tree/master/tools/internal_ci/linux/grpc_publish_packages.sh)
every day and upload them to a Google Cloud Storage bucket
(`gs://packages.grpc.io`), also served on the web on the [packages.grpc.io
website](https://packages.grpc.io).

The [index page](https://packages.grpc.io/index.xml) is an XML feed (stylized
through an XSL stylesheet for web browser viewability) pointing to each build
for users to parse and fetch the latest packages and feed it into their test
infrastructure for continuous testing under the bleeding-edge gRPC, thereby
encountering future issues early and filing bug reports to avoid accidental
regressions.
