---
title: "Graceful Shutdown"
date: 2023-04-03T18:58:13-03:00
draft: false
---

## How to implement a graceful shutdown in Go?

  You need to start thinking about these kinds of questions when creating a server that needs to maintain a state.   For example, if you have a web server running in one AWS EC2 instance and you fix some security issues and want to update the server.
    If you don't manage correctly maybe you can lose the data that never was saved in the database or can provoke some bad user experience on every update because lose all connections without any response to the requests because you don't wait to finish sending the response. Also, if you keep receiving requests until the server finishes the shutdown it will be impossible to manage those connections and never have to forget all the issues that you can get in some internal operations when you don't save data in the right way.


#### To shutdown gracefully a server you need to stop after:

- All pending processes (requests, loops) are to be completed - no new processes should start and no new requests should be accepted.
- Need close all open connections to external services you are consuming and databases.

In a few words, you need to try to make sure that everything is predictable, loss of data is bad during a shutdown.
Those means blocking the main go routine without waiting on anything or `Calling os.Exit(1)` (is similar to `SIGKILL`), while other go routines are still running, are bad ideas.

#### What do we need to understand to make right?

The most important task to make these is first know how to wait until all goroutines end and how to propagate the termination signal to all of them.

&nbsp;

### Wait to finish.

Let's see what options we have available on waiting goroutines.

#### Using channels.

With channel primitive we can create an empty struct channel `make(chan struct{}, 1)` (empty struct requires no memory).
Every child go routine should shared to the channel when it is done (defer can be useful here). The parent go routine should consume from the channel as many times as the expected go routines.
Note: This is mostly useful when waiting on a single go routine.

Example:
```go
func run(ctx) {
        wait := make(chan struct{}, 1)

	go func() {
		defer func() {
	            wait <- struct{}{}
		}()
		for {
			select {
	                case <-ctx.Done():
				fmt.Println("Break the loop")
				break;
                        case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	}()

	go func() {
		defer func() {
	           wait <- struct{}{}
		}()
		for {
			select {
	                case <-ctx.Done():
				fmt.Println("Break the loop")
				break;
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

	// wait for two goroutines to finish
	<-wait
	<-wait

	fmt.Println("Main done")
}
```

#### With WaitGroup

The channel solution can be a bit ugly with many go routines. So using `sync.WaitGroup` from standard library package, can be a more idiomatic way to achieve the above.

Example:

```go
func run(ctx) {
    var wg sync.WaitGroup

    wg.Add(1)
    go func() {
		defer wg.Done()
		for {
			select {
                        case <-ctx.Done():
				fmt.Println("Break the loop")
				return;
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	}()

    wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
	                case <-ctx.Done():
				fmt.Println("Break the loop")
				return;
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

    wg.Wait()
    fmt.Println("Main done")
}
```

#### With errgroup

`sync/errgroup` package exposes two methods .Go and .Wait that are more readable and easier to maintain in comparison to WaitGroup. In addition, it does error propagation and cancels the context in order to terminate the other go-routines in case of an error.
Example:
```go
func run(ctx) {
	g, gCtx := errgroup.WithContext(ctx)
	g.Go(func() error {
		for {
			select {
                        case <-gCtx.Done():
				fmt.Println("Break the loop")
				return nil;
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	})

	g.Go(func() error {
		for {
			select {
                        case <-gCtx.Done():
				fmt.Println("Break the loop")
				return nil;
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

        err := g.Wait()
	if err != nil {
		fmt.Println("Error group: ", err)
	}
	fmt.Println("Main done")
}
```

&nbsp;

### Terminating a process

Even if we have figured out how to properly communicate the state of processes and wait for them, we still have to implement termination. Let's see how this can be done with some examples, introducing all the necessary Go primitives.

#### With signal handling

To listen for an OS signal to stop the progress we need to use `os.Interrupt` to gracefully shutdown on `Ctrl+C` which is `SIGINT`.
`syscall.SIGTERM` is the usual signal for termination and the default one (at least for now) for docker containers, which is also used by kubernetes.

Example:
```go 
// buffer size is 1 because need not block the notifier
exit := make(chan os.Signal, 1) 
signal.Notify(exit, os.Interrupt, syscall.SIGTERM)
```
### Breaking the loop after capturing signals.

#### Non-Blocking Channel Select
`select` gives you the ability to consume from multiple channels in each case.

Example:
```go
func main() {
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt, syscall.SIGTERM)
	// we'll print in every loop after a minute while wait for signal interrupt on channel c
	for {
		select {
                case <-c:
			fmt.Println("Break the loop")
			return;
		case <-time.After(1 * time.Second):
			fmt.Println("Hello in a loop")
		}
	}
}
```

### How to do it using Context

`Context` is a useful interface in go, that should be used and propagated in all blocking functions. It enables the propagation of cancelation throughout the program. It is considered good practice for `ctx context.Context` to be the first argument in every method or function that is used directly or indirectly for external dependencies.

#### Channel sharing issue

Let's examine how context properties could help in a more complex situation. Having multiple loops running in parallel, using channels (counter-example):

Example: (Warning!, don't use this code)
```go
func main() {
	exit := make(chan os.Signal, 1)
	signal.Notify(exit, os.Interrupt, syscall.SIGTERM)
	
        // This will not work as expected!!
	var wg sync.WaitGroup

	wg.Add(1)
        go func() {
		defer wg.Done()
		for {
			select {
	                case <-exit: // Only one go routine will get the termination signal
				fmt.Println("Break the loop: hello")
				break;
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	}()

	wg.Add(1)
        go func() {
		defer wg.Done()
		for {
			select {
	                case <-exit: // Only one go routine will get the termination signal
				fmt.Println("Break the loop: ciao")
				break;
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

	wg.Wait()
	fmt.Println("Main done")
}
```

Why this not going to work? Go channels do not work in a broadcast way, only one go routine will receive a single os.Signal. Also, there is no guarantee which go routine will receive it.

#### Using Context for termination

Let's try to fix this problem by introducing `context.WithCancel`

```go
func main() {
        ctx, cancel := context.WithCancel(context.Background())

	go func() {
		exit := make(chan os.Signal, 1)
		signal.Notify(c, os.Interrupt, syscall.SIGTERM)
		cancel()
	}()

	var wg sync.WaitGroup

	wg.Add(1)
        go func() {
		defer wg.Done()
		for {
			select {
	                case <-ctx.Done():
				fmt.Println("Break the loop")
				break;
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	}()

	wg.Add(1)
        go func() {
		defer wg.Done()
		for {
			select {
	                case <-ctx.Done():
				fmt.Println("Break the loop")
				break;
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

	wg.Wait()
	fmt.Println("Main done")
}
```

Essentially `cancel()` is broadcasted to all the go-routines that call `.Done()`. The returned context's Done channel is closed when the returned cancel function is called or when the parent context's Done channel is closed, whichever happens first.

#### NotifyContext

In go 1.16 a new helpful method was introduced in signal package, `singal.NotifyContext`:

```go
func NotifyContext(parent context.Context, signals ...os.Signal) (ctx context.Context, stop context.CancelFunc)
```
`NotifyContext` returns a copy of the parent context that is marked done (its Done channel is closed) when one of the listed signals arrives, when the returned stop function is called, or when the parent context's Done channel is closed, whichever happens first.

Example:

```go
func main() {
        ctx, stop := context.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
        defer stop()

	var wg sync.WaitGroup

	wg.Add(1)
        go func() {
		defer wg.Done()
		for {
			select {
	                case <-ctx.Done():
				fmt.Println("Break the loop")
				break;
			case <-time.After(1 * time.Second):
				fmt.Println("Hello in a loop")
			}
		}
	}()

	wg.Add(1)
        go func() {
		defer wg.Done()
		for {
			select {
	                case <-ctx.Done():
				fmt.Println("Break the loop")
				break;
			case <-time.After(1 * time.Second):
				fmt.Println("Ciao in a loop")
			}
		}
	}()

	wg.Wait()
	fmt.Println("Main done")
}
```

### Common libraries

#### HTTP server

The examples above included a for loop for simplification, but let's examine something more practical.

During a non-graceful shutdown, inflight HTTP requests could face the following issues:

- They never get a response back, so they timeout.
- Some progress has been made, but it is interrupted halfway, causing a waste of resources or data inconsistencies if transactions are not used properly.
- A connection to an external dependency is closed by another go routine, so the request can not progress further.

Having your HTTP server shutting down gracefully is really important. In a cloud-native environment services/pods shutdown multiple times within a day either for autoscaling, applying a configuration, or deploying a new version of a service. Thus, the impact of interrupted or timeout requests can be significant in the service's SLAs. Fortunately, go provides a way to gracefully shutdown an HTTP server.

Example:
```go
func main() {

	ctx, cancel := context.WithCancel(context.Background())

	go func() {
		c := make(chan os.Signal, 1) // we need to reserve to buffer size 1, so the notifier are not blocked
		signal.Notify(c, os.Interrupt, syscall.SIGTERM)

		<-c
		cancel()
	}()

	db, err := repo.SetupPostgresDB(ctx, getConfig("DB_DSN", "root@tcp(127.0.0.1:3306)/service"))
	if err != nil {
		panic(err)
	}

	httpServer := &http.Server{
		Addr:    ":8000",
	}

	g, gCtx := errgroup.WithContext(ctx)
	g.Go(func() error {
		return httpServer.ListenAndServe()
	})
	g.Go(func() error {
		<-gCtx.Done()
		return httpServer.Shutdown(context.Background())
	})

	if err := g.Wait(); err != nil {
		fmt.Printf("exit reason: %s \n", err)
	}
}
```
We are using two go routines:

- run `httpServer.ListenAndServe()` as usual
- wait for `<-gCtx.Done()` and then call `httpServer.Shutdown(context.Background())`

It is important to read the package documentation in order to understand how this works: "Shutdown gracefully shuts down the server without interrupting any active connections."

Nice, but how? Shutdown works by first closing all open listeners, then closing all idle connections, and then waiting indefinitely for connections to return to idle and then shut down.

Why do I have to provide a context? If the provided context expires before the shutdown is complete, Shutdown returns the context's error, otherwise it returns any error returned from closing the Server's underlying Listener(s).

In the example, we chose to provide `context.Background()` which has no expiration.

#### Canceling long running requests

When `.Shutdown` is method is called the serve stop accepting new connections and it waits for existing once to finish before `.ListenAndServe()` may return. There are cases where http requests require quite a long time to be terminated. So, what is the best way to terminate those gracefully and not hang waiting for them to finish?

First of all you need to extract the context from `http.Request` `ctx := req.Context()` and use this context to terminate your long running process. Use `BaseContext` (introduced in go1.13), to pass your main ctx as the context in every request. `BaseContext` optionally specifies a function that returns the base context for incoming requests on this server.
The provided Listener is the specific Listener that's about to start accepting requests.
If `BaseContext` is nil, the default is `context.Background()`.
If non-nil, it must return a non-nil context.

Example:

```go
func main() {
	mainCtx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
	defer stop()

	httpServer := &http.Server{
		Addr: ":8000",
		Handler: http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			ctx := r.Context()

			for {
				select {
				case <-ctx.Done():
					fmt.Println("Graceful handler exit")
					w.WriteHeader(http.StatusOK)
					return
				case <-time.After(1 * time.Second):
					fmt.Println("Hello in a loop")
				}
			}
		}),
		BaseContext: func(_ net.Listener) context.Context {
			return mainCtx
		},
	}
	g, gCtx := errgroup.WithContext(mainCtx)
	g.Go(func() error {
		return httpServer.ListenAndServe()
	})
	g.Go(func() error {
		<-gCtx.Done()
		return httpServer.Shutdown(context.Background())
	})

	if err := g.Wait(); err != nil {
		fmt.Printf("exit reason: %s \n", err)
	}
}
```

#### HTTP Client

Go standard libraries provides a way to pass a context when making an HTTP request: `NewRequestWithContext`

Without ctx:
```go
resp, err := netClient.Post(uri, "application/json; charset=utf-8",
					bytes.NewBuffer(payload))
```

And the equivalent with passing ctx:

```go
req, err := http.NewRequestWithContext(ctx, "POST", uri, bytes.NewBuffer(payload))
if err != nil {
			return err
}
req.Header.Set("Content-Type", "application/json; charset=utf-8")

resp, err := netClient.Do(req)
```
The following techniques are necessary for more advanced use cases. For instance, if you are using a pool of workers or you have a chain of component dependencies that need to shutdown in order.

### Draining Worker Channels
When you have worker go routines that are consuming/producing from/to a channel, special care must be taken to make sure no items are left in the channels when the process shuts down. To do this we need to utilize go close method on the channel.

You need to remember two things about closing a channel. Writing to a close channel will result in a `panic`. When reading for a channel, you can use value, `ok <- ch`. Reading from a close channel will return all the buffered items. Once the buffer items are "drained", the channel will return zero value and ok will be false. Note: While the channel still has items ok will be true.
    Alternative you can do a range on the channel `for value := range ch {`. In this case the for loop will stop when no more items are left on the channel and the channel is closed. This is much prettier than the approach above, but not always possible.

If you have a single worker writing to the channel, close the channel once you are done.
Example:
```go
go func() {
    defer close(ch) // close after write is no longer possible
    for {
	  	select {
		case <-ctx.Done():
		return
		...
                ch <- value // write to the channel only happens inside the loop
    }
}()
```

If you have multiple workers writing to the same channel, close the channel after waiting for all workers to finish.
Example:

```go
g, gCtx := errgroup.WithContext(ctx)
ch = make(...) // channel will be written from multiple workers
for w := range workers { // create n number of workers
  g.Go(func() error {
		return w.Run(ctx, ch) // workers will publish
    })
}
g.Wait() // we need to wait for all workers to stop
close(ch) // and then close the channel
```

If you're reading from a channel, exit only when the channel has no more data. Essentially it's the responsibility of the writer to stop the readers, by closing the channel.
Example:

```go
for v := range ch {

}

// or
for {
   select {
      case v, ok <- ch:
         if !ok { // nothing left to read
           return;
         }
	 foo(v) // process `v` normally
      case ...:
					...
   }
}
```

If a worker is both reading and writing, the worker should stop when the channel that it is reading from has no more data, and then close the writer.

### Graceful methods

We have seen several techniques so far for gracefully terminating a piece of long running code. It is also useful to examine how components can expose exported methods that can be called and then facilitate gracefully shutdown.

#### Blocking with ctx

This is the most common approach and the easier to understand and implement.

- You call a method
- You pass it a context
- The method blocks
- It returns in case of an error or when context is cancelled / timeout.

```go
// calling:
err := srv.Run(ctx, ...)

// implementation
func (srv *Service) Run(ctx context.Context, ...) {
	...
	...
	for {
		...
                select {
			case <- ctx.Done()
				return ctx.Err() // Depending on our business logic,
                                //   we may or may not want to return a ctx error:
				//   https://pkg.go.dev/context#pkg-variables
		 }
	}
```

### Setup/Shutdown
There are cases when blocking with ctx code is not the best approach. This is the case when we want greater control over when `.Shutdown` happens. This approach is a bit more complex and there is also the danger of people forgetting to call `.Shutdown`.

#### Use case
The code bellow demonstrates why this pattern might be useful. We want to make sure that db Shutdown happens only after the Service is no longer running, because the Service is depending on the database to run for it to work. By calling `db.Shutdown()` on defer, we ensure it runs after `g.Wait` returns.

Example:

```go
// calling:
func () {
   err := db.Setup() // will not block
   defer db.Shutdown()

   svc := Service{
     DB: db
   }

   g.Run(...
     svc.Run(ctx, ...)
   )
   g.Wait()
}
```

Implementation example

```go
type Database struct {
  ...
  cancel func()
  wait func() err
}

func (db *Database) Setup() {
  // ...
  // ...

  ctx, cancel := context.WithCancel(context.Background())
  g, gCtx := errgroup.WithContext(ctx)

  db.cancel = cancel
  db.wait = g.Wait

  for {
	...
    select {
	case <- ctx.Done()
	  return ctx.Err() // Depending on our business logic,
	  //   we may or may not want to return a ctx error:
	  //   https://pkg.go.dev/context#pkg-variables
		 }
	}
}

func (db *Database) Shutdown() error {
	db.cancel()
	return db.wait()
}
```
&nbsp;

### Another and final example using Fiber framework.


```go
package main

import (
	"os"
	"os/signal"
	"sync"
	"time"

	"github.com/gofiber/fiber/v2"
	"github.com/jackgris/goscrapy/config"
	"github.com/jackgris/goscrapy/database"
	"github.com/sirupsen/logrus"
)

var err error
var setup config.Data

func main() {
	log := logrus.New()
	log.SetLevel(logrus.InfoLevel)
	log.SetFormatter(&logrus.TextFormatter{
		FullTimestamp:          true,
		TimestampFormat:        "2006-01-02 15:04:05",
		ForceColors:            true,
		DisableLevelTruncation: true,
	})

	// Getting all config needed for connections and pages login
	setup = config.Get("../../data.env", log)

	// Starting DB connection
	_, err = database.Connect(...)
	if err != nil {
		panic("Error database connection: " + err.Error())
	}
	defer database.Disconnect()

	app := fiber.New()

	// Will wait for signal interrupt, to wait for a while and clean all the pending tasks.
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt)

	var serverShutdown sync.WaitGroup

	go func() {
		<-c
		log.Warn("Gracefully shutting down...")
		serverShutdown.Add(1)
		defer serverShutdown.Done()
                // We add a timeout, after signal Interrupt we will wait for our
                // server end process or in worst case for 1 minute
		_ = app.ShutdownWithTimeout(60 * time.Second)
	}()

	app.Route("/", routes, "main")
	if err := app.Listen(":3000"); err != nil {
		log.Fatal(err)
	}

	// Waiting for start shutting down
	serverShutdown.Wait()
	log.Warn("Running cleanup tasks...")
}
```

### Closing this post

Terminating your long-running services gracefully is an important pattern that you will have to implement sooner or later. This is especially true for systems that act as middlewares where many connections to external services exist and high volumes of data are handled concurrently. Go offers all the tools we need to implement this pattern, and selecting the right ones depends a lot on your use case. I hope this will helpful to anyone who want to implement it.
