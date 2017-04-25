---
layout: post
---

# (1 Week)

# Introduction

This week's lecture notes discuss general authentication concepts, as well
as the concept of using external, HTTP-based APIs to improve your applications.
These two concepts are tied together because understanding API-based
authentication and how you should implement it plays into how many APIs
authenticate requests to their services.  In general, a reasonable understanding
of how web-based authentication works will keep you conscious of information
security concepts and concerns that any backend developer should be knowledgable of.

# What is Authentication?

As we discussed during the first week of this course, the web is stateless.
Every request made to a server's API requires complete information in order to
perform work on the server.  This means that, in addition to telling the server
to do something (make a todo-list, search based on a query string, etc.), you
need some way of letting the server know who you are every time you make a
request, and the server needs some way of validating that you are who you say
you are on every request.  These needs of both the client and the server
encapsulate the idea of **authentication**.  

# Let's Talk About Sign In

In this day and age, most services have some sort of way of getting users to sign up,
either through a "local" sign up experience or one through a particular service like
Facebook or Google.  For now, we'll focus on a "local" sign in experience, a.k.a.
entering your email and confirming a password to lock-down your account.  By signing up,
you are asking that service to store your credentials in some sort of User row / document / entity
in their database, as well as depending on them to secure your information via some
sort of hashing and / or encryption (more on this later).  Now that you've signed up,
in order to access your new account, you need to sign in.  Signing in means you
must send your email and password over the network to the API.  Then we're signed in,
but what happened when we signed in?  Magic?

What actually happens depends on how the developer of the server's implementation.

# Basic is Basic

Back in the old days of the web, logging in merely meant sending your username and
password to the server, getting a 200 status-code back and storing the username
and password as a [`cookie`](http://www.pctools.com/security-news/what-are-browser-cookies/)
on the browser to resend to the server on every request.  This means of
authenticating is called `HTTP Basic Authentication`.  It is dubbed so because it
is the most basic way of identifying who you are on every request to the server.  You
send your complete credentials (in this case, email and password) on every request.

# Sessions are Cooler

Instead of `HTTP Basic Authentication`, most services opt for `HTTP Session Authentication`
after a user has signed in, which is a lot more secure while operating
on the present day `www`, where several web-based security exploits exist.  We will
be discussing `session`-based authentication and an implementation strategy for securing
your APIs once a user has signed in.  

# What the Heck is a Session?

Before delving into implementation details, let's talk at a high level about what a session
is and how the `HTTP Session Authentication` flow works.  To motivate this approach vs.
`HTTP Basic Authentication`, we will discuss some of the advantages from a security perspective
to using `HTTP Session Authentication`.  

A `session` identifies a user's recent "session" interacting with the server.  Imagine that,
by signing in, I unlock a door to enter a room to interact with the server, but I can only
stay in the room for so long.  My stay in the room is my "session" with the server.  

The `HTTP Session Authentication` flow is as follows:

1. The client (iOS / Android app, browser) authenticates (signs in) with its credentials (e.g. username
and password)
2. The server validates the sign-in attempt and generates a `session_token` for the user that
signed in.  This token has an expiration date (e.g. 30 minutes into the future, 2 weeks into the future,
whatever the implementer of the authentication flow prefers).  This `session_token` and all metadata
are stored and properly associated with the user row / document / entity in the server's database.
3. The server sends this `session_token` back to the client as a result of the successful sign-in.
4. The client stores this `session_token` as a cookie if it's a browser or on disk in a
secure fashion if it's an mobile application of some sort.
5. The client packages this `session_token` with every request to the server so the server
can associate the request with the user the `session_token` belongs to.

# Why are Sessions So Great?

What are some of the benefits of using a `session` vs. a `username-password` pair on every request?

Generally, when you are writing an API that will be running in production,
communication with that API should be secured via [`SSL`](https://www.digicert.com/ssl.htm)
(e.g. when you're on a site like `Gmail` and you see the green lock along with
`HTTPS` at the top of your browser), and this prevents someone from performing a
["man-in-the-middle"](https://goo.gl/HulIhK) attack, where an individual sees `HTTP`
request traffic in plain-text.  However, let's say your API is not secured by `SSL`
or that someone somehow decrypts your communication with the sever... if they see
your username and password being passed with every request, your account is
absolutely compromised.  They can login and wreak havoc.  If the attacked grabs
some obscure token, however, then they must utilize endpoints for all state changes
they wish to perform on your account, which involves a lot of trial and error on their part.  

In addition, I mentioned that a `session_token` has a tunable expiration date.  If an
attacked grabs a `session_token` that is about to expire, this means of accessing
your account is extremely temporary.  This is not the case with a username and a
password.

Finally, sessions give you a lot more control over how the client interacts
with your API and how many permissions they have.  Sessions can contain the
following information (some of which repeats what we've already said):

* The `session_token`
* The session's expiration date
* Permissions (e.g. this session only allows me **read** access to my profile,
nothing else)
* An `update_token`, which the client can send to the server on realizing
that the `session_token` has expired.  This updates the `session_token`,
the expiration date, and the `update_token`.  This means that the `update_token`
is a one-time-use token, so, even if someone uncovers what it is via intercepting
web traffic, it is of little use to them.

# How Should You Design Session-Based Authentication?

The following guidelines suggest a way to implement session-based authentication:

1. Make a database entity / table called `sessions` with `Session` objects containing
3 fields: `session_token`, `expires_at`, and `update_token`
2. Have this database entity / table refer to your `users` entity / table via
some foreign key (if you're using `SQL`) or equivalent (if you're using some
`NoSQL` database)
3. On a user signing into your app successfully, create a `Session` and send it
(all 3 fields included) within the `JSON` response to the sign-in.  This will allow
the client to use the `session_token` in requests, check when the `session_token`
will expire via the `expires_at` field, and eventually send the `update_token`
on `session_token` expiration to refresh the session, this way the user of the
client application doesn't have to re-sign-in.  

Your `session_token` and `update_token` should be `url-safe` and random.  It is very
important to not reveal any details of the user via these tokens.  In addition,
it is protocol to have the user send the `session_token` in the form of an [`HTTP
header`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) of the following format:

{% highlight bash %}
AUTHORIZATION: Bearer <session_token>
{% endhighlight %}  

# Moving onto APIs

Great, now you know about authentication!  We now can move onto HTTP-based APIs
and how to use them.  HTTP-based APIs are basically web-interfaces offered up to
the general public for information-consumption purposes.  These APIs serve up information,
allow one to perform actions on behalf of a user on platforms like Twitter and Facebook,
etc.  To make the picture more clear, I'll describe two examples of two different
web APIs with two equivalently useful purposes below.

The first web API is the [`Pokéapi`](https://pokeapi.co/), a Pokémon API that tons of people
use to consume information about the Pokémon universe.  This API exposes a suite of
GET-requestable endpoints to retrieve info about various components of the Pokémon
universe.  This is clearly a very informative / information-driven API.  

The second web API is the [`Soundcloud API`](https://developers.soundcloud.com/docs).
Soundcloud is a music and podcast streaming platform very similar to Spotify.  Soundcloud's
API is very information-driven as well, allowing one to search and
GET-request particular resources, but it extends beyond an information-serving
purpose.  Given that a user authenticates and packages in a `token` with certain
POST-requests, a user can do things like comment on tracks, follow content-creators,
like songs, etc.  As you can see, being handed this capability via an HTTP interface
opens up a lot of possibilities.  

How does `Soundcloud`, and, by extension, most large companies with popular account-
based platforms, go about giving users' access to their APIs in a secure way?  
Some of these big companies actually use `HTTP Session Authentication` as discussed
above, but many go a step further and utilize an authentication flow standard
called `OAuth 2.0` for maximum stability and security in various use-case driven
situations.  We will describe `OAuth 2.0` below, why it is used for certain
web API use-cases, etc.

# What is OAuth 2.0?
I'm sure you've logged onto some sort of platform like `Grubhub`, `Pinterest`, `Tumblr`,
etc. and found that, conveniently, the site offers `Google` or `Facebook` sign up flows.
On clicking on the `Facebook Sign In` button, you're redirected to a page hosted by `Facebook`,
you sign in, click a button accepting that whatever platform you're onboarding for
will be allowed to access a limited scope of your account, and you're in.  If you've
underwent said experience or similar before, you've experience `OAuth 2.0`.  

`OAuth 2.0` involves 3 different stakeholders:

1. The third-party "client" - in this case a service like `Pinterest`
2. The API "resource server" - in this case `Facebook`
3. The resource owner "user" - the individual who has a `Facebook` account is trying to sign up
for `Pinterest`

All 3 of these entities benefit from the `OAuth 2.0` process.  
* The "client" gets to offer a mainstream sign-up flow that accelerates how quickly people can
onboard to their application.  In addition, the client gets access to functionality
offered by the service they have the "user" authenticate with (e.g. if `Pinterest`
lets me login with `Facebook`, `Pinterest` could then recommend me `Facebook` friends I
have who are on `Pinterest` as potential connections on their platform).  
* The "resource server" popularizes itself and its various features by exposing a subset
of them to the "client" once someone has agreed to give the "client" access to those
procedures.  
* The "user" benefits because he / she only needs one set of credentials for multiple platforms
(just a `Facebook` account for platforms like `Pinterest` and `Tumblr`).  

# Technical Details of OAuth 2.0

Now that we have a better sense of what `OAuth 2.0` involves, let's dive into the
technical details.

Let's track the steps and the flow of `OAuth 2.0`:

#### 1. Registration
An application **A** like `Pinterest` registers itself, as an application,
with some service-provider **B** like `Facebook`.  **A** gets a `CLIENT_ID` and
a `CLIENT_SECRET` that it keeps under lock-and-key because this is used by **B** to
identify what application is making requests.

#### 2. Redirect URI
In addition to receiving credentials from **B**, **A** also designates
a `redirect_uri` to **B**.  This is the `URI` that **B** uses to
redirect the authenticated user back to **A** once the individual signs in /
approves permissions on the **B** site.

#### 3. Code Generation
**A** provides a button on the sign-up / sign-in screen that links the user to
a **B** `URL` with the following generic format:

{% highlight bash %}
https://service-B.com/auth?response_type=code&
  client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=friends,timeline&state=1234zyx
{% endhighlight %}

This generic `URL` has several components that are worth noting, which are common
amongst most all `OAuth 2.0` flows:

* `response_type` is `code` because `Pinterest` will be receiving `code` when `Facebook`
redirects back to `Pinterest` (via the `redirect_uri`)
* `client_id` specifies the `CLIENT_ID` of `Pinterest`
* `redirect_uri` specifies where `Facebook` should redirect to on successful sign-in /
authorization.  An error will be thrown if this `redirect_uri` differs from the one
that `Pinterest` specified on registering with `Facebook`
* `scope` specifies the resources of `Facebook` that `Pinterest` would like to have
access to via users who have authenticated with `Facebook` for `Pinterest`
* `state` is a random `url-safe` string [used for security purposes](http://www.twobotechnologies.com/blog/2014/02/importance-of-state-in-oauth2.html)

On clicking the button that **A** provides, linking to **B**, a user is forwarded to
**B** and signs in.  Signing in and approving the `scopes` requested by **A** results
in **B** redirecting the user to a `URL` hosted by **A** that looks like the following:

{% highlight bash %}
https://client-A.com/cb?code=AUTH_CODE_HERE&state=1234zyx
{% endhighlight %}

We can break down this `URL` as follows:

* `code` is the `code` that **B** generated on successful sign-in.  This `code` is **NOT**
a `token`, but can be used by **A** to request a `token` for use when interacting with **B**'s
API.  More on this very soon.
* `state` is returned to **A** to ensure that the response is actually from service **B**.
As stated before, this parameter is [used for security purposes](http://www.twobotechnologies.com/blog/2014/02/importance-of-state-in-oauth2.html).

Now that **A** has a `code` from **B** representing a user's successful sign-up / approval of
permissions, **A** can hit an endpoint provided by **B** to retrieve a `token` that will
serve, much like in our `HTTP Session Authentication` flow, as a representation of a user
logged into **B** through **A** on making requests to **B**.  The *token exchange* `URL` looks
as follows:

{% highlight bash %}
POST https://service-B.com/token
  grant_type=authorization_code&
  code=AUTH_CODE_HERE&
  redirect_uri=REDIRECT_URI&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
{% endhighlight %}

This `URL` contains a lot of information we've seen before.  This `URL` is where the `CLIENT_SECRET`
comes into play.  It is offered by **A** to **B** to prove that we're trying to authorize
a user on behalf of **A** and not some other, malicious server.  This endpoint takes the
`AUTH_CODE` and generates an `access_token` to be used in requests to **B** through
the client **A**.  The response's `session` looks as follows:

{% highlight javascript %}
{
  "access_token": "a-random-token-generated-by-B",
  "expires_at": 1493145112,
  "update_token": "another-random-token-generated-by-B"
}
{% endhighlight %}

Now that we have this token that represents a user of service **B**, **A**
can request that user's information to get an identifier for the user on service
**B** (some `id` or equivalent), which **A** then stores in a `user` entity / row
in its database, linking the user's account on **B** to the user's account on **A**.
If this user re-logs into **A** via **B**'s `OAuth 2.0` flow, we can request
a `token` again, retrieve the `id` of the user on **B**, and match that up to the
`user` entity / row that already exists in **A**'s database.  

**A**'s app / web application can store the `session` response to authenticate
every request to **A** (which requires calling **B** and validating which
user is making the request).  In addition, this `session_code` can be used to
GET and POST to **B** via **A** to perform certain operations within the original
scopes requested by **A**.

# That's all cool and well, now what?

The best way to improve your understanding of this topic is to explore
APIs available through some of your favorite platforms.  Explore their `OAuth 2.0`
flow and the endpoints they expose to you given certain requested scopes.  Many
platforms provide you an extremely powerful amount of information and capability,
which can greatly improve the quality and utility of your server-side applications.

# Further Readings
* [Session Authentication Cheat Sheet](https://www.owasp.org/index.php/Session_Management_Cheat_Sheet#Session_ID_Properties)
* [Aaron Parecki's OAuth 2.0 Overview](https://aaronparecki.com/oauth-2-simplified/)
* [Why the State Variable is Important](http://www.twobotechnologies.com/blog/2014/02/importance-of-state-in-oauth2.html)
* [Pokéapi](https://pokeapi.co/)
* [Soundcloud API](https://developers.soundcloud.com/docs#authentication)
