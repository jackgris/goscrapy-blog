---
title: "How to Write a Pub Sub Service With Go"
date: 2023-06-04T12:14:36-03:00
draft: false
tags: ["pubsub", "golang", "redis", "nats", "fiber", "channels"]
---

![pubsub_gopher logo](https://jackgris.github.io/goscrapy-blog/img/gopher-sending-a-message.png)


## An overview of the Pub/Sub pattern and how it works.

### What is Pub/Sub?

The `Pub/Sub` (publish/subscribe) pattern is a messaging pattern used in distributed systems to enable asynchronous communication between components or systems. It allows for the decoupling of senders (publishers) and receivers (subscribers) by introducing an intermediary called a message broker or message bus.

### How does its work?

![pubsub_flow](https://jackgris.github.io/goscrapy-blog/img/pubsub-flow.svg)


- `Publishers:` Publishers are responsible for sending messages to specific topics. A topic is a named channel or category to which messages can be published. Publishers don't need to know the identities or number of subscribers; they simply publish messages to the desired topic.

- `Subscribers:` Subscribers express interest in specific topics and receive messages published to those topics. They subscribe to topics they are interested in, and the message broker delivers messages to them. Subscribers can be one or more components or systems that want to consume the published messages.

- `Message Broker or Message Bus:` The message broker or message bus acts as an intermediary between publishers and subscribers. It receives messages published by publishers and delivers them to subscribers interested in the respective topics. The message broker ensures the reliable and efficient distribution of messages.

- `Asynchronous Communication:` Pub/Sub enables asynchronous communication between publishers and subscribers. Publishers publish messages to topics without knowing who, if anyone, will receive them immediately. Subscribers consume messages at their own pace, allowing them to process messages independently and asynchronously from the publishers.

- `Decoupling and Scalability:` Pub/Sub decouples publishers from subscribers, allowing them to evolve independently. Publishers and subscribers don't have to be aware of each other's existence or rely on direct communication. This loose coupling makes systems more scalable, as publishers can send messages to multiple subscribers without knowledge of their identities or numbers.

- `Reliable Message Delivery:` Pub/Sub systems ensure reliable message delivery. The message broker manages the delivery of messages to subscribers, handling issues like network failures, subscriber unavailability, and message persistence. Subscribers typically acknowledge the receipt of messages, allowing the message broker to track delivery status and handle failures.

- `Message Filtering and Routing:` These systems often provide mechanisms for message filtering and routing. Subscribers can express their interest in specific subsets of messages based on criteria such as message attributes, content, or routing rules. This allows for more targeted and efficient message delivery to relevant subscribers.

- `Message Ordering:` Depending on the Pub/Sub system, message ordering semantics can vary. Some systems guarantee "at least once" delivery, where messages are delivered to subscribers but may result in duplicates. Others provide "exactly once" delivery, ensuring that each message is delivered only once to subscribers.

- `Scalability and Elasticity:` These systems are designed to handle high message volumes and scale horizontally. They can accommodate varying numbers of publishers and subscribers without affecting performance. Additionally, Pub/Sub systems often support dynamic scaling, automatically adjusting resources based on workload demands.

- `Integration and Ecosystem:` These systems often integrate with other components and services in distributed systems. They can be combined with data analytics platforms, serverless computing frameworks, event-driven architectures, and more, providing a flexible and extensible ecosystem for building complex systems.

### When use Pub/Sub?

Pub/Sub is used in various scenarios where asynchronous messaging and decoupled communication between components or systems are required. For example:

- `Microservices Architecture:` Is well-suited for building microservices architectures. It enables individual services to communicate with each other without direct dependencies or knowledge of other services. Pub/Sub allows for loose coupling, scalability, and flexibility in integrating and orchestrating microservices.

- `Real-Time Data Processing:` Pub/Sub is ideal for handling real-time data processing and streaming scenarios. It allows systems to process and react to incoming data events in real-time. Applications such as real-time analytics, fraud detection, and monitoring systems can benefit from Pub/Sub by enabling immediate and event-driven processing of data.

- `Event-Driven Architectures:` Is a fundamental building block for event-driven architectures. It facilitates the flow of events within a system, where components or services react to events by subscribing to relevant topics. Event-driven architectures are well-suited for building scalable and responsive systems that can handle high volumes of events.

- `Internet of Things (IoT):` Pub/Sub is widely used in IoT applications. It enables devices and sensors to publish data to topics, allowing other components or systems to subscribe and consume that data. Provides a scalable and efficient way to handle large volumes of sensor data and enables real-time monitoring and control of IoT devices.

- `Distributed Systems:` Is employed in distributed systems where components or services need to communicate asynchronously. It allows for loose coupling and fault-tolerant communication between distributed components, enabling robust and scalable architectures.

- `Batch Processing and Workflows:` Can be used to manage batch processing and workflow systems. Publishers can publish job requests or tasks, and subscribers can consume those tasks to perform the necessary processing. Provides an efficient and scalable way to distribute tasks across multiple workers or processing nodes.

- `Cross-System Integration:` Is valuable for integrating different systems or services that may be developed independently or come from different vendors. It enables seamless communication and data exchange between systems, facilitating interoperability and extensibility.

- `Asynchronous Communication:` Well-suited for scenarios where asynchronous communication is beneficial. It allows systems to operate independently, sending and receiving messages at their own pace. Asynchronous communication enables better scalability, fault tolerance, and responsiveness in distributed systems.

- `Scalable Event Notification:` Can be used for scalable event notification systems. It enables broadcasting notifications to multiple subscribers efficiently, ensuring that all interested parties receive the notifications in a timely manner.

- `Cloud-Native and Serverless Architectures:` Pub/Sub is a key component in cloud-native and serverless architectures. It enables decoupled communication and event-driven workflows between various serverless functions or services, facilitating the building of highly scalable and responsive applications in the cloud.

## The benefits of using Pub/Sub over other messaging patterns.

When compared to other messaging patterns like point-to-point, request/reply, or message queues, Pub/Sub offers advantages such as decoupled communication, scalability, asynchronous processing, and flexible message routing. It allows for independent evolution of components, supports high message volumes, and provides fault tolerance, making it well-suited for building distributed systems and event-driven architectures.

## How to implement Pub/Sub in Go using channels and goroutines

We need to follow these basics steps:

1- Define a channel that will be used to communicate between the publishers and subscribers.

2- Create a goroutine that will publish messages to the channel.

3- Create one or more goroutines that will subscribe to the channel and receive the published messages.

4- Publish messages to the channel using the channel's <- operator.

5- Receive messages from the channel using the channel's <- operator.

Here is a basic example of how to implement Pub/Sub in Go using channels and goroutines:

```go
// Create a goroutine that will publish messages to the channel.
func publisher(wg *sync.WaitGroup, msgChan chan string) {
	for i := 0; i < 10; i++ {
		msgChan <- fmt.Sprintf("Message %d", i)
	}
	close(msgChan)
	wg.Done()
}

// Create one or more goroutines that will subscribe to the channel and receive the published messages.
func subscriber(id int, wg *sync.WaitGroup, msgChan chan string) {
	for message := range msgChan {
		fmt.Printf("Subscriber %d received message: %s\n", id, message)
	}
	wg.Done()
}

// In the main function, start the publisher and subscriber goroutines.
func main() {
	// Define a channel that will be used to communicate between publishers and subscribers.
	var msgChan = make(chan string)
	// We use a WaitGroup to coordinate goroutines
	wg := &sync.WaitGroup{}
	wg.Add(1)
	go publisher(wg, msgChan)
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go subscriber(i, wg, msgChan)
	}
	// Wait for all the subscribers to finish receiving messages.
	wg.Wait()
}
```

In this example, we first define a channel called `msgChan`. We then create a publisher goroutine that publishes 10 messages to the channel and closes the channel. We also create three subscriber goroutines that receive messages from the channel. Finally, we start the publisher and subscriber goroutines in the `main()` function and wait for all the subscribers to finish receiving messages using the `Wait` function of our `WaitGroup`.

This is just a basic example of how to implement `Pub/Sub in Go` using channels and goroutines. There are many other ways to implement Pub/Sub using different libraries and frameworks, such as Redis Pub/Sub or NATS. We will talk about that below.

## Here is one example similar a real applications

In this case, I create an interface and his implementation, you will see that I create a map that holds the topics and a list of channels. Every user that subscribes to one topic, will receive a channel. So when someone writes something in one topic, our implementation will re-send the same message to all the channels related to that topic.

```go
// PubSub Interface: Start by defining an interface that represents the Pub/Sub functionality.
// This interface should include methods for publishing messages to topics and subscribing to topics.
type PubSub interface {
	Publish(topic string, message interface{})
	Subscribe(topic string) <-chan interface{}
	Wait()
}

// PubSubImpl implement the PubSub interface. This struct will manage the topics, subscribers, and message distribution.
type PubSubImpl struct {
	waitGroup        sync.WaitGroup
	topics           map[string][]chan interface{}
	subscriptionLock sync.Mutex
}

// NewPubSub create a struct that implements the PubSub interface.
func NewPubSub() *PubSubImpl {
	return &PubSubImpl{
		topics: make(map[string][]chan interface{}),
	}
}

// Publish in the PubSubImpl struct, implement the Publish method to
// publish messages to a specific topic. This method will iterate over the subscribers of the topic
// and send the message through the corresponding channels.
func (ps *PubSubImpl) Publish(topic string, message interface{}) {
	ps.subscriptionLock.Lock()
	defer ps.subscriptionLock.Unlock()

	subscribers := ps.topics[topic]
	for _, subscriber := range subscribers {
		ps.waitGroup.Add(1)
		go func(subscriber chan interface{}) {
			msg := fmt.Sprintf("%s %v", topic, message)
			subscriber <- msg
			ps.waitGroup.Done()
		}(subscriber)
	}
}

// Subscribe in the PubSubImpl implement the Subscribe method to allow subscribers to subscribe to a topic.
// It creates a new channel for the subscriber and adds it to the list of subscribers for the specified topic.
func (ps *PubSubImpl) Subscribe(topic string) <-chan interface{} {
	ps.subscriptionLock.Lock()
	defer ps.subscriptionLock.Unlock()

	subscriber := make(chan interface{})
	ps.topics[topic] = append(ps.topics[topic], subscriber)

	return subscriber
}

// Wait will wait until all messages are published
func (ps *PubSubImpl) Wait() {
	ps.waitGroup.Wait()
}

var pubsub PubSub

// In this example, a message is published to "topic1", and the subscriber receives the message through the channel.
// You can have multiple subscribers to the same topic, and each subscriber will receive the published message independently.
// By leveraging channels and goroutines, you can achieve concurrent and asynchronous message distribution,
// enabling the Pub/Sub pattern in your Go application.
func main() {
	// Use the Pub/Sub implementation in your application by creating a new instance of PubSubImpl,
	// publishing messages, and subscribing to topics.
	pubsub = NewPubSub()

	// Subscribe to different topics
	subscriber1 := pubsub.Subscribe("topic1")
	subscriber2 := pubsub.Subscribe("topic2")
	subscriber3 := pubsub.Subscribe("topic3")
	subscriber4 := pubsub.Subscribe("topic3")
	subscriber5 := pubsub.Subscribe("topic3")

	// Publish a message to the topics
	pubsub.Publish("topic1", "Hello, subscribers number one!")
	pubsub.Publish("topic1", "Bye, subscribers number one!")
	pubsub.Publish("topic2", "Hello, subscribers number two!")
	pubsub.Publish("topic2", "How are you? subscribers number two!")
	pubsub.Publish("topic2", "Bye, subscribers number two!")
	pubsub.Publish("topic3", "Hello, subscribers number three!")
	pubsub.Publish("topic3", "How are you? subscribers number three!")
	pubsub.Publish("topic3", "Bye, subscribers number three!")

	// Receive messages from different topics
	go func() {
		for {
			select {
			case message := <-subscriber1:
				fmt.Println("subcriber 1", message)
			case message := <-subscriber2:
				fmt.Println("subcriber 2", message)
			case message := <-subscriber3:
				fmt.Println("subcriber 3", message)
			case message := <-subscriber4:
				fmt.Println("subcriber 4", message)
			case message := <-subscriber5:
				fmt.Println("subcriber 5", message)
			}

		}
	}()

	// Wait for all messages to be received by subscribers
	pubsub.Wait()
}
```
## Use cases for Pub/Sub, such as chat applications and real-time notifications

Now I will show you an example using our Pub/Sub implementation to create a real-time chat room. This is a silly example, but I think is useful to understand in deep these concepts. For that purpose I create a simple fron-end, you can see the code and the instruction to run it here: [chat example code](https://github.com/jackgris/go-pubsub-example/tree/main/chat).

### First our Pub/Sub implementation

The code is pretty similar to the code that we see before, but I made some modifications, like creating a new type for our messages:

```go
// Msg is the structure that will contain the necessary data for the chat.
type Msg struct {
	Id       string `json:"id"` // The ID in this example is necessary to identify the connection.
	Username string `json:"username"`
	Message  string `json:"message"`
}

// PubSub Interface: Start by defining an interface that represents the Pub/Sub functionality.
// This interface should include methods for publishing messages to topics and subscribing to topics.
type PubSub interface {
	Publish(topic string, message Msg)
	Subscribe(topic string) <-chan Msg
}
```
### Our server

And this is our server, this will receive all the messages from the chat application fron-end and will re-send all the messages to all the other users.

```go
// Create our server and configure the middleware that will help us to establish the WebSocket connection.
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

	// Initiate the Pub/Sub service created by us.
	var pubsub PubSub = NewPubSub()
	// In this case we only have one topic, one chat room.
	subscribe := pubsub.Subscribe("chat")
	// With this will save all the websocket connections that are active.
	connections := make(map[string]*websocket.Conn)

	app.Get("/ws/:id", websocket.New(func(c *websocket.Conn) {

		// Create an Id to identify the connection.
		id := uuid.New()
		// Save the connection in our pool. And remove the connection when the websocket is close.
		connections[id.String()] = c
		defer delete(connections, id.String())

		var (
			mt  int
			msg []byte
			err error
		)
		for {
			// We read everything we receive from the front-end.
			if mt, msg, err = c.ReadMessage(); err != nil {
				log.Println("read:", err)
				break
			}
			log.Printf("recv: %s - mt: %d", msg, mt)

			// We get the data from the message and publish that in our Pub/Sub service.
			m := Msg{}
			err := json.Unmarshal(msg, &m)
			if err != nil {
				log.Println(err)
				continue
			}
			m.Id = id.String() // add the Id to identify the owner
			pubsub.Publish("chat", m)
		}

	}))

	// Every time that receive a message, we will send the message to all the active connections.
	// For that we need the channel for communication and all the connections.
	go func(subs <-chan Msg, conn map[string]*websocket.Conn) {
		for {
			message := <-subs
			m, err := json.Marshal(message)
			if err != nil {
				log.Println("Error while marshal our message: ", err)
				continue
			}

			for _, c := range conn {
				if err = c.WriteMessage(1, m); err != nil {
					log.Printf("Error while write to ID: %s with err: %s\n", message.Id, err)
				}
			}
		}
	}(subscribe, connections)

	log.Fatal(app.Listen(":8080"))
```

If you want to run the code, remember follow the instructions here: [Chat example code](https://github.com/jackgris/go-pubsub-example/tree/main/chat)

Below you will see a couple of examples using Redis and Nats.

## How to use Pub/Sub with Redis?

This is a basic example using Redis, the main idea is we have a server that receives a POST request with a message and the server will share all the messages with the clients that are connected.

### Simple server code

In our main function, the code that will receive and send the message to the clients:

```go
        rdb := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       0,  // use default DB
	})
	ctx := context.Background()

	// There is no error because go-redis automatically reconnects on error.
	pubsub := rdb.Subscribe(ctx, "send-user-data")
	// Close the subscription when we are done.
	defer pubsub.Close()

	ch := pubsub.Channel()

	app := fiber.New()

	app.Get("/", func(c *fiber.Ctx) error {
		return c.SendString("Hello there 👋")
	})

	app.Post("/", func(c *fiber.Ctx) error {
		user := new(User)

		if err := c.BodyParser(user); err != nil {
			log.Println("Body Parse: ", err)
			return err
		}

		payload, err := json.Marshal(user)
		if err != nil {
			return err
		}

		if err := rdb.Publish(ctx, "send-user-data", payload).Err(); err != nil {
			return err
		}

		return c.SendStatus(200)
	})

	go func(ch <-chan *redis.Message) {
		for msg := range ch {
			fmt.Println(msg.Channel, msg.Payload)
		}
	}(ch)

	log.Println(app.Listen(":3000"))
```

### Client code

This code in the client will receive all the messages from our server:

```go
        ctx := context.Background()
	redisClient := redis.NewClient(&redis.Options{
		Addr: "localhost:6379",
	})

	subscriber := redisClient.Subscribe(ctx, "send-user-data")

	user := User{}

	for {
		msg, err := subscriber.ReceiveMessage(ctx)
		if err != nil {
			panic(err)
		}

		if err := json.Unmarshal([]byte(msg.Payload), &user); err != nil {
			panic(err)
		}

		log.Printf("Received message from %s channel.", msg.Channel)
		log.Printf("%+v\n", user)
	}
```

If you want to run the code, remember follow the instructions here: [Redis example code](https://github.com/jackgris/go-pubsub-example/tree/main/redis)


## Sample code in Go on how to use Nats Pub/Sub with JetStream

In this example I'll use Docker to run the Nats server, you can see the instruction in the repository for that. But also you can install it on your PC. Basically, I create a server that will receive all the messages that the clients send to the Nats server. And I wrote a client that sends 500 messages to the Nats server. Here is the code:

### Server code

```go
func main() {

	// Connect to a server
	nc, err := nats.Connect(nats.DefaultURL)
	if err != nil {
		fmt.Println(err)
	}

	stop := make(chan os.Signal, 1)
	signal.Notify(stop, os.Interrupt)
	var shutdown sync.WaitGroup
	shutdown.Add(1)

	// Create JetStream Context
	js, err := nc.JetStream(nats.PublishAsyncMaxPending(256))
	if err != nil {
		fmt.Println("Error jetstream subscriber: ", err)
	}

	// Create a Stream
	_, err = js.AddStream(&nats.StreamConfig{
		Name:     "ORDERS",
		Subjects: []string{"ORDERS.*"},
	})
	if err != nil {
		fmt.Println("Error JetStream add stream: ", err)
	}

	ch := make(chan *nats.Msg, 64)

	// Simple Async Ephemeral Consumer
	_, err = js.Subscribe("ORDERS.*", func(m *nats.Msg) {
		ch <- m
	})
	if err != nil {
		fmt.Println("Error subscribe: ", err)
	}

	go func() {
		for {
			select {
			case msg := <-ch:
				fmt.Printf("Received a message: %s\n", string(msg.Data))
			case <-stop:
				shutdown.Done()
				break
			}
		}
	}()

	shutdown.Wait()
}
```

### Client

```go
func main() {
	// Connect to NATS
	nc, err := nats.Connect(nats.DefaultURL)
	if err != nil {
		fmt.Println("Error while connect: ", err)
	}

	// Create JetStream Context
	js, err := nc.JetStream(nats.PublishAsyncMaxPending(256))
	if err != nil {
		fmt.Println("Error JetStream Context: ", err)
	}

	// Simple Async Stream Publisher
	for i := 0; i < 500; i++ {
		hello := fmt.Sprintf("Hello number %d", i)
		_, err = js.PublishAsync("ORDERS.scratch", []byte(hello))
		if err != nil {
			fmt.Println("Error publishing: ", err)
		}
	}

	select {
	case <-js.PublishAsyncComplete():
	case <-time.After(5 * time.Second):
		fmt.Println("Did not resolve in time")
	}
}
```

In this example, I use JetStream which is the NATS persistence engine providing streaming, message, and worker queues with At-Least-Once semantics. JetStream stores messages in streams. A stream defines how messages are stored and limits such as how long they persist or how many to keep. To store a message in JetStream, you simply need to publish to a subject that is associated with a stream.

If you want to run the code, remember follow the instructions here: [Nats example code](https://github.com/jackgris/go-pubsub-example/tree/main/nats)

## Beyond Pub/Sub: additional features that can be added to provide more functionality:

- `Authentication:` could be extended to support authentication, which would allow to control who has access to publish and subscribe to messages. This would be useful for applications that need to protect sensitive data.

- `Message retrieval:` could be extended to support message retrieval, which would allow searching for and retrieving specific messages. This would be useful for applications that need to audit or investigate past events.

- `Message filtering:` could be extended to support message filtering, which would allow filtering messages based on their content. This would be useful for applications that need to process large volumes of data and only need to process specific types of messages.

- `Message retry:` Pub/Sub could be extended to support message retry, which would allow configuring Pub/Sub to automatically retry publishing or subscribing to messages that fail for some reason. This would be useful for applications that need to ensure that all messages are delivered, even if there are temporary network or server errors.

- `Message batching:` Pub/Sub could be extended to support message batching, which would allow developers to send multiple messages as a single batch. This would reduce the number of API calls that need to be made, which could improve performance and reduce costs.

These are just a few ideas for additional features that could be added to Pub/Sub.

## Tips for optimizing Pub/Sub performance and scalability in Go-based systems.

1- `Use the right message size.` The size of the messages you publish to Pub/Sub can have a big impact on performance. Larger messages can take longer to publish and receive, and they can also use more storage space. If you can, try to keep your messages as small as possible.

2- `Use the right number of publishers and subscribers.` The number of publishers and subscribers you have can also affect performance. Too many publishers or subscribers can overload the Pub/Sub system, which can lead to delays in message delivery. If you have a large number of publishers or subscribers, you may need to increase the number of Pub/Sub nodes in your cluster.

3- `Use message batching.` Message batching can help to improve performance by reducing the number of API calls that need to be made. When you batch messages, Pub/Sub will deliver them as a single message, which can improve performance and reduce costs.

4- `Use a load balancer.` A load balancer can help to improve performance by distributing traffic across multiple Pub/Sub nodes. This can help to prevent any single node from becoming overloaded.

5- `Use a caching layer.` A caching layer can help to improve performance by storing frequently accessed messages in memory. This can reduce the number of times that Pub/Sub needs to be accessed, which can improve performance.

6- `Monitor your system.` It is important to monitor your system to ensure that it is performing as expected. You can use the Pub/Sub monitoring dashboard to view metrics such as message throughput, latency, and error rates. This information can help you to identify any performance bottlenecks and take steps to improve performance.

These are just a few tips by following these tips, you can improve the performance and scalability of your applications.

## Conclusion

In conclusion, Pub/Sub is a powerful messaging pattern that can be used to decouple applications, increase scalability, and improve performance. It is a simple yet effective way to send messages between applications, and it can be used in a variety of different scenarios. Is a valuable tool for building distributed systems, microservices architectures, real-time data processing pipelines, and event-driven applications. If you are looking for a way to improve the communication between your applications, Pub/Sub is a great option to consider.

I hope this article is useful for you, see you in the next post.

### Sources

[Guide to Pub/Sub in Golang](https://ably.com/blog/pubsub-golang)

[Building a pub/sub service in Go](https://blog.logrocket.com/building-pub-sub-service-go/)

[Implementing Pub-Sub in Golang](https://www.linkedin.com/pulse/implementing-pub-sub-golang-sakshyam-ghimire/)

[What is Pub/Sub?](https://cloud.google.com/pubsub/docs/overview)

[Publish-Subscribe Design Pattern: Introduction to Scalable Messaging](https://hackernoon.com/publish-subscribe-design-pattern-introduction-to-scalable-messaging-781k3tae)

[Publish-Subscribe: Introduction to Scalable Messaging](https://thenewstack.io/publish-subscribe-introduction-to-scalable-messaging/)

[Pub/Sub use cases: When to use the Pub/Sub pattern](https://ably.com/blog/pub-sub-pattern-examples)

[How to use Redis Pub/Sub in Go Chat Application](https://www.whichdev.com/using-redis-pub-sub-in-go-chat-application-part-3/)

[Using Redis Pub/Sub with Golang](https://dev.to/franciscomendes10866/using-redis-pub-sub-with-golang-mf9)

[Redis Go client](https://github.com/redis/go-redis)

[Golang client for NATS, the cloud native messaging system.](https://github.com/nats-io/nats.go)
