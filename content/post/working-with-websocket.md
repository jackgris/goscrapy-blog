---
title: "Working With Websocket"
date: 2023-05-20T20:33:13-03:00
draft: false
tags: ["websocket", "golang"]
---

![websocket_gopher logo](https://jackgris.github.io/goscrapy-blog/img/websocket-hotpot.png)

## What are websockets and why are they important for web applications?

WebSockets are an essential technology for enabling two-way communication between a client and a server over a single (Enable real-time communication), long-lived connection. They provide real-time, low-latency communication between a web application and a server, allowing for a more interactive and dynamic user experience.

Are a separate implementation on top of TCP, unlike HTTP. They were standardized by the IETF as [RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455) in 2011, and the current API specification allowing web applications to use this protocol is known as websockets. It is a living standard maintained by the [WHATWG](https://whatwg.org/) and a successor to [The WebSocket API from the W3C](https://www.w3.org/TR/2009/WD-websockets-20091222/).

WebSockets are important because they allow full-duplex communication between a client and server, so that either side can push data to the other through an established connection. HTTP is not meant for keeping open a connection for the server to frequently push data to a web browser. Previously, most web applications would implement long polling via frequent [Asynchronous JavaScript and XML (AJAX)](https://es.wikipedia.org/wiki/AJAX) requests. Server push is more efficient and scalable than long polling because the web browser does not have to constantly ask for updates through a stream of AJAX requests. The websockets approach for server- and client-pushed updates works well for certain categories of web applications such as chat rooms, which is why that's often an example application for a websocket library.

## How does the websocket protocol work, including the handshake process and message structure?


To establish a WebSocket connection, the client sends a WebSocket handshake request to the server. The handshake starts with an HTTP request/response, allowing servers to handle HTTP connections as well as WebSocket connections on the same port. The client request includes the following headers:

```
    HTTP method: GET
    Resource name: /chat (or some other endpoint)
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: a random base64-encoded key
    Sec-WebSocket-Protocol (optional): the subprotocol the client is requesting to use
    Sec-WebSocket-Version: the version of the WebSocket protocol being used
    Origin (optional): the origin of the request
```

When the server receives the handshake request, it should send back a special response that indicates that the protocol will be changing from HTTP to WebSocket. The server response includes the following headers:

```
    HTTP/1.1 101 Switching Protocols
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Accept: a hash of the key in the Sec-WebSocket-Key header
```

In addition to upgrade headers, the client sends a Sec-WebSocket-Key header containing base64-encoded random bytes, and the server replies with a hash of the key in the Sec-WebSocket-Accept header. This is intended to prevent a caching proxy from re-sending a previous WebSocket conversation, and does not provide any authentication, privacy, or integrity. The hashing function appends the fixed string (a UUID) to the value from Sec-WebSocket-Key header (which is not decoded from base64), applies the SHA-1 hashing function, and encodes the result using base64.

Once the connection is established, communication switches to a bidirectional binary protocol which does not conform to the HTTP protocol. The WebSocket protocol facilitates message passing between a client and server, and the protocol consists of an opening handshake followed by basic message framing, layered over TCP.

The WebSocket protocol defines two types of messages, text and binary, but their content is undefined. The [WebSocket RFC](https://datatracker.ietf.org/doc/html/rfc6455) defines the use of sub-protocols. During the handshake, the client and server can use the header Sec-WebSocket-Protocol to agree on a sub-protocol, i.e. a higher, application-level protocol to use. The use of a sub-protocol is not required, but even if not used, applications will still need to choose a message format that both the client and server can understand. That format can be custom, framework-specific, or a standard messaging protocol.

## What are the techniques for broadcasting messages or implementing pub/sub?

When implementing broadcasting or pub/sub functionality using WebSockets, you need to establish a mechanism to distribute messages from a sender to multiple recipients (subscribers). Here are a few techniques you can employ:

- `Maintain a List of Connections:` Create a data structure, such as a list or map, to store WebSocket connections for each subscriber. When a message needs to be broadcasted, iterate through the list of connections and send the message to each subscriber individually. This approach is suitable for smaller-scale applications with a limited number of subscribers.

- `Use a Message Broker:` Introduce a message broker or a publish-subscribe system to handle the distribution of messages. When a message is received from a sender, the message is published to a specific topic or channel. Subscribers who are interested in that topic or channel receive the message. Popular message brokers that can be integrated with WebSockets include `RabbitMQ, Apache Kafka, or Redis Pub/Sub`.
  
- `Rooms/Groups:` Assign connections or clients to specific rooms or groups based on their interests or subscriptions. When a message needs to be broadcasted, send the message to all connections in the relevant rooms or groups. This approach allows for more efficient message distribution, as the broadcast is limited to a subset of connected clients.

- `Event-driven Architecture:` Implement an event-driven architecture where subscribers can register specific events or topics they are interested in. When a message is received, the server determines the appropriate subscribers for that message based on their registered interests and sends the message to those subscribers only.

- `Libraries and Frameworks:` Utilize WebSocket libraries or frameworks that provide built-in broadcasting or pub/sub functionality. These libraries often have abstractions and APIs specifically designed for managing connections, message distribution, and pub/sub scenarios. For example, [Emitter](https://github.com/emitter-io/emitter) or [Centrifugo](https://github.com/centrifugal/centrifugo) for Go provides an event-based system with built-in broadcasting capabilities.

## How should errors and close events be handled?

Handling errors and close events is crucial to ensure the reliability and stability of your WebSocket-based application. Here are some best practices for handling errors and close events effectively:

`Error Handling:`
        Implement an error handler function to capture and handle WebSocket errors. Log the error details for debugging purposes and consider taking appropriate actions based on the error type. For example, you might want to notify the user, attempt to reconnect, or gracefully terminate the WebSocket connection. Different types of errors can occur, such as network errors, malformed messages, or server-side errors. Handle each error type appropriately based on the requirements of your application.

`Close Event Handling:`
        Register a close event handler function to handle WebSocket connection closure. The close event occurs when either the client or the server initiates the closing of the WebSocket connection. Handle the close event by performing necessary cleanup tasks, updating application state, and potentially notifying users or triggering appropriate actions. Consider differentiating between expected and unexpected close events. Expected closures may occur as a result of user actions, application-specific logic, or server-side events. Unexpected closures may be due to network issues or server failures. Take appropriate actions based on the type of close event.

`Graceful Disconnection:`
        Ensure that your application gracefully handles disconnection scenarios to avoid leaving resources in an inconsistent state. Clean up any resources associated with the WebSocket connection, such as closing database connections, releasing locks, or cleaning up temporary files. Notify other parts of your application or users about the disconnection event, especially if it impacts shared resources or ongoing processes.

`Reconnection and Retry Mechanisms:`
        If it's appropriate for your application, consider implementing reconnection and retry mechanisms to handle temporary network disruptions or server failures. This can help maintain a more stable and resilient WebSocket connection. Retry connection attempts with increasing delays to avoid overwhelming the server or network with repeated connection requests. Implement a maximum retry limit to prevent endless reconnection attempts in case of persistent issues.


## What are the security considerations when working with websocket connections?

Here are some security considerations to keep in mind:

WebSocket requests are not restricted by the same-origin policy, which means that WebSocket `servers must validate the "Origin" header` against the expected origins during connection establishment to avoid cross-site WebSocket hijacking attacks. WebSocket hijacking attacks are similar to cross-site request forgery, which can be possible when the connection is authenticated with cookies or HTTP authentication.

To authenticate the WebSocket connection when sensitive data is being transferred over the WebSocket, it is better to use `tokens or similar protection mechanisms`. It is important to validate the data that comes from the server via a WebSocket connection, as data returned by the server can potentially be problematic.

It is a good practice to use the `wss://` protocol instead of `ws://`, as it is much safer and can prevent a huge number of attacks from the outset. Using the `WebSocket Secure` protocol can avoid data leaks, as data transfer over the WebSocket protocol is done in plain text, similar to HTTP.

WebSocket gateways/servers `should have fallback` for cases when a WebSocket connection cannot be established. This is a practical concern as many WebSocket applications can expect to have many users relying on fallback methods in the real world. Any security features used for WebSocket should also apply to the fallback when a WebSocket connection cannot be established.

An enterprise WebSocket gateway/server will have security built into the architecture that can be configured when needed. If you are building a real-world or enterprise WebSocket-based application, it is important to `think about your security needs early`. It is not something you want to "bolt on" later, as that will mean having to change your architecture or write a lot of extra code.

## How can applications be scaled and optimized for performance?

WebSocket applications can be scaled and optimized for performance by following these best practices:

- Use horizontal scaling rather than vertical scaling, as it is more reliable and can handle more concurrent connections. If possible, use smaller machines (servers) rather than large ones, as they are easier and faster to spin up, and costs are more granular.

- Use a load balancer (such as HAProxy or Nginx) configured with a sticky-session and messaging system to balance traffic between server processes.

- Have a homogeneous server farm to balance load efficiently and evenly across machines with similar configurations.

- Ensure your server layer is dynamically elastic, so you can quickly scale out when you have traffic spikes. You should also operate with some capacity margin and have backups for various system components to ensure redundancy and remove single points of failure.

- Use a robust real-time monitoring and alerting stack to detect and quickly implement remedial measures when issues occur. Run load and stress testing to understand how your system behaves under peak load and enforce hard limits (for example, maximum number of concurrent connections) to have some predictability.

- Use a scalable architecture pattern, and auto-scalable orchestrators like `Kubernetes`, to handle high traffic and scale in and out as needed.

- Consider the use of fallback transports, because WebSockets are blocked by certain enterprise firewalls and networks. Falling back to another protocol changes your scaling parameters, so you need a strategy to scale both.

## What are some practical use cases and examples of websockets in action?

Some practical examples of how WebSockets are used in action can be:
`
Real-Time Chat Applications / Collaborative Editing Tools /Live Feeds and Notifications / Multiplayer Online Games / Live Tracking and Geolocation / Real-Time Analytics and Dashboards / IoT (Internet of Things) Applications /Live Auctions and Bidding Platforms / Collaborative Drawing or Whiteboarding / Live Customer Support and Help Desks.
`
These are just a few examples of how WebSockets are applied in practical use cases. The real-time and bidirectional nature of WebSockets make them valuable for applications that require instant communication, updates, and collaboration between clients and servers.

## How do I broadcast messages to connected clients?

To broadcast messages to connected clients using WebSockets in Go, you can follow these steps:

1- Use a WebSocket server-side library that supports message broadcasting. For example, you can use the [Gorilla WebSocket library](https://github.com/gorilla/websocket)(I know is archived but still used), which provides a high-performance, scalable WebSocket server implementation for Go.

2- Define a WebSocket channel or room using a unique name that all clients can subscribe to. For example, you can use "my-channel".

3- Store the WebSocket connections in a data structure that allows you to manage and access the connections. For example, you can use a map that maps the WebSocket connection to a boolean value to indicate whether the connection is still open.

```go
clients := make(map[*websocket.Conn]bool)
```

4- Use a goroutine to listen for incoming messages from clients and broadcast them to all other clients that are subscribed to the channel. Here is an example of a goroutine that listens for incoming messages and broadcasts them to all clients:

```go
func broadcastMessage(message []byte) {
    for client := range clients {
        err := client.WriteMessage(websocket.TextMessage, message)
        if err != nil {
            log.Printf("error: %v", err)
            client.Close()
            delete(clients, client)
        }
    }
}
```

5- When a client connects, add the WebSocket connection to the data structure that stores the connections. When a client disconnects, remove the WebSocket connection from the data structure.

```go
func handleWebSocketConnection(w http.ResponseWriter, r *http.Request) {
    // Upgrade HTTP connection to WebSocket
    conn, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Println(err)
        return
    }
    defer conn.Close()

    // Add the WebSocket connection to the list of clients
    clients[conn] = true

    // Listen for incoming messages from the client
    for {
        _, message, err := conn.ReadMessage()
        if err != nil {
            log.Printf("error: %v", err)
            delete(clients, conn)
            break
        }
        // Broadcast the message to all clients
        broadcastMessage(message)
    }
}
```

6- Clients that are subscribed to the channel will receive the message in real-time.

In summary, to broadcast messages to connected clients using WebSockets in Go, you can use a WebSocket server-side library like Gorilla, define a WebSocket channel, store the WebSocket connections in a data structure, use a goroutine to listen for incoming messages and broadcast them to all clients, and clients that are subscribed to the channel will receive the message in real-time.

## How can I secure websocket connections with TLS?

To secure WebSocket connections with TLS in Go, you can follow these steps:

1- Generate a TLS certificate and key pair. You can use a tool like OpenSSL to generate a self-signed certificate and key pair.

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
```
2- Use the http.ListenAndServeTLS() function to encrypt the underlying network connections used for the HTTP protocol and the WebSocket protocol. This will secure the entire communication on the underlying network connection, including all WebSocket traffic.

```go
err := http.ListenAndServeTLS(":8080", "cert.pem", "key.pem", nil)
if err != nil {
    log.Fatal("ListenAndServeTLS: ", err)
}
```

3- Use the "wss" scheme in the client-side WebSocket URL to indicate that the WebSocket connection should be secured with TLS.

```go
conn, err := websocket.Dial("wss://localhost:8080/ws", "", "https://localhost")
if err != nil {
    log.Fatal("Dial: ", err)
}
```

## How do I handle errors and ensure graceful shutdown in Go websocket applications?

To stop receiving and drain messages before closing a WebSocket connection in Go, you can follow these steps:

1- Send a close message to the client to initiate a normal shutdown of the WebSocket connection. You can use the conn.WriteMessage() function to send a close message to the client.

```go
err := conn.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseNormalClosure, ""))
if err != nil {
    log.Println("write close:", err)
    return
}
```

2- Wait for the client to acknowledge the close message by reading from the WebSocket connection until an error occurs. You can use the conn.ReadMessage() function to read a message from the client.

```go
_, _, err := conn.ReadMessage()
if err != nil && !websocket.IsCloseError(err, websocket.CloseNormalClosure, websocket.CloseGoingAway) {
    log.Println("read close:", err)
    return
}
```
3- Close the underlying network connection to complete the normal shutdown of the WebSocket connection. You can use the conn.Close() function to close the WebSocket connection.

```go
conn.Close()
```

4- To drain messages before closing the WebSocket connection, you can use a loop to read messages from the client until an error occurs. You should add a timeout to the read operation to prevent the loop from blocking indefinitely.

```go
for {
    _, _, err := conn.ReadMessage()
    if err != nil {
        if websocket.IsCloseError(err, websocket.CloseNormalClosure, websocket.CloseGoingAway) {
            break
        }
        log.Println("read:", err)
        break
    }
    // Process the message here
}
```

In summary, you should send a close message to the client, wait for the client to acknowledge the close message by reading from the WebSocket connection, close the underlying network connection, and use a loop to read messages from the client until an error occurs.

## How can I implement a basic websocket server using Fiber?

```go
package main

import (
	"log"

	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/websocket/v2"
)

func main() {
	app := fiber.New()

	app.Use("/ws", func(c *fiber.Ctx) error {
		// IsWebSocketUpgrade returns true if the client
		// requested upgrade to the WebSocket protocol.
		if websocket.IsWebSocketUpgrade(c) {
			c.Locals("allowed", true)
			return c.Next()
		}
		return fiber.ErrUpgradeRequired
	})

	app.Get("/ws/:id", websocket.New(func(c *websocket.Conn) {
		// c.Locals is added to the *websocket.Conn
		log.Println(c.Locals("allowed"))  // true
		log.Println(c.Params("id"))       // 123
		log.Println(c.Query("v"))         // 1.0
		log.Println(c.Cookies("session")) // ""

		// websocket.Conn bindings https://pkg.go.dev/github.com/fasthttp/websocket?tab=doc#pkg-index
		var (
			mt  int
			msg []byte
			err error
		)
		for {
			if mt, msg, err = c.ReadMessage(); err != nil {
				log.Println("read:", err)
				break
			}
			log.Printf("recv: %s", msg)

			if err = c.WriteMessage(mt, msg); err != nil {
				log.Println("write:", err)
				break
			}
		}

	}))

	log.Fatal(app.Listen(":3000"))
	// Access the websocket server: ws://localhost:3000/ws/123?v=1.0
	// https://www.websocket.org/echo.html
}
```
If you want you can use Postman for test purposes, in this post you can see an explanation about that: [Test WebSocket APIs With Postman](https://www.baeldung.com/postman-websocket-apis)

[Go WebSocket example](https://github.com/jackgris/go-websocket-example) This repository contains all the code. And you can see an explanation about how to run it and test the endpoint.

## Conclusion

WebSockets are a great solution for services that require continuous data exchange, such as instant messengers, online games, and real-time trading systems. When working with WebSockets in Go, it is important to consider advanced topics and optimizations, such as using channels for concurrency, using a message queue to manage traffic, optimizing I/O buffer sizes, using connection pooling, using message compression, and using load balancing. It is also important to consider issues such as typing, keepalive, and reconnect, and to be aware of solutions to those issues. In conclusion, Go provides an excellent platform for building WebSocket applications, offering simplicity, concurrency, performance, and a robust ecosystem of libraries and tools.

I hope this article is useful for you, see you in the next post.

### Sources

[RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455) / [Go Fiber WebSocket](https://github.com/gofiber/websocket) / [How to build websockets in Go](https://yalantis.com/blog/how-to-build-websockets-in-go/) / [Using WebSockets in Golang](https://blog.logrocket.com/using-websockets-go/) / [How to Manually Test WebSocket APIs](https://adequatica.medium.com/how-to-manually-test-websocket-apis-855393911d1a) / [Go Websocket Tutorial](https://tutorialedge.net/golang/go-websocket-tutorial/) / [Test WebSocket APIs With Postman](https://www.baeldung.com/postman-websocket-apis) / [Testing WebSockets for beginners](https://blog.scottlogic.com/2019/07/23/Testing-WebSockets-for-beginners.html) / [The WebSocket API (WebSockets)](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) / [WebSockets](https://www.fullstackpython.com/websockets.html) / [How Do Websockets Work?](https://sookocheff.com/post/networking/how-do-websockets-work/) / [What is Pub/Sub?](https://cloud.google.com/pubsub/docs/overview) / [How secure is WebSocket?](https://blog.passwork.pro/how-secure-is-websocket/) / [Scaling WebSockets](https://ably.com/topic/the-challenge-of-scaling-websockets) / [Building Hyper Scaling WebSockets based Web Application](https://medium.com/dataorc/building-hyper-scaling-websockets-based-web-application-348734d5308) / [How to scale WebSocket – horizontal scaling with WebSocket tutorial](https://tsh.io/blog/how-to-scale-websocket/) / [A Million WebSockets and Go](https://www.freecodecamp.org/news/million-websockets-and-go-cc58418460bb/) / [Scaling realtime apps with Golang and WebSockets](https://ably.com/topic/websockets-golang) / [Scaling Websocket](https://centrifugal.github.io/centrifugo/blog/scaling_websocket/)

