---
title: "Building an Authorization Service With Fiber Using JWT"
date: 2023-05-10T23:05:55-03:00
draft: true
tags: ["api", "oauth", "oauth2", "golang", "authentication", "jwt"]
---

![jwt logo](https://jackgris.github.io/goscrapy-blog/img/jwt-logo.svg)


# What is JWT?

JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA or ECDSA.

Although JWTs can be encrypted to provide secrecy between parties, we will focus on signed tokens. Signed tokens can verify the integrity of the claims contained, while encrypted tokens hide those claims from other parties. When tokens are signed using public/private key pairs, the signature also certifies that only the party holding the private key is the one that signed it.

## Why you should use it?

Here are some scenarios where JSON Web Tokens are useful:

- Authorization: This is the most common scenario for using JWT. Once the user is logged in, each subsequent request will include the JWT, allowing the user to access routes, services, and resources that are permitted with that token. Single Sign On is a feature that widely uses JWT nowadays, because of its small overhead and its ability to be easily used across different domains.

- Information Exchange: JSON Web Tokens are a good way of securely transmitting information between parties. Because JWTs can be signed—for example, using public/private key pairs—you can be sure the senders are who they say they are. Additionally, as the signature is calculated using the header and the payload, you can also verify that the content hasn't been tampered with.

- Statelessness: JWTs are stateless, meaning the server does not need to keep track of user sessions or store session data on the server-side. This allows for easier scalability and better performance in distributed systems.

- Extensibility: JWTs allow for the inclusion of custom claims, enabling developers to add additional information or context relevant to their application. This flexibility makes JWTs adaptable to a wide range of use cases.

- Verbose: JWT is less verbose than XML, when it is encoded its size is also smaller, making JWT more compact than Security Assertion Markup Language Tokens. This makes JWT a good choice to be passed in HTML and HTTP environments.

If you want to learn more specific things about JWT, I recommend you read the official [Introduction](https://jwt.io/introduction)


## What are the components of a JWT and how are they structured?

A JSON Web Token (JWT) consists of three main components: the header, the payload, and the signature. These components are combined together to form the structure of a JWT.

- Header: The header component of a JWT contains metadata about the token and how it should be processed. It is a JSON object and typically consists of two properties:
        "alg" (algorithm): Specifies the algorithm used for signing the token, such as HMAC, RSA, or others.
        "typ" (type): Indicates the type of the token, which is typically set to "JWT". The header is Base64Url encoded to form the first part of the JWT.

For example:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- Payload: The payload component of a JWT contains the claims, which are statements about the entity (user, application, etc.) and additional metadata. It is also a JSON object and consists of three types of claims:
        Registered claims: These are predefined claims standardized by the JWT specification. Examples include "iss" (issuer), "sub" (subject), "exp" (expiration time), and "iat" (issued at time).
        Public claims: These are custom claims defined by the application developers to include additional information. They should be defined in a way that avoids collisions with other claims.
        Private claims: These are custom claims that are meant to be used in private agreements between parties. The payload is also Base64Url encoded to form the second part of the JWT.

An example payload could be:
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

- Signature: The signature component of a JWT is used to verify the integrity of the token and ensure that it hasn't been tampered with. It is created by combining the encoded header, encoded payload, and a secret key (or a public/private key pair in case of asymmetric algorithms) using the algorithm specified in the header. The signature is appended as the third part of the JWT, separated by a dot.

The three components (header, payload, and signature) are concatenated together with dots to form the complete JWT: `header.payload.signature`

It is important to note that JWTs are typically compact, URL-safe strings that can be easily transmitted over different communication channels, such as HTTP headers or URL parameters. The structure and encoding of JWTs make them self-contained and efficient for carrying information securely.

Remember, in this web [JWT.io](https://jwt.io/) you have a debugger where you can test your JSON Web Token.

## How do JSON Web Tokens work?

1- When a user authenticates with a server (usually through a login process), the server verifies the user's credentials (e.g., username and password). Upon successful authentication, the server generates a JWT.

2- The server sends the JWT back to the client as a response to successful authentication. The client can store the token (e.g., in local storage or a cookie) for subsequent requests.

3- The client includes the JWT in the headers (e.g., Authorization header) of subsequent requests to access protected resources or perform authenticated actions. This allows the server to identify and authorize the user based on the information contained within the JWT.

4- When the server receives a request with a JWT, it first verifies the signature by recalculating it using the same algorithm and the secret or public key. If the recalculated signature matches the received signature, the token is considered valid. The server then decodes the Base64Url encoded header and payload to extract the claims and verify any necessary criteria (e.g., expiration time).

5- Once the token is validated, the server can access the claims within the payload to determine the user's identity, role, and permissions. Based on this information, the server can authorize or deny access to the requested resource or perform other actions accordingly.


## How does JWT differ from other authentication mechanisms like session cookies?

- JWT is a stateless mechanism, while session cookies are stateful. This means that JWT does not require the server to keep track of user sessions, whereas session cookies do.
- JWT is more flexible and easier to use in an API because it is self-contained and does not require database lookups.
- JWT can be stored in various ways in browsers or front-end applications, including local storage and session storage. In contrast, session cookies are typically stored in the browser's cookie jar.
- JWT can be used across different platforms, including mobile and IoT applications, while session cookies are limited to web applications.
- JWT tokens cannot be revoked, even if they are compromised, until they expire. This can lead to security vulnerabilities. In contrast, session cookies can be invalidated on the server-side.
- JWT tokens are larger in size compared to session cookies, which can be a concern for performance.

Overall, JWT is a more modern approach to authentication and is better suited for stateless APIs and distributed systems. However, session cookies may still be a viable option for web applications that require stateful authentication and session management.

## What are the security considerations when using JWTs?

There are several security considerations to keep in mind:

- JWTs should be signed with a strong secret key to prevent tampering by unauthorized parties.
- JWTs should have an expiration time (exp) to limit the lifetime of the token and prevent replay attacks. auth0.com
- JWTs should not contain sensitive information such as passwords or credit card numbers. Instead, they should be used to transmit non-sensitive data such as user IDs or permissions, which can be used to look up additional information on the server-side. serengetitech.com
- JWTs should be stored securely on the client-side, for example in an HttpOnly cookie. serengetitech.com
- JWTs can be vulnerable to certain attacks such as timing attacks and signature attacks. To mitigate these risks, it is important to use a secure signing algorithm and validate the signature and expiration time on the server-side. serengetitech.com
- If a JWT is compromised, it cannot be revoked until it expires, which can be a security concern. To mitigate this risk, it is important to limit the lifetime of the token and to store sensitive information on the server-side. jwt.io

## Which programming languages and frameworks support JWT?

In general, most modern web frameworks support JWT in some form, and there are also many third-party libraries available for different programming languages.
    `Go, Python, Java, Javascript, C#, Ruby, PHP`
These are just a few examples, and JWT support is available in many other languages and frameworks as well. When working with JWTs, it's recommended to choose a library or framework that is actively maintained, has good community support, and follows best practices for security and token validation.

## What are the best practices for using JWTs in web applications?

- Don't use JWTs as session tokens. While it may seem like a good idea to use JWTs for session validation, they are not well-suited for this purpose. JWTs are not easily invalidated, and using them for sessions can create security vulnerabilities. Instead, use JWTs for authentication and pass the user information to the server to establish a session.
- Always sign JWTs with a strong secret key to prevent tampering.
- Set an expiration time (exp) for JWTs to prevent replay attacks.
- Don't store sensitive information such as passwords or credit card numbers in JWTs. Instead, store non-sensitive data such as user IDs or permissions.
- Store JWTs securely on the client-side, for example in an HttpOnly cookie.
- Validate the signature and expiration time of JWTs on the server-side to prevent tampering and replay attacks.
- Use a secure signing algorithm such as HMAC with SHA-256 or RSA with SHA-256 to sign JWTs.
- Avoid sending JWTs over unencrypted channels such as HTTP. Instead, use HTTPS to encrypt the communication between the client and server.

These best practices can help ensure that JWTs are used securely and effectively in web applications.

## What are the common pitfalls and challenges when implementing JWT-based authentication?

I recommend you read these posts, they contain important advice.

[Websecuritylens.org](https://www.websecuritylens.org/common-jwt-implementation-mistakes-and-how-to-exploit-them/)

[7 ways to avoid jwt pitfalls](https://42crunch.com/7-ways-to-avoid-jwt-pitfalls/)

[angular-university.io](https://blog.angular-university.io/angular-jwt/)

## How can JWTs be revoked or invalidated if needed?

JWTs can be revoked or invalidated using a token blacklist method or by utilizing a database. Here are some ways to implement these methods:

- Token Blacklist Method: One of the main properties of JWT is that it's stateless and is stored on the client and not in the Database. You don't have to query the database to validate the token. For as long as the signature is correct and the token hasn't expired, it would allow the user to access the restricted resource. This is most efficient when you wish to reduce the load on the database. The downside, however, is that it makes invalidating the existing, non-expired token difficult. [4]
    To invalidate a token, create a blacklist. The logic behind it is straightforward and easy to understand and implement. A JWT can still be valid even after it has been deleted from the client, depending on the expiration date of the token. So, invalidating it makes sure it's not being used again for authentication purposes. [4]
    You can store all compromised JWTs in the database and when an authorization request is received, you check that the JWT used is not part of this list. [1]

- Database: If you need to keep track of information about revoked JWTs, it is recommended to utilize a database. This allows you to easily store and utilize metadata for revoked tokens, such as when it was revoked, who revoked it, whether can it be un-revoked, etc. [0]
    In this method, a user ID and the application for which the revocation occurred are included when a jwt.refresh-token.revoke event is emitted. It also includes the JWT lifespan for this application. Any JWT for this user that was issued before the time when this event was received is thus revoked. JWTManager maintains a list of user IDs whose JWTs are revoked. You could also use JWT IDs (the jti claim), which, in some cases, might be simpler. JWTManager also launches a thread to clean up the cache to remove expired users and avoid running out of memory. For each API call, in addition to all the other checks that should be performed against a JWT (verifying the signature, expiration, issuer and so on), the backend needs to check validity with JWTManager. [2]

- Redis: If your only requirements are to check if a JWT has been revoked, it is recommended to use Redis. It is blazing fast, can be configured to persist data to disk, and can automatically clear out JWTs after they expire by utilizing the Time To Live (TTL) functionality when storing a JWT. [0]

You should be aware of the limitations of JWTs before implementing their usage. JWTs should only be used if the web application is completely stateless. Otherwise, classic systems with sessions should be considered. [3]

## Let's code

First, set up your Go workspace and initialize the Go modules file `go.mod`, run this command on your terminal in the workspace directory to install the necessary packages:
```bash
go get github.com/gofiber/fiber/v2
go get github.com/gofiber/jwt/v3
go get github.com/golang-jwt/jwt/v4
```

Let’s start with creating a simple web server with an endpoint that will be secured with a JWT.

```go
func main() {
	app := fiber.New()

	// Login route
	app.Post("/login", login)

	// Unauthenticated route
	app.Get("/", accessible)

	// JWT Middleware
	app.Use(jwtware.New(jwtware.Config{
		SigningKey: []byte("secret"),
	}))

	// Restricted Routes
	app.Get("/restricted", restricted)

	fmt.Println(app.Listen(":3000"))
}
```

You can see here, we have one endpoint accessible to anybody, another to manage the user login, and the third one that is restricted only for users that have the right jwt token (in this case only for a particular user).

This is the handler accessible to anybody:

```go
func accessible(c *fiber.Ctx) error {
	return c.SendString("Accessible")
}
```

In this handler we will manage the login:

```go
func login(c *fiber.Ctx) error {
	user := c.FormValue("user")
	pass := c.FormValue("pass")

	fmt.Printf("/login we received user %s and pass %s\n", user, pass)

	// Off course, here you should use a database or some service where you have the user data saved.
	if user != "john" || pass != "doe" {
		// Throws Unauthorized error
		return c.SendStatus(fiber.StatusUnauthorized)
	}

	// Create the Claims
	claims := jwt.MapClaims{
		"name":  "John Doe",
		"admin": true,
		// Here you config the time that the token will be ok before being invalidated.
		"exp": time.Now().Add(time.Hour * 72).Unix(),
	}

	// Create token
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

	// Generate encoded token and send it as response.
	t, err := token.SignedString([]byte("secret"))
	if err != nil {
		return c.SendStatus(fiber.StatusInternalServerError)
	}

	return c.JSON(fiber.Map{"token": t})
}
```

And finally the restricted endpoint:

```go
func restricted(c *fiber.Ctx) error {
	user := c.Locals("user").(*jwt.Token)
	claims := user.Claims.(jwt.MapClaims)
	name := claims["name"].(string)

	fmt.Printf("/restricted receive user token: %s\n", user.Raw)
	fmt.Printf("And we know that his name is: %s\n", name)

	return c.SendString("Welcome " + name + "\n")
}
```
[Go JWT example](https://github.com/jackgris/go-jwt-example) This repository contains all the code. And you can see an explanation about how to run it and test the endpoint.

## Conclusion

In conclusion, JSON Web Tokens (JWT) is a widely adopted method for securely transmitting information between parties in a compact and self-contained manner. They provide a means of authentication and authorization, allowing users to access protected resources without relying on traditional session-based mechanisms.

JWTs offer several advantages, including statelessness, scalability, and ease of integration. Since the token contains all necessary information, server-side storage of session data is not required, resulting in a more scalable architecture. Furthermore, JWTs can be easily used across different domains and platforms, making them suitable for microservices and distributed systems.

However, it is essential to consider some considerations and best practices when using JWTs. These include properly validating and verifying tokens to prevent unauthorized access, securely storing and transmitting tokens to mitigate risks of token theft or tampering, and carefully selecting the token's expiration time to balance security and usability.

Overall, JWTs provide a flexible and efficient solution for authentication and authorization in modern web applications, but their successful and secure implementation relies on following best practices and understanding their limitations.


### Sources

[JWT.io](https://jwt.io/)
[Auth0](https://auth0.com/)
[Okta: cookies vs tokens](https://developer.okta.com/blog/2022/02/08/cookies-vs-tokens)

[Jerrynsh](https://jerrynsh.com/all-to-know-about-auth-and-cookies/)
[Bitsrc.io](https://blog.bitsrc.io/best-practices-for-using-jwt-df3788433fd3)
[Curity.io](https://curity.io/resources/learn/jwt-best-practices/)

[Loginradius.com](https://www.loginradius.com/blog/engineering/guest-post/jwt-authentication-best-practices-and-when-to-use/)
[Websecuritylens.org](https://www.websecuritylens.org/common-jwt-implementation-mistakes-and-how-to-exploit-them/)
[7 ways to avoid jwt pitfalls](https://42crunch.com/7-ways-to-avoid-jwt-pitfalls/)

[angular-university.io](https://blog.angular-university.io/angular-jwt/)
[Revoking access with a JWT Blacklist](https://supertokens.com/blog/revoking-access-with-a-jwt-blacklist)
[Invalidating JWT](https://www.loginradius.com/blog/engineering/invalidating-jwt/)

[JWT authentication in Golang](https://codewithmukesh.com/blog/jwt-authentication-in-golang/#JWT_Explained)
[JWT authentication Go](https://blog.logrocket.com/jwt-authentication-go/)
[Golang Mongodb JWT Authentication Authorization](https://codevoweb.com/golang-mongodb-jwt-authentication-authorization/)
[JWT based authentication for Go Rest ackend](https://enlear.academy/jwt-based-authentication-for-go-rest-backend-9973a30f7ccb)
