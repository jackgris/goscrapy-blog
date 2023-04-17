---
title: "Basics about a gRPC Server"
date: 2023-04-16T20:39:08-03:00
draft: true
tags: ["api", "documentation", "golang", "grpc"]
---

### Why use gRPC?

The main usage scenarios:

- Low latency, highly scalable, distributed systems.
- Developing mobile clients which are communicating to a cloud server.
- Designing a new protocol that needs to be accurate, efficient and language independent.
- Layered design to enable extension eg. authentication, load balancing, logging and monitoring etc.

#### Who’s using this and why?

gRPC is a Cloud Native Computing Foundation (CNCF) project. Google has been using a lot of the underlying technologies and concepts in gRPC for a long time. The current implementation is being used in several of Google’s cloud products and Google externally facing APIs. It is also being used by Square, Netflix, CoreOS, Docker, CockroachDB, Cisco, Juniper Networks and many other organizations and individuals.

[Here](https://grpc.io/docs/#official-support) you can see the officials languages supported. Also JavaScript implementation of gRPC for browser clients (you can see it [here](https://github.com/grpc/grpc-web)). And it's under Apache 2.0 license.

#### How does gRPC help in mobile application development?
gRPC and Protobuf provide an easy way to precisely define a service and auto generate reliable client libraries for iOS, Android and the servers providing the back end. The clients can take advantage of advanced streaming and connection features which help save bandwidth, do more over fewer TCP connections and save CPU usage and battery life.

### What's protocol buffers?

https://protobuf.dev/overview/

### RPC life cycle

#### Timeouts and termination

#### Metadata

#### Channels

### You need

Install the protocol compiler plugins for Go using the following commands:
```bash
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```
#### Generate gRPC code

Before you can use the new service method, you need to recompile the updated `.proto` file.
While you are in the directory where is the `.proto` file, run the following command:
```bash
$ protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    your_file.proto
```

This will regenerate the your_file.pb.go and your_file_grpc.pb.go files, which contain code for populating, serializing, and retrieving request and reply message types. Generated client and server code

You have regenerated server and client code, but you still need to implement and call the new method in the human-written parts of the example application.

#### I highly recommend reading these articles.

[gRPC](https://grpc.io/)

[Protocol Buffer](https://protobuf.dev/)

[gRPC Motivation and Design Principles](https://grpc.io/blog/principles/)

[Stubbing gRPC in Go](https://jadekler.github.io/2020/10/08/stubbing-grpc.html)

[Talking to Go gRPC Services Via HTTP/1](https://www.youtube.com/watch?v=Vbw8h0RCn2E)
