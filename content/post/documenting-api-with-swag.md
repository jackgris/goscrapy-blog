---
title: "Documenting Api With Swag"
date: 2023-04-09T19:23:48-03:00
draft: true
tags: ["api", "swagger", "documentation", "curl", "golang"]
---

Think on this, you’ve finished developing a new API, and now have to write documentation to guide you when building client-side applications that consume the API. You start thinking of various ways to achieve this, and you lay out multiple alternatives like Swagger, Docusaurus, Postman and many more.
You remember all the work involved in the API documentation phase and wonder if there are shortcuts to speed things up. You need to do this because who will use an API without any documentation?

One great tool for creating API documentation is Swagger because of its ease of creating, maintaining, and publishing API documentation. Swagger is a professional, open source toolset that helps users, teams, and enterprises easily create and document APIs at scale.

#### Some benefits of using Swagger in your next project include:

- Allowing you to create, maintain, and publish API documentation quickly and easily.
- Generating interactive documentation that allows you to validate and test API endpoints from your browser without third-party software.
- Easily understandable by developers and non-developers
- Functionality to generate API client libraries (SDKs) for various languages and frameworks directly from an OpenAPI specification.

Here you will learn how to create Swagger documentation for Go web APIs directly from the source code using annotations and Swag. In this article, we will build a demo web API with Go and Fiber, then create documentation for it using Swag.

## Build a demo Go web API

Fiber is an Express inspired web framework built on top of Fasthttp, the fastest HTTP engine for Go. Designed to ease things up for fast development with zero memory allocation and performance in mind.

Now let’s build the web API for a basic “to do” application.

##### Step 1: Set up your development environment

Create a new Go project in your text editor or IDE and initialize your go.mod file. You are free to use any name for your package:

```bash
go mod init go-api-with-swarg
```

##### Step 2: Install Fiber

Install the Fiber web framework in your project. In the terminal, type the following:
```bash
go get -u github.com/gofiber/fiber/v2
```

##### Step 3: Set up a Gin server

Create a file named main.go and save the following code in it:
```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "net/http"
    "log"
)

func main() {
        // configure the Fiber server
        app := fiber.New()

        // run the Fiber server
        log.Fatal(app.Listen(":3000"))
}
```
##### Step 4: Create the getAllTodos route

Let’s create a todo type and seed the list with some data. Add the following code to the main.go file:
```go
// todo represents data about a task in the todo list
type todo struct {
        ID   string `json:"id"`
        Task string `json:"task"`
}

// message represents request response with a message
type message struct {
        Message string `json:"message"`
}

// todo slice to seed todo list data
var todoList = []todo{
        {"1", "Learn Go"},
        {"2", "Build an API with Go"},
        {"3", "Document the API with swag"},
}
```
Create a route handler that will accept a GET request from the client then return all the items in the to do list.

Add the following code to the main.go file:

```go
func getAllTodos(c *fiber.Ctx) error {
	c.Status(http.StatusOK)
	return c.JSON(todoList)
}
```

Register the getAllTodos handler to the Fiber router. Update the main function in main.go with the following code:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "net/http"
    "log"
)

func main() {
        // configure the Fiber server
        app := fiber.New()
        app.Get("/todo", getAllTodos)

        // run the Fiber server
        log.Fatal(app.Listen(":3000"))
}
```

Test the getAllTodos route by running the Fiber server and making a request via curl like so:

```bash
go run main.go
```
```bash
curl -X 'GET' \
  'http://localhost:3000/todo' \
  -H 'accept: application/json'
```

##### Step 5: Create the getTodoByID route

Create a route handler that will accept a GET request from the client and a todo ID, then return the details of the associated item from the todo list.

Add the following code to the main.go file:
```go
func getTodoByID(c *fiber.Ctx) error {
	ID := c.Params("id")

	// loop through todoList and return item with matching ID
	for _, todo := range todoList {
		if todo.ID == ID {
			c.Status(http.StatusOK)
			return c.JSON(todo)
		}
	}

	// return error message if todo is not found
	r := message{"todo not found"}
	c.Status(http.StatusNotFound)
	return c.JSON(r)
}
```

Register the getTodoById handler to the Fiber router. Add the following code to the router configuration in main.go:

```go
app.Get("/todo/:id", getTodoByID)
```

Test the getTodoById route by making a request via Postman like so:

##### Step 6: Create the createTodo route

Create a route handler that will accept a POST request from the client with a todo ID and task, then add a new item to the todo list.

Add the following code to the main.go file:

```go
func createTodo(c *fiber.Ctx) error {
	var newTodo todo

	// bind the received JSON data to newTodo
	if err := c.BodyParser(&newTodo); err != nil {
		r := message{"an error occurred while creating todo"}
		c.Status(http.StatusBadRequest)
		return c.JSON(r)
	}

	// add the new todo item to todoList
	todoList = append(todoList, newTodo)
	c.Status(http.StatusCreated)
	return c.JSON(newTodo)
}
```

Register the createTodo handler to the Fiber router. Add the following code to the router configuration in `main.go`:
```go
app.Get("/todo", getAllTodos)
```

Test the createTodo route by making a request via curl like so:
```bash
curl -X 'GET' \
  'http://localhost:3000/todo' \
  -H 'accept: application/json'
```

##### Step 7: Create the deleteTodo route

Create a route handler that will accept a DELETE request from the client along with a todo ID, then remove the associated item from the todo list. Add the following code to the `main.go` file:

```go
func deleteTodo(c *fiber.Ctx) error {
	ID := c.Params("id")

	// loop through todoList and delete item with matching ID
	for index, todo := range todoList {
		if todo.ID == ID {
			todoList = append(todoList[:index], todoList[index+1:]...)
			r := message{"successfully deleted todo"}
			c.Status(http.StatusOK)
			return c.JSON(r)
		}
	}

	// return error message if todo is not found
	r := message{"todo not found"}
	c.Status(http.StatusNotFound)
	return c.JSON(r)
}
```
Register the deleteTodo handler to the Fiber router. Add the following code to the router configuration in `main.go`:

```go
app.Delete("/todo/:id", deleteTodo)
```
Test the deleteTodo route by making a request via `curl` like so:
```bash
curl -X 'DELETE' \
  'http://localhost:3000/todo/5' \
  -H 'accept: application/json'
```

### Document the web API with Swag

Swag is middleware that helps to automatically generate RESTful API documentation with Swagger 2.0 for Go directly from source code using annotations. It requires you to specify how your routes work and automates the entire Swagger documentation creation process.

Swag is compatible with many Go web frameworks and has various integrations for them. This tutorial will use the Fiber integration.
#### Step 1: Install Swag

Install the Swag package in your project. In the terminal, type:
```bash
go get -u github.com/swaggo/swag/cmd/swag
go get -u github.com/swaggo/fiber-swagger
go get -u github.com/swaggo/files
```

#### Step 2: Initialize Swag

Initialize Swag in your project. In the terminal, type:
```bash
swag init
```

This will make Swag parse your annotations and generate the Swagger documentation for your code into the newly created docs folder.

If your terminal does not recognize `swag init` when executed, you need to add the Go bin folder to PATH.

#### Step 3: Import the Swag package into your project

Update the imports in the main.go file with the code below:

```go
import (
	"log"
	"net/http"

	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/swagger"
	_ "github.com/jackgris/go-api-with-swarg/docs"
)
```

#### Step 4: Add general API annotations to the code

The General API annotations contain basic information about the API documentation (title, description, version, contact info, host, and license).

Add the following set of annotations to the `main.go` file (preferably before the main function):

```go
//	@title			Go + Fiber Todo API
//	@version		1.0
//	@description	Sample todo server. You can visit the GitHub repository at https://github.com/jackgris/go-api-with-swarg

//	@contact.name	API Support
//	@contact.url	http://www.swagger.io/support
//	@contact.email	support@swagger.io

//	@license.name	Apache 2.0
//	@license.url	http://www.apache.org/licenses/LICENSE-2.0.html

//	@host						localhost:3000
//	@BasePath					/
//	@query.collection.format	multi
```
Swag also lets you define your General API annotations in another file. You can learn how to do that here.

#### Step 5: Add API operation annotations to controller code

API operation annotations contain how the controller works (description, router, request type, parameters, and response codes). Let’s see how to add annotations for the getAllTodos route.

Add the following annotations right before the getAllTodos function in the main.go file:

```go
//	@Summary	get all items in the todo list
//	@ID			get-all-todos
//	@Produce	json
//	@Success	200	{object}	todo
//	@Router		/todo [get]
```

In the code above, we defined the following:

- @Summary, the summary of what the route does
- @ID, a unique identifier for the route (mandatory for every route)
- @Produce, the route response data type
- @Success 200, the response model for expected status codes
- @Router /todo [get], the route URI and accepted request method

We will add annotations for the `getTodoByID` route. Add the following code right before the `getTodoByID` function in the `main.go` file:

```go
// @Summary get a todo item by ID
// @ID get-todo-by-id
// @Produce json
// @Param id path string true "todo ID"
// @Success 200 {object} todo
// @Failure 404 {object} message
// @Router /todo/{id} [get]
```

annotated form of the `getTodoByID` route


Here, we specified to Swag that the route accepts a mandatory string parameter called id attached to the request path. It has the name todo ID with @Param id path string true "todo ID".

Next, we will add annotations for the createTodo route. Add the following code right before the createTodo function in the `main.go` file:
```go
// @Summary add a new item to the todo list
// @ID create-todo
// @Produce json
// @Param data body todo true "todo data"
// @Success 200 {object} todo
// @Failure 400 {object} message
// @Router /todo [post]
```

Here, we specified to Swag that the route accepts a mandatory todo parameter called data attached to the request body. It has the name todo data with @Param data body todo true "todo data".

We will add annotations for the deleteTodo route. Add the following code right before the deleteTodo function in the `main.go` file:
```go
// @Summary delete a todo item by ID
// @ID delete-todo-by-id
// @Produce json
// @Param id path string true "todo ID"
// @Success 200 {object} todo
// @Failure 404 {object} message
// @Router /todo/{id} [delete]
```
annotated form of the `deleteTodo` route

#### View and test the documentation

Now you have defined all the annotations for the server and routes, let’s view and test the documentation.

To generate the documentation from your code, run `swag init` again in the terminal.

We have to run `swag init` each time we update the annotations in the code, so the documentation is regenerated and updated accordingly. And also you can run `swag fmt` if you want to format your swagger comments.

We also have to register a route handler to the Fiber router responsible for rendering the Swagger documentation created by Swag. Add the following code to the router configuration in main.go:

```go
app.Get("/swagger/*", swagger.HandlerDefault)
```

Now that we have configured the docs route, run the server and navigate to the `/swagger` URI in your browser and you’ll see the generated Swagger documentation.
```bash
go run main.go
```

### Conclusion

This article showed you how to seamlessly generate Swagger documentation for web APIs built with Go using Swag. You can learn more about Swag from its [official documentation](https://github.com/swaggo/swag).

We chose to use Swagger because of the numerous features and functionalities that make it easy to create and maintain documentation for web APIs.

The source code of the web API built and documented in this tutorial is available on GitHub for you to explore. [Repository](https://github.com/jackgris/go-api-with-swarg)

This post is highly inspired by [this](https://blog.logrocket.com/documenting-go-web-apis-with-swag/) I hope you also enjoy it.
