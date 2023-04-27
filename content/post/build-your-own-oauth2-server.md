---
title: "Build your own OAuth2 server"
date: 2023-04-26T18:09:36-03:00
draft: true
---

# What is OAuth2?

Open Authorization Version 2.0 is known as OAuth2. It’s one kind of protocol or framework to secure RESTful Web Services. OAuth2 is very powerful. Now a days, majority of the REST API are protected with OAuth2 due to it’s rock solid security.

```
 +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+
```

There are four types of OAuth2 server based of the Grant Flow type:

01. Authorization Code Grant

02. Implicit Grant

03. Client Credentials Grant

04. Password Grant
