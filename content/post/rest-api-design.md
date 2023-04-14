---
title: "Rest Api Design"
date: 2023-04-13T19:34:36-03:00
draft: false
tags: ["api", "swagger", "documentation", "openapi"]
---

Maybe I should have written this article before the previous one. But sometimes one doesn't know where to start, so now I will explain what you need to take care of when you design a Rest API.

### Why?

Designing a REST API follows specific design principles that allow for flexibility, scalability, and independence between client and server applications. The design should follow the six REST design principles to ensure uniformity, decoupling, statelessness, cacheability, a layered system, and optional code on demand. In a few words will help you to decouple the client and service implementations. And will make it easy to scale and split the work.

Another advantage of REST over HTTP is that it uses open standards, and does not bind the implementation of the API or the client applications to any specific implementation. For example, a REST web service could be written in `Go`, and client applications can use any language or toolset that can generate HTTP requests and parse HTTP responses.

#### What methods to use?

REST is independent of any underlying protocol and is not necessarily tied to HTTP. However, most common REST API implementations use HTTP as the application protocol, and this guide focuses on designing REST APIs for HTTP.

The set of standard `HTTP` methods that should be used for different types of requests and the most commonly used methods are `GET, POST, PUT/PATCH,` and `DELETE`.

For more details see [here](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#define-api-operations-in-terms-of-http-methods)

#### And codes?

These are the most commonly used:

- 200 OK: The request was successful
- 201 Created: The request was successful and a new resource was created
- 400 Bad Request: The request was malformed or invalid
- 401 Unauthorized: The request requires authentication or authorization
- 404 Not Found: The requested resource was not found
- 500 Internal Server Error: An error occurred on the server

But for more specific codes, is necessary that you see the link at the bottom about: "Rest APIs Status Code"

For more details see [here](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#conform-to-http-semantics) and [this](https://restfulapi.net/http-status-codes/)


### Six REST design principles?

The design of REST APIs should follow the six REST design principles, also known as architectural constraints:

1 - Uniform interface: All API requests for the same resource should look the same, no matter where the request comes from. The REST API should ensure that the same piece of data belongs to only one uniform resource identifier (URI). Resources shouldn’t be too large but should contain every piece of information that the client might need.

Example:
```http
https://bookshop.com/book/1
```

2 - Client-server decoupling: In REST API design, client and server applications must be completely independent of each other. The only information the client application should know is the URI of the requested resource; it can't interact with the server application in any other ways. Similarly, a server application shouldn't modify the client application other than passing it to the requested data via HTTP.

3 - Stateless: REST APIs are stateless, meaning that each request needs to include all the information necessary for processing it. In other words, REST APIs do not require any server-side sessions. Server applications aren’t allowed to store any data related to a client request.

Example: (Many web APIs use JSON as the exchange format. For example, a GET request to the URI listed above might return this response body)
```json
{"bookId":1,"value":99.90,"quantity":1}
```

4 - Cacheability: REST APIs should be designed to be cacheable to improve performance. Responses from a REST API should explicitly state whether they can be cached or not.

5 - Layered system: REST APIs should be designed as a layered system, with each layer having a specific responsibility. This allows for better scalability and flexibility.

6 - Code on demand (optional): REST APIs should be designed to allow for the downloading and execution of code on the client side. This constraint is optional, and not all REST APIs need to support it.

### Maturity Model

In 2008, Leonard Richardson proposed the following maturity model for web APIs:

- Level 0: Define one URI, and all operations are POST requests to this URI.
- Level 1: Create separate URIs for individual resources.
- Level 2: Use HTTP methods to define operations on resources.
- Level 3: Use hypermedia (HATEOAS, described [here](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#use-hateoas-to-enable-navigation-to-related-resources)).

Level 3 corresponds to a truly RESTful API according to Fielding's definition. In practice, many published web APIs fall somewhere around level 2.

### Versioning

Versioning a REST API is a critical aspect of API design that helps to manage the changes to the API transparently. When changes are made to an API, it can impact existing client integrations, so versioning helps to manage this complexity. REST APIs are uniform and stateless, and they use standard HTTP verbs to perform operations on resources. There are different ways to version a REST API, and each approach has its advantages and disadvantages.

For more information see [here](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#versioning-a-restful-web-api)

### Open API

The OpenAPI Specification (OAS) is a vendor neutral description format for HTTP-based remote APIs. It was originally based on the Swagger 2.0 Specification, donated by SmartBear Software in 2015. This openness has encouraged the creation of a vast amount of tools which perfectly showcase the power of open, machine-readable API descriptions (called documents in OpenAPI).
It’s probably because of the amount of tools available when working with OpenAPI that it has become the most broadly adopted industry standard for describing modern APIs.
OpenAPI can describe APIs based on the HTTP protocol (like RESTful ones) but also APIs based on HTTP-like protocols like CoAP (Constrained Application Protocol) or WebSockets. This allows OpenAPI to be used in resource-restricted scenarios like IoT (Internet of Things), for example.

For more information see [here](https://oai.github.io/Documentation/specification)

### Final Thoughts

In conclusion, RESTful API is an essential tool for creating modern, scalable, and interoperable web applications. It provides a standard and consistent approach to building RESTful web services, making it easier for developers to collaborate and integrate different systems. It also enables the development of loosely coupled services, which can be more easily maintained and updated over time. However, it is important to follow best practices when designing and implementing RESTful APIs to ensure they are scalable, secure, and maintainable.

&emsp;
 
#### I highly recommend reading these articles.

[OpenAPI Specification](https://oai.github.io/Documentation/specification)

[Rest APIs Status Code](https://restfulapi.net/http-status-codes)

[APIs Design](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)
