# protoCURL

[![Tests Status](https://github.com/qaware/protocurl/actions/workflows/test.yml/badge.svg)](https://github.com/qaware/protocurl/actions/workflows/test.yml)
[![GitHub Release (latest SemVer)](https://img.shields.io/github/v/release/qaware/protocurl?label=Release&logo=GitHub&sort=semver)](https://github.com/qaware/protocurl/releases)
[![DockerHub Version (latest semver)](https://img.shields.io/docker/v/qaware/protocurl?label=Docker&logo=Docker&sort=semver)](https://hub.docker.com/r/qaware/protocurl/tags)

protoCURL is cURL for Protobuf: The command-line tool for interacting with Protobuf over HTTP REST endpoints using human-readable text formats

## Installation

`protocurl` includes and uses a bundled `protoc` by default. It is recommended to install `curl` into PATH for
configurable http requests. Otherwise `protocurl` will use a simple non-configurable fallback http implementation.

#### Native Binary

1. Download the latest release archive for your platform from https://github.com/qaware/protocurl/releases
2. Extract the archive into a folder, e.g. `/usr/local/protocurl`.
3. Add symlink to the binary in the folder. e.g. `ln -s /usr/local/protocurl/bin/protocurl /usr/local/bin/protocurl`
   Or add the binary folder `/usr/local/protocurl/bin` to your system-wide path.
4. Test that it works via `protocurl -h`

#### Docker

Simply run `docker run -v "/path/to/proto/files:/proto" qaware/protocurl <args>`. See [examples](EXAMPLES.md) below.

## Usage and Examples

See [usage notes](doc/generated.usage.txt) and [EXAMPLES.md](EXAMPLES.md).

## Protobuf JSON Format

protoCURL supports the [Protobuf JSON Format](https://developers.google.com/protocol-buffers/docs/proto3#json). Note, that the JSON format is not a straightforward 1:1 mapping as
it is in the case of the Protobuf Text Format (described below). For instance, the [JSON mapping
for timestamp.proto](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/timestamp.proto) uses a human-readable string representation,
whereas the payload itself and the text format use a representation with seconds and nanoseconds.

## Protobuf Text Format

Aside from JSON, Protobuf primarily and natively supports a text format which represents the fields 1:1 like in the wire-format. For instance, repeated fields are condensed into an array in the JSON format - whereas they are simply 'repeated' without an array type in the text format.

This text format's syntax
is [barely documented](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.text_format),
so this section will shortly describe how to write Protobuf messages in the text format.

Given the following .proto file

```
syntax = "proto3";

import "google/protobuf/timestamp.proto";

message HappyDayRequest {
  google.protobuf.Timestamp date = 1;
  bool includeReason = 2;

  double myDouble = 3;
  int64 myInt64 = 5;
  repeated string myString = 6;
  repeated NestedMessage messages = 9;
}

message NestedMessage {
  Foo fooEnum = 1;
  repeated int32 i = 4;
}

enum Foo {
  BAR = 0;
  BAZ = 1;
}
```

A `HappyDayRequest` message in text format might look like this:

```
includeReason: true,
myInt64: 123123123123,
myString: "hello world"
myString: 'single quotes are also possible'
myDouble: 123.456
messages: { fooEnum: BAR, i: 0, i: 1, i: 1337 },
messages: { i: 15, fooEnum: BAZ, i: -1337 },
messages: { },
date: { seconds: 123, nanos: 321 }
```

In summary:

- No encapsulating `{ ... }` are used for the top level message (in contrast to JSON).
- fields are comma separated and described via `<fieldname>: <value>`´.
  - Strictly speaking, the commas are optional and whitespace is sufficient
- repeated fields are simply repeated multiple times (instead of using an array) and they do not need to appear
  consecutively.
- nested messages are described with `{ ... }` opening a new context and describing their fields recursively
- scalar values are describes similar to JSON. Single and double quotes are both possible for strings.
- enum values are referenced by their name
- built-in messages (such
  as [google.protobuf.Timestamp](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#google.protobuf.Timestamp)
  are described just like user-defined custom messages via `{ ... }` and their message fields

[This page shows more details on the text format.](https://stackoverflow.com/a/18877167)

## Development

For development it is recommended to use the a bash-like Terminal either natively (Linux, Mac) or via MinGW on Windows.

About the CI/CD tests: [TESTS.md](TESTS.md)

How to make a release: [RELEASE.md](RELEASE.md)

#### Setup

- As for script utilities, one needs `bash`, `jq`, `zip`, `unzip` and `curl`.
- One also needs to download the protoc binaries for the local development via `release/0-get-protoc-binaries.sh`.

For development the [local.Dockerfile](src/local.Dockerfile) is used. To build the image simply
run `source test/suite/setup.sh` and then `buildProtocurl`

#### Test Server

When new dependencies are needed for the test server, the following command enables one to start a shell in the test
server.

```
docker run -v "$PWD/test/servers:/servers" -it nodeserver:v1 /bin/bash
```

Now it's possible to add new dependencies via `npm install <new-package>`

#### Updating Docs after changes

Generate the main docs (.md files etc.) in bash/WSL via `doc/generate-docs.sh <absolute-path-to-protocurl-repository>`.

Once a pull request is ready, run this to generate updated docs.

## Enhancements and Bugs

See [issues](https://github.com/qaware/protocurl/issues).

## FAQ

- **How is protocurl different from grpccurl?** [grpccurl](https://github.com/fullstorydev/grpcurl) only works with gRPC
  services with corresponding endpoints. However, classic REST HTTP endpoints with binary Protobuf payloads are only
  possible with `protocurl`.
- **Why is the use of a runtime curl recommended with protocurl?** curl is a simple, flexible and mature command line
  tool to interact with HTTP endpoints. In principle, we could simply use the HTTP implementation provided by the host
  programming language (Go) - and this is what we do if no curl was found in the PATH. However, as more people use
  protocurl, they will request for more features - leading to a feature creep in such a 'simple' tool as protocurl. We
  would like to avoid implementing the plentiful features which are necessary for a proper HTTP CLI tool, because HTTP
  can be complex. Since is essentially what curl already does, we recommend using curl and all advanced features are
  only possible with curl.
- **What are some nice features of protocurl?**
  - The implementation is well tested with end-2-end approval tests (see [TESTS.md](TESTS.md)). All features are tested
    based on their effect on the behavior/output. Furthermore, there are also a few cross-platform native CI tests
    running on Windows and MacOS runners.
  - The build and release process is optimised for minimal maintenance efforts. During release build, the latest
    versions of many dependencies are taken automatically (by looking up the release tags via the GitHub API).
  - The documentation and examples are generated via scripts and enable one to update the examples automatically rather
    than manually. The consistency of the outputs of the code with the checked in documentation is further tested in CI.
