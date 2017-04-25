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
