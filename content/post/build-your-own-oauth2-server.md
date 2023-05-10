---
title: "Build your own OAuth2 server"
date: 2023-05-09T18:09:36-03:00
draft: false
tags: ["api", "oauth", "oauth2", "golang", "authentication"]
---

# What is OAuth2?

OAuth 2.0, which stands for “Open Authorization”, is a standard designed to allow a website or application to access resources hosted by other web apps on behalf of a user. It replaced OAuth 1.0 in 2012 and is now the de facto industry standard for online authorization. OAuth 2.0 provides consented access and restricts actions of what the client app can perform on resources on behalf of the user, without ever sharing the user's credentials.

Although the web is the main platform for OAuth 2, the specification also describes how to handle this kind of delegated access to other client types (browser-based applications, server-side web applications, native/mobile apps, connected devices, etc.)

![oauth2 logo](https://jackgris.github.io/goscrapy-blog/img/Oauth_logo.svg.png)


More info on [OAuth web page](https://oauth.net/2/)

## Why I should use it?

A few reasons why you might want to use OAuth2:

- Improved Security: OAuth2 allows for secure, token-based authentication, which helps reduce the risk of user credentials being compromised.

- User-Friendly: By allowing users to grant or revoke access to their resources on a per-application basis, OAuth2 helps ensure that users maintain control over their data.

- Third-Party Integration: OAuth2 enables third-party services to access user resources on their behalf, making it easier to integrate with other applications and services.

- Scalability: OAuth2 is designed to be scalable, making it ideal for large-scale applications that need to handle a high volume of user requests.

Overall, OAuth2 provides a flexible, secure, and user-friendly way to manage access to web-based resources, making it a popular choice for developers and organizations alike.

## How many ways or modalities we have to use it?

Originally we have for modalities:

- Authorization Code Grant: A code is issued and used to obtain the access_token. This code is released to a front-end application (on the browser) after the user logs in. The access_token instead, is issued Server side, authenticating the client with its password and the obtained code.
- Implicit Grant: after the user logs in, the `access_token` is issued immediately.
- Client Credential Grant: the `access_token` is issued on the server, authenticating only the client, not the user.
- Password Grant: the `access_token` is issued immediately with a single request containing all login information: username, user password, client id, and client secret. It could look easier to implement, but it has some complications.

![oauth2 cheetsheet](https://jackgris.github.io/goscrapy-blog/img/oauth2-cheatsheet.png)

You can read about it on the [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749) but now that changed, you can see the recommended options here [Grant Types](https://oauth.net/2/grant-types/) For example a new way is to use [PKCE](https://oauth.net/2/pkce/) which is an extension to the Authorization Code flow to prevent CSRF and authorization code injection attacks.

## OAuth2 Client

![oauth2 client](https://jackgris.github.io/goscrapy-blog/img/oauth2-client.png)

If you’re familiar with this screen, you know what I’m talking about. Anyway, Let me explain how this work:

You’re building a user facing application that works with user’s `github` repositories. For example: CI tools like TravisCI, CircleCI, Drone etc.

But user’s github account is secured and no one can access it if the owner doesn’t want. So how do these CI tools access user’s `github` account and repositories?

Well, your application will ask the user

`“In order to work with us, you need to give read access to your github repos. Do you agree?”`

Then the user will say

`“Yes I do. And do whatever needs to do”.`

Then your application will contact github’s authority to grant access to that particular user’s github account. Github will check if it’s true and ask that user to authorize it. Then github will issue an ephemeral token to the client.

Now, when your application needs to access it after the authentication and authorization, it needs to send the access token with the request so that github will think:

`“Oh, the access token looks familiar, maybe we have given that to you. Ok, you can access”`

That’s the long story. Now you don’t need to go to github authority physically each time (we never had to do that). Everything can be done automatically.

![oauth2 flow](https://jackgris.github.io/goscrapy-blog/img/oauth2-flow.png)


This is a UML sequence diagram is just a graphical representation.

From the above picture, we found some important things.

### OAuth2 has 4 roles:

01. User: The end user who will use your application

02. Client: The application you’re building that will use github account and the user will use it

03. Auth Server: The server that deals with the main OAuth things

04. Resource Server: The server that has the protected resources. For example github

Client sends OAuth2 request to the auth server on behalf of the user.

## OAuth2 Server

Why build an OAuth2 server? Imagine, you’re building a very useful application that gives accurate weather information (there are plenty of this kind of api out there). Now you want to make it open so that public can use it or you want to make money with it.

Whatever the case is, you need to protect your resources from un-authorized access or malicious attacks. In order to do that you need to secure your API resources. Here comes the OAuth2!

From the picture above, we can see that we need to place an Auth Server in front of our REST API Resource Server. That’s what we’re talking about. The Auth Server will be built using OAuth2 specification.

The primary goal of the OAuth2 server is to provide an access token to the client. That’s why OAuth2 Server is also known as OAuth2 Provider, because they provide token.


### Client Credentials Grant Flow Based Server

When implementing Client Credentials Grant Flow based OAuth2 Server, we need to know a couple of things.

In this grant type, there’s no user interaction (i.e. signup, sign-in). Three things are needed, and they are the `client_id`, the `client_secret` and `grant_type` in this case `client_credentials`. With these two things, we can obtain `access_token`. Client is the 3rd party application. When you need to access a resource server without the user or only by the client application, then this grant type is simple and best suited.

Here’s a UML Sequence Diagram of it.

![oauth2 client credentials grant flow](https://jackgris.github.io/goscrapy-blog/img/oauth2-clien-credentials-grant-flow.png)

## Coding

Now we will start with the funny thing.

### Routes

These are the necessary endpoints to manage the credentials grant flow:

01. /credentials for issuing client credentials (client_id and client_secret)

02. /token to issue token with client credentials

03. /protected this will be only available if you pass the token

We need to implement these three routes. The first two are related to the OAuth2 flow, and the third show you how it works. And can be any route you want to use with validation.

Here’s the preliminary setup

Run the code and go to http://localhost:3000/credentials route to register and get `client_id` and `client_secret`. And this is the handler:

```go
// Credentials will return the client ID and the client secret necessary to get the token
func Credentials(clientStore *store.ClientStore) fiber.Handler {
	handler := func(c *fiber.Ctx) error {
		clientId := uuid.New().String()[:8]
		clientSecret := uuid.New().String()[:8]
		err := clientStore.Set(clientId, &models.Client{
			ID:     clientId,
			Secret: clientSecret,
			Domain: "http://localhost:9094",
		})
		if err != nil {
			return err
		}
		r := map[string]string{"CLIENT_ID": clientId, "CLIENT_SECRET": clientSecret}
		c.Status(http.StatusOK)
		return c.JSON(r)
	}

	return handler
}
```

Now go to this URL: `http://localhost:3000/token?grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&scope=all` (remember to change the value of CLIENT_ID and CLIENT_SECRET to the values received from the credentials endpoint)
And this is the handler:

```go
// Token handler will return the token related to a client ID
func Token(srv *server.Server) fiber.Handler {
	// first, create a Handlerfunc because go-oauth2/oauth2 library was
	// created to be used with the standard net/http library
	var h http.HandlerFunc = func(w http.ResponseWriter, r *http.Request) {
		_ = srv.HandleTokenRequest(w, r)
	}

	// wraps net/http HandlerFunc to fiber handler
	return func(c *fiber.Ctx) error {
		c.Request()
		handler := fasthttpadaptor.NewFastHTTPHandler(h)
		handler(c.Context())
		return nil
	}
}
```

You will get the `access_token` with the expiry time and some other information.

Now we got our `access_token`. But our `/protected` route is still un-protected. We need to setup a way that will check if a valid token exists with each client request. If yes, then we give the client access. Otherwise not. We can do this with a middleware.

Writing middleware in Go is much more fun if you know what you are doing. Here’s the middleware:

```go
// ValidateToken will be the middleware used to protect the URL that you want to be private
func ValidateToken(hand fiber.Handler, srv *server.Server) fiber.Handler {

	handler := func(c *fiber.Ctx) error {

		ctx := context.TODO()
		method := c.Method()
		url := c.OriginalURL()
		body := c.Body()
		r, err := http.NewRequestWithContext(ctx, method, url, bytes.NewReader(body))
		if err != nil {
			return NotAllowed(c)
		}
		_, err = srv.ValidationBearerToken(r)
		if err != nil {
			return NotAllowed(c)
		}
		return hand(c)
	}

	return handler
}
```

Now run the server and try to access `/protected` endpoint without the `access_token` as URL Query. Then try to give the `wrong access_token`. Either way, the auth server will stop you.

Now get the credentials and `access_token` again from the server and send a request to the protected endpoint :

`http://localhost:3000/test?access_token=YOUR_ACCESS_TOKEN`

Bingo! You’ll get access to it. So we’ve learned how to setup our own `OAuth2` Server using `Go`.


For writing a client for example for Google u another option you can see this package that contains a client implementation for OAuth 2.0 spec for many providers. [Go OAuth2 Providers](https://github.com/golang/oauth2)


#### You can see the complete code in this repository

[Go OAuth2 example](https://github.com/jackgris/go-oauth2-example) keep in mind that this library is based on the standard Go net/http library. And I modified some things to use it with Fiber, just for fun.

## Final Conclusion


After understanding how it work is sad to say that is better to use a service that implements it. At least you are in a big team, where you have the right resources and developers to create a secure service. There are several reasons why you might want to use a service solution for `OAuth2` instead of implementing the protocol yourself:

- Simplified Implementation: Implementing `OAuth2` can be a complex and time-consuming process, especially if you're not familiar with the protocol. A service solution can simplify the implementation process by providing pre-built libraries, SDKs, and documentation.

- Security: `OAuth2` implementations can be vulnerable to security issues if not implemented correctly. A service solution can provide additional security measures such as encryption, rate limiting, and auditing to help prevent unauthorized access.

- Scalability: As your application grows, managing `OAuth2` requests can become increasingly complex. A service solution can provide a scalable infrastructure to handle a high volume of requests and ensure high availability.

- Time and Cost Savings: Building and maintaining an `OAuth2` implementation can be expensive and time-consuming. A service solution can save you time and money by providing a ready-to-use solution that can be easily integrated into your application.

- Compliance: Service solutions often provide built-in compliance with regulations such as `GDPR` and `HIPAA`, helping you ensure that your `OAuth2` implementation meets regulatory requirements.

Overall, using a service solution for `OAuth2` can provide significant benefits in terms of security, scalability, compliance, and cost-effectiveness. It allows you to focus on your core business logic instead of worrying about the complexities of implementing and maintaining an OAuth2 infrastructure.

I hope you find this post useful.

#### This article was based on some articles, official data, and also other blogs, here are the links:

[An OAuth 2 Introduction for beginners](https://itnext.io/an-oauth-2-0-introduction-for-beginners-6e386b19f7a9)

[Build your own OAuth2 server in Go](https://hackernoon.com/build-your-own-oauth2-server-in-go-7d0f660732c3?utm_source=dormosheio&utm_campaign=dormosheio)

[Go OAuth2 Tutorial](https://tutorialedge.net/golang/go-oauth2-tutorial/)

##### Why not use Implicit Grant and Password Grant:

[Whats wrong with implicit grant](https://fusionauth.io/blog/2021/04/29/whats-wrong-with-implicit-grant)

[Grant Types Implicit](https://oauth.net/2/grant-types/implicit/)

[Grant Types Password](https://oauth.net/2/grant-types/password/)

##### Server library:

[go-oauth2](https://github.com/go-oauth2/oauth2)
