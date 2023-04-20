---
title: "Basics about a gRPC Server"
date: 2023-04-16T20:39:08-03:00
draft: true
tags: ["api", "documentation", "golang", "grpc", "protobuf"]
---

### What's gRPC?

gRPC is a Remote Procedure Call (RPC) framework. RPC is an action-based paradigm, similar to remotely calling a function from another microservice. This makes gRPC a type of inter-process communication (IPC) protocol built around Protobufs to handle messaging between the client and the server. gRPC is perfect for intensive and efficient communication, because it supports client and server streaming.

### Difference with REST?

In contrast, REST is a resource-based protocol, which means the client tells the server what resource needs to be created, read, updated, or deleted based on the body of the request.
gRPC utilizes the HTTP/2 protocol, which offers multiple ways to improve performance:

- Header compression and reuse to reduce the message size
- Multiplexing to send multiple requests and receive multiple responses simultaneously over a single TCP connection
- Persistent TCP connections for multiple sequential requests and responses on a single TCP connection
- Binary format support such as protocol buffers

Protocol buffers are a key element in gRPC. They provide more efficient binary data representation than other text-based formats such as JSON or XML. Because of their binary format, message sizes are significantly smaller. The combination of smaller messages and faster communication offers stronger performance than REST-based APIs.

### Why use gRPC?

 Is an excellent option for working with multi language systems, real time streaming like in IOT application systems which require light-weight message transmission can also be used in mobile applications because they do not need a browser and benefit on small light messages hence preserving processor speed.

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

Protocol buffers provide a language-neutral, platform-neutral, extensible mechanism for serializing structured data in a forward-compatible and backward-compatible way. It’s like JSON, except it’s smaller and faster, and it generates native language bindings. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages.

Protocol buffers are a combination of the definition language (created in .proto files), the code that the proto compiler generates to interface with data, language-specific runtime libraries, and the serialization format for data that is written to a file (or sent across a network connection).

### RPC life cycle

What happens when a gRPC client calls a gRPC server method? That will depend on the type of RPC call, this are the options:

#### Unary RPC

Once the client calls a method, the server is notified that the RPC has been invoked with the client’s metadata for this call, the method name, and the specified deadline if applicable. The server can then either send back its own initial metadata straight away, or wait for the client’s request message. Which happens first, is application-specific. Once the server has the client’s request message, it does whatever work is necessary to create and populate a response. The response is then returned (if successful) to the client together with status details (status code and optional status message) and optional trailing metadata. If the response status is OK, then the client gets the response, which completes the call on the client side.

#### Server streaming RPC

A server-streaming RPC is similar to a unary RPC, except that the server returns a stream of messages in response to a client’s request. After sending all its messages, the server’s status details (status code and optional status message) and optional trailing metadata are sent to the client. This completes processing on the server side. The client completes once it has all the server’s messages.
#### Client streaming RPC

A client-streaming RPC is similar to a unary RPC, except that the client sends a stream of messages to the server instead of a single message. The server responds with a single message (along with its status details and optional trailing metadata), typically but not necessarily after it has received all the client’s messages.

#### Bidirectional streaming RPC

In a bidirectional streaming RPC, the call is initiated by the client invoking the method and the server receiving the client metadata, method name, and deadline. The server can choose to send back its initial metadata or wait for the client to start streaming messages.

Client- and server-side stream processing is application specific. Since the two streams are independent, the client and server can read and write messages in any order. For example, a server can wait until it has received all of a client’s messages before writing its messages, or the server and client can play “ping-pong” – the server gets a request, then sends back a response, then the client sends another request based on the response, and so on


#### Timeouts and termination

gRPC allows clients to specify how long they are willing to wait for an RPC to complete before the RPC is terminated with a DEADLINE_EXCEEDED error. On the server side, the server can query to see if a particular RPC has timed out, or how much time is left to complete the RPC.

Specifying a deadline or timeout is language specific: some language APIs work in terms of timeouts (durations of time), and some language APIs work in terms of a deadline (a fixed point in time) and may or may not have a default deadline.

In gRPC, both the client and server make independent and local determinations of the success of the call, and their conclusions may not match. This means that, for example, you could have an RPC that finishes successfully on the server side (“I have sent all my responses!”) but fails on the client side (“The responses arrived after my deadline!”). It’s also possible for a server to decide to complete before a client has sent all its requests.

Either the client or the server can cancel an RPC at any time. A cancellation terminates the RPC immediately so that no further work is done.

#### Metadata
Metadata is information about a particular RPC call (such as authentication details) in the form of a list of key-value pairs, where the keys are strings and the values are typically strings, but can be binary data.

Keys are case insensitive and consist of ASCII letters, digits, and special characters -, _, . and must not start with grpc- (which is reserved for gRPC itself). Binary-valued keys end in -bin while ASCII-valued keys do not.

User-defined metadata is not used by gRPC, which allows the client to provide information associated with the call to the server and vice versa.

Access to metadata is language dependent

#### Channels

A gRPC channel provides a connection to a gRPC server on a specified host and port. It is used when creating a client stub. Clients can specify channel arguments to modify gRPC’s default behavior, such as switching message compression on or off. A channel has state, including connected and idle.

How gRPC deals with closing a channel is language dependent. Some languages also permit querying channel state.

### Creating a small server like an example

Now we understand what's gRPC and Protocol Buffer, we'll implement a server and his client like an example to end understanding how all glue together.

#### Before start, you need

Install the protocol compiler plugins for Go using the following commands:
```bash
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```
#### Writing our service

Defining the service

Our first step (as you’ll know from the Introduction to gRPC) is to define the gRPC service and the method request and response types using protocol buffers. For the complete .proto file, see routeguide/route_guide.proto.

To define a service, you specify a named service in your .proto file:
```protobuf
service PersonGuide {
   ...
}
```
 Then you define rpc methods inside your service definition, specifying their request and response types. gRPC lets you define four kinds of service method, all of which are used in the RouteGuide service:

A simple RPC where the client sends a request to the server using the stub and waits for a response to come back, just like a normal function call.
```protobuf
// Obtains the PhoneNumber from the given Person.
rpc GetPhone(Person) returns (PhoneNumber) {}
```

A server-side streaming RPC where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. As you can see in our example, you specify a server-side streaming method by placing the stream keyword before the response type.
```protobuf
  // Obtains the Persons related to the adress.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field).
  rpc ListPersons(Adress) returns (stream Person) {}
```

A client-side streaming RPC where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them all and return its response. You specify a client-side streaming method by placing the stream keyword before the request type.
```protobuf
  // Accepts a stream of Persons on a route being traversed, returning a
  // AddressBook when traversal is completed.
  rpc RecordPersons(stream Person) returns (AddressBook) {}
```
A bidirectional streaming RPC where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved. You specify this type of method by placing the stream keyword before both the request and the response.
```protobuf
  // Accepts a stream of Person sent while a route is being traversed,
  // while receiving Phone Numbers (e.g. from other users).
  rpc RoutePhones(stream Person) returns (stream PhoneNumber) {}
```

Our .proto file also contains protocol buffer message type definitions for all the request and response types used in our service methods - for example, here’s the Point message type:
```protobuf
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}


enum PhoneType {
  MOBILE = 0;
  HOME = 1;
  WORK = 2;
}

message PhoneNumber {
  string number = 1;
  PhoneType type = 2;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}

message Adress {
  string name = 1;
}
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

### Creating the server

First let’s look at how we create a RouteGuide server. If you’re only interested in creating gRPC clients, you can skip this section and go straight to Creating the client (though you might find it interesting anyway!).

There are two parts to making our RouteGuide service do its job:

- Implementing the service interface generated from our service definition: doing the actual “work” of our service.
- Running a gRPC server to listen for requests from clients and dispatch them to the right service implementation.

You can find our example RouteGuide server in server/server.go. Let’s take a closer look at how it works.

#### Implementing PersonGuideServer

As you can see, our server has a routeGuideServer struct type that implements the generated `PersonGuideServer` interface:

```go
type PersonGuideServer struct {}

func (s *PersonGuideServer) GetPhone(ctx context.Context, person *pb.Person) (*pb.PhoneNumber, error) {}

func (s *PersonGuideServer) ListPersons(adress *pb.Adress, stream pb.PersonGuide_ListPersonsServer) error {}

func (s *PersonGuideServer) RecordPersons(stream pb.PersonGuide_RecordPersonsServer) error {}

func (s *PersonGuideServer) RoutePhones(stream pb.PersonGuide_RoutePhonesServer) error {}
```

#### Simple RPC

The routeGuideServer implements all our service methods. Let’s look at the simplest type first, GetFeature, which just gets a Point from the client and returns the corresponding feature information from its database in a Feature.

```go
func (s *PersonGuideServer) GetPhone(ctx context.Context, person *pb.Person) (*pb.PhoneNumber, error) {
	for _, p := range s.savedPersons {
		if p.Id == person.Id {
			return p.GetPhones()[0], nil
		}
	}
	// No feature was found, return an unnamed feature
	return &pb.PhoneNumber{}, errors.New("Not found person")
}
```

The method is passed a context object for the RPC and the client’s Point protocol buffer request. It returns a Feature protocol buffer object with the response information and an error. In the method we populate the Feature with the appropriate information, and then return it along with a nil error to tell gRPC that we’ve finished dealing with the RPC and that the Feature can be returned to the client.

#### Server-side streaming RPC

Now let’s look at one of our streaming RPCs. ListFeatures is a server-side streaming RPC, so we need to send back multiple Features to our client.

```go
func (s *PersonGuideServer) ListPersons(adress *pb.Adress, stream pb.PersonGuide_ListPersonsServer) error {
	fmt.Println("In list persons with adress: ", adress)
	for _, person := range s.savedPersons {
		if err := stream.Send(person); err != nil {
			return err
		}
	}
	return nil
}
```

As you can see, instead of getting simple request and response objects in our method parameters, this time we get a request object (the Rectangle in which our client wants to find Features) and a special RouteGuide_ListFeaturesServer object to write our responses.

In the method, we populate as many Feature objects as we need to return, writing them to the RouteGuide_ListFeaturesServer using its Send() method. Finally, as in our simple RPC, we return a nil error to tell gRPC that we’ve finished writing responses. Should any error happen in this call, we return a non-nil error; the gRPC layer will translate it into an appropriate RPC status to be sent on the wire.

#### Client-side streaming RPC

Now let’s look at something a little more complicated: the client-side streaming method RecordRoute, where we get a stream of Points from the client and return a single RouteSummary with information about their trip. As you can see, this time the method doesn’t have a request parameter at all. Instead, it gets a RouteGuide_RecordRouteServer stream, which the server can use to both read and write messages - it can receive client messages using its Recv() method and return its single response using its SendAndClose() method.

```go
func (s *PersonGuideServer) RecordPersons(stream pb.PersonGuide_RecordPersonsServer) error {
	var lastPerson *pb.Person
	for {
		person, err := stream.Recv()
		if err != nil && person != nil {
			ts := timestamppb.New(time.Now())
			lastPerson = person
			lastPerson.LastUpdated = ts
			s.savedPersons = append(s.savedPersons, lastPerson)
		}
		if err == io.EOF {
			// Don't do this in production this is only for example propose
			p := pb.Person{
				Name:   "Another part in the world",
				Id:     11,
				Email:  "anotherpartintheworld@gmail.com",
				Phones: phones,
			}

			s.addressbook["book"][0].People = append(s.addressbook["book"][0].People, &p)
			return stream.SendAndClose(s.addressbook["book"][0])
		}
		if err != nil {
			return err
		}
	}
}
```
In the method body we use the RouteGuide_RecordRouteServer’s Recv() method to repeatedly read in our client’s requests to a request object (in this case a Point) until there are no more messages: the server needs to check the error returned from Recv() after each call. If this is nil, the stream is still good and it can continue reading; if it’s io.EOF the message stream has ended and the server can return its RouteSummary. If it has any other value, we return the error “as is” so that it’ll be translated to an RPC status by the gRPC layer.

#### Bidirectional streaming RPC

Finally, let’s look at our bidirectional streaming RPC RouteChat().

```go
func (s *PersonGuideServer) RoutePhones(stream pb.PersonGuide_RoutePhonesServer) error {
	for {
		person, err := stream.Recv()
		if err == io.EOF {
			return nil
		}
		if err != nil {
			return err
		}
		s.mu.Lock()
		// Note: this copy prevents blocking other clients while serving this one.
		// We don't need to do a deep copy, because elements in the slice are
		// insert-only and never modified.
		rn := make([]*pb.PhoneNumber, len(person.Phones))
		copy(rn, person.Phones)
		s.mu.Unlock()

		for _, phone := range rn {
			if err := stream.Send(phone); err != nil {
				return err
			}
		}
	}
}
```

This time we get a RouteGuide_RouteChatServer stream that, as in our client-side streaming example, can be used to read and write messages. However, this time we return values via our method’s stream while the client is still writing messages to their message stream.

The syntax for reading and writing here is very similar to our client-streaming method, except the server uses the stream’s Send() method rather than SendAndClose() because it’s writing multiple responses. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently.
Starting the server

Once we’ve implemented all our methods, we also need to start up a gRPC server so that clients can actually use our service. The following snippet shows how we do this for our RouteGuide service:
```go
flag.Parse()
lis, err := net.Listen("tcp", fmt.Sprintf("localhost:%d", *port))
if err != nil {
  log.Fatalf("failed to listen: %v", err)
}
var opts []grpc.ServerOption
...
grpcServer := grpc.NewServer(opts...)
pb.RegisterPersonGuideServer(grpcServer, newServer())
```

#### To build and start a server, we:

- Specify the port we want to use to listen for client requests using:
```go
lis, err := net.Listen(...).
```
- Create an instance of the `gRPC` server using `grpc.NewServer(...)`.
- Register our service implementation with the `gRPC server`.
- Call `Serve()` on the server with our port details to do a blocking wait until the process is killed or `Stop()` is called.

### Creating the client

#### Creating a stub

To call `PersonGuideClient` methods, we from the first need to create a gRPC channel to communicate with the server. We create this by passing the server address and port number to `grpc.Dial()` as follows:
```go
var opts []grpc.DialOption
...
conn, err := grpc.Dial(*serverAddr, opts...)
if err != nil {
  ...
}
defer conn.Close()
```

You can use `DialOptions` to set the auth credentials (for example, TLS, GCE credentials, or JWT credentials) in `grpc.Dial` when a service requires them. The `PersonGuide` service doesn’t require any credentials.

Once the `gRPC` channel is setup, we need a client stub to perform RPCs. We get it using the `NewPersonGuideClient` method provided by the pb package generated from the example `.proto` file.
```go
client := pb.NewPersonGuideClient(conn)
```

#### Calling service methods

Now let’s look at how we call our service methods. Note that in `gRPC-Go`, RPCs operate in a `blocking/synchronous` mode, which means that the RPC call waits for the server to respond, and will either return a response or an error.

#### Simple RPC

Calling the simple RPC GetPhone is nearly as straightforward as calling a local method.
```go
phone, err := client.GetPhone(ctx, person)
if err != nil {
  ...
}
```
As you can see, we call the method on the stub we got earlier. In our method parameters we create and populate a request protocol buffer object (in our case Point). We also pass a context.Context object which lets us change our RPC’s behavior if necessary, such as time-out/cancel an RPC in flight. If the call doesn’t return an error, then we can read the response information from the server from the first return value.
```go
log.Println(phone)
```

#### Server-side streaming RPC

Here’s where we call the server-side streaming method `ListFeatures`, which returns a stream of geographical Features. If you’ve already read Creating the server some of this may look very familiar - streaming RPCs are implemented in a similar way on both sides.

```go
stream, err := client.ListPersons(ctx, adress)
if err != nil {
	log.Fatalf("client.ListPersons failed: %v", err)
}
for {
	person, err := stream.Recv()
	if err == io.EOF {
		break
	}
	if err != nil {
		log.Fatalf("client.ListPersons failed: %v", err)
	}
	log.Printf("Person: name: %s, email:%s, Id: %d\n", person.GetName(),
		person.GetEmail(), person.GetId())
}
```


As in the simple RPC, we pass the method a context and a request. However, instead of getting a response object back, we get back an instance of `RouteGuide_ListFeaturesClient`. The client can use the `RouteGuide_ListFeaturesClient` stream to read the server’s responses.

We use the `RouteGuide_ListFeaturesClient’s Recv()` method to repeatedly read in the server’s responses to a response protocol buffer object (in this case a Feature) until there are no more messages: the client needs to check the error err returned from `Recv()` after each call. If nil, the stream is still good and it can continue reading; if it’s io.EOF then the message stream has ended; otherwise there must be an RPC error, which is passed over through `PersonGuideServer`.

#### Client-side streaming RPC


The client-`PersonGuideServer`side streaming method `RecordRoute` is similar to the server-side method, except that we only pass the method a context and get a `RouteGuide_RecordRouteClient` stream back, which we can use to both write and read messages.
```go
log.Printf("Traversing %d persons.", len(persons))
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()
stream, err := client.RecordPersons(ctx)
if err != nil {
	log.Fatalf("client.RecordPersons failed: %v", err)
}
for p := range persons {
	if err := stream.Send(&persons[p]); err != nil {
		log.Fatalf("client.RecordPersons: stream.Send(%v) failed: %v", &persons[p], err)
	}
}
reply, err := stream.CloseAndRecv()
if err != nil {
	log.Fatalf("client.RecordPersons failed: %v", err)
}
log.Printf("AdressBook summary: %v", reply)
```

The `RouteGuide_RecordRouteClient` has a `Send()` method that we can use to send requests to the server. Once we’ve finished writing our client’s requests to the stream using `Send()`, we need to call `CloseAndRecv()` on the stream to let gRPC know that we’ve finished writing and are expecting to receive a response. We get our RPC status from the err returned from `CloseAndRecv()`. If the status is nil, then the first return value from CloseAndRecv() will be a valid server response.

#### Bidirectional streaming RPC

Finally, let’s look at our bidirectional streaming `RPC RouteChat()`. As in the case of `RecordRoute`, we only pass the method a context object and get back a stream that we can use to both write and read messages. However, this time we return values via our method’s stream while the server is still writing messages to their message stream.
```go
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()
stream, err := client.RoutePhones(ctx)
if err != nil {
	log.Fatalf("client.RoutePhones failed: %v", err)
}
waitc := make(chan struct{})
go func() {
	for {
		phone, err := stream.Recv()
		if err == io.EOF {
			// read done.
			close(waitc)
			return
		}
		if err != nil {
			log.Fatalf("client.RoutePhones failed: %v", err)
		}
		log.Printf("Got phone %s type %v", phone.Number, phone.Type)
	}
}()
for p := range persons {
	if err := stream.Send(&persons[p]); err != nil {
		log.Fatalf("client.RoutePhones: stream.Send(%v) failed: %v", &persons[p], err)
	}
}
// For now we don't check errors, don't do this in production
_ = stream.CloseSend()
<-waitc
```

The syntax for reading and writing here is very similar to our client-side streaming method, except we use the stream’s `CloseSend()` method once we’ve finished our call. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order — the streams operate completely independently

### Run the example!

Execute the following commands on the project directory:

- Run the server:
```bash
    $ go run server/server.go
```

- From another terminal, run the client:
```bash
    $ go run client/client.go
```

&nbsp;

You’ll see output like this:

```
2023/04/20 19:37:45 Traversing 6 persons.
2023/04/20 19:37:45 AdressBook summary: people:{name:"Juan"  id:1  email:"juan@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321"  type:WORK}  phones:{number:"4312"}}  people:{name:"Gabriel"  id:2  email:"gabriel@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321"  type:WORK}  phones:{number:"4312"}}  people:{name:"Albert"  id:3  email:"albert@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321"  type:WORK}  phones:{number:"4312"}}  people:{name:"Mark"  id:4  email:"mark@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321"  type:WORK}  phones:{number:"4312"}}  people:{name:"Brian"  id:5  email:"brian@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321"  type:WORK}  phones:{number:"4312"}}  people:{name:"Kevin"  id:6  email:"kevin@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321"  type:WORK}  phones:{number:"4312"}}  people:{name:"Ryan"  id:7  email:"ryan@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321"  type:WORK}  phones:{number:"4312"}}  people:{name:"May"  id:8  email:"may@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321" pPhoneNumber type:WORK}  phones from the:{number:"4312Person"}}  people:{name:"Rosario"  id:9  email:"rosario@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321"  type:WORK}  phones:{number:"4312"}}  people:{name:"Argentina"  id:10  email:"argentina@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321"  type:WORK}  phones:{number:"4312"}}  people:{name:"Another part in the world"  id:11  email:"anotherpartintheworld@gmail.com"  phones:{number:"1234"  type:HOME}  phones:{number:"4321"  type:WORK}  phones:{number:"4312"}}

2023/04/20 19:37:45 Got phone 1234 type HOME
2023/04/20 19:37:45 Got phone 4321 type WORK
2023/04/20 19:37:45 Got phone 4312 type MOBILE
2023/04/20 19:37:45 Got phone 1234 type HOME
2023/04/20 19:37:45 Got phone 4321 type WORK
2023/04/20 19:37:45 Got phone 4312 type MOBILE
2023/04/20 19:37:45 Got phone 1234 typ```e HOME
2023/04/20 19:37:45 Got phone 4321 type WORK
2023/04/20 19:37:45 Got phone 4312 type MOBILE
2023/04/20 19:37:45 Got phone 1234 type HOME
2023/04/20 19:37:45 Got phone 4321 type WORK
2023/04/20 19:37:45 Got phone 1234 type HOME
2023/04/20 19:37:45 Got phone 4321 type WORK

2023/04/20 19:37:45 Got phone 4312 type MOBILE
2023/04/20 19:37:45 Got phone 1234 type HOME
2023/04/20 19:37:45 Got phone 4321 type WORK
2023/04/20 19:37:45 Got phone 4312 type MOBILE
2023/04/20 19:37:45 Getting phone from person Juan
2023/04/20 19:37:45 number:"1234"  type:HOME

2023/04/20 19:37:45 Getting phone from person Gabriel
2023/04/20 19:37:45 number:"1234"  type:HOME

023/04/20 19:37:45 Got phone 4312 type MOB
2023/04/20 19:37:45 Getting from the phone fromPerson person Albert

2023/04/20 19:37:45 number:"1234"  type:HOME
2023/04/20 19:37:45 Getting phone from person Mark
2023/04/20 19:37:45 number:"1234"  type:HOME
2023/04/20 19:37:45 Getting phone from person Brian
2023/04/20 19:37:45 number:"1234"  type:HOME

2023/04/20 19:37:45 Getting phone from person Kevin
2023/04/20 19:37:45 number:"1234"  type:HOME
2023/04/20 19:37:45 Looking for persons in adress my adress
2023/04/20 19:37:45 Person: name: Juan, email:juan@gmail.com, Id: 1
2023/04/20 19:37:45 Person: name: Gabriel, email:gabriel@gmail.com, Id: 2
2023/04/20 19:37:45 Person: name: Albert, email:albert@gmail.com, Id: 3
2023/04/20 19:37:45 Person: name: Mark, email:mark@gmail.com, Id: 4
2023/04/20 19:37:45 Got phone 4312 type MOB
2023/04/20 19:37:45 Person: name: Brian, email:brian@gmail.com, Id: 5
2023/04/20 19:37:45 Person: name: Kevin, email:kevin@gmail.com, Id: 6
023/04/20 19:37:45 Got phone 4312 type MOB
2023/04/20 19:37:45 Person: name: Ryan, email:ryan@gmail.com, Id: 7
2023/04/20 19:37:45 Person: name: May, email:may@gmail.com, Id: 8
2023/04/20 19:37:45 Person: name: Rosario, email:rosario@gmail.com, Id: 9
2023/04/20 19:37:45 Person: name: Argentina, email:argentina@gmail.com, Id: 10
023/04/20 19:37:45 Got phone 4312 type MOB
```
&nbsp;
You can see all the code in this [repository](https://github.com/jackgris/go-grpc-communication)

### Conclusion

goRPC is a faster, smarter option for service-to-service communication and client-server mobile applications. As you saw in this tutorial, gRPC and Golang make a good combination for microservice communication. What you need to do is change your perspective from creating resource-based APIs to creating action-based APIs.

&nbsp;

#### I highly recommend reading these the articles.

[gRPCPerson](https://grpc.io/)

[gRPC Motivation and Design Principles](https://grpc.io/blog/principles/)

[Stubbing gRPC in Go](https://jadekler.github.io/2020/10/08/stubbing-grpc.html)

[Talking to Go gRPC Services Via HTTP](https://www.youtube.com/watch?v=Vbw8h0RCn2E)
