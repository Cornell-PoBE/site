---
layout: post
---

# (1 Week)

# Introduction

This week, we will talk about tier-separation in a backend stack and different
languages that you might write backend in.  This week is heavily driven by
outside resources, so this write-up will be shorter but will contain ample
references to some of our favorite online resources to gain more information
about the topics introduced.

# What is a Tier?

A tier is a layer in a backend stack that relies on the layers around it, can
only function given communication with the layers around it, but has no knowledge
of how the layers around it are implemented.  Tiers stack on each other in a
backend "stack" like pancakes.  Each tier has a separate concern and can be swapped
out with a differently implemented tier with the same expected functionality
and interface without affecting the tiers surrounding it.

The most famous and "well-known" tier configuration is the 3-tier architecture.
It features the following setup:

<br/>

| Layers                   |
|--------------------------|
| Presentation Layer       |
| Business-Logic Layer     |
| Data-Access-Logic Layer  |

<br/>

Let's break down each layer accordingly (descriptions from Tony
Marston's excellent
[overview](http://www.tonymarston.net/php-mysql/3-tier-architecture.html)
of 3-tier architectures).  

#### Presentation Layer
The presentation layer involves the user interface (UI) which displays
data to the user and accepts input from the user. In a web application
this is the part which receives the HTTP request and returns the HTML
response.

#### Business-Logic Layer
The business-logic layer handles data validation, business rules and
task-specific behavior.

#### Data-Access-Logic Layer
The data-access-logic layer communicates with the database by constructing
queries and executing them via the relevant API of whatever database is
involved at the data-access-logic layer.

These layers are helpful because they decouple logic in the application that
can be implemented by different technologies.  In addition, the layers provide
developers with flexibility as the application matures.  Say the original
database is too slow to support the types of traffic your app is receiving
now.  However, your models (different entities within the database and the way
these entities are validated) stay the same.  You can swap out your
data-access-logic layer and nothing would change overall, minus the data
storage and data querying logic.  

# But Wait?  What about MVC?  

We've learned about `Model-View-Controller` in this course so far, and this
architecture is extremely useful for thinking about the responsibilities of
different parts of an overall application.  However, `MVC != 3-Tier Architecture`.

The `MVC` paradigm fits into the `3-Tier Architecture` pattern in the following way:

<br/>

![MVC in 3-Tier](http://www.tonymarston.net/php-mysql/model-view-controller-03a.png)

<br/>

Now, there's a high possibility that throughout this course so far, you've written
a query to the database in a `Flask` controller function.  **This is not necessarily
a bad thing**.  The 3-Tier architecture pattern is **one of many ways of thinking
about application development**.  It has gained so much support because of the flexibility
of language selection per tier, horizontal scalability, software logic decoupling, etc.
that it has provided major software entities for 10's of years now.  However,
not every software stack needs to be absolutely isolated into 3, rigid tiers.

Maintaining the idea of a 3-tier architecture during application development
can help one build well-constructed apps that scale beyond the limits of the current day.

What do I mean by this?

Let's say I am currently limited to one server supporting / serving my website,
answering API calls, and housing the database.  If that single server starts taking
on too much load, I'm going to need to add computing resources to that server (more
RAM, more storage space, etc.).  In this day and age, there is a limit on how many
resources we can allocate to a single machine, and the expense of "vertically" scaling
up this single machine (adding stacks and stacks of resources to it, hence, vertically
scaling it) can get expensive quickly.  Therefore, we might think to "horizontally" scale,
a.k.a. have multiple servers run and manage app traffic.  But all the software is coupled and
reliant on each other within that single server.  A rewrite is required to isolate the
components of the system from each other and place them on separate servers to better
manage load.  Establishing a tier system prevents this rewrite from being necessary.

For additional reading on this subject, see Tony Marston's excellent
[overview](http://www.tonymarston.net/php-mysql/3-tier-architecture.html).

# What About Other Tier Architectures

I'm glad you asked!  You can think about various types of tiers in different ways,
beyond the scopes and responsibilities enumerated in the 3-Tier Architecture.
Below are various tiers that can exist in a backend stack.  

#### Load-Balancing Tier

This tier aims to distribute application traffic on the tier coming after it.
The tier after it generally has several, identical instances, each servicing a
particular amount of traffic.  This tier is a proxy to the rest of the app.

#### Web Tier

This tier serves static files (like HTML pages, JavaScript files, images, etc.)
and routes HTTP / socket requests to different parts of the next tier.  

#### App Tier

This is the tier we've generally been working in so far.  It's your app's main
process.  This might contain state, or it might not.  If there's several
instances of your app, all servicing traffic, it might have limited state (a.k.a.
only a specific subset of users' session information, a small cache, etc.)

#### Cache Tier

A cache aims to store frequently used, but non-permanent data in a way that
is quick to access.  This tier might exist within the same server as the app
layer, or be a large, single entity that several instances of the app layer
access together.  Regardless, it serves to temporarily store information
that is frequently accessed to increase the velocity at which your app can function.

#### Database Tier

This tier involves all things databases.  If you have a custom database configuration,
this tier might contain some additional logic associated with each working server
in your configuration, but in general the database can be thought of as a black
box that protects your apps information and certain guarantees / expected
behavior regarding how it persists, serves up, and manages your data.

# Wait... What do all these words mean?

This is an extremely brief overview of each major tier in most modern application
tier-driven setups.  We encourage you to search up resources to discuss these various
tiers at length.  Our favorite is this [`extremely lengthy and
informative article`](https://www.airpair.com/aws/posts/building-a-scalable-web-app-on-amazon-web-services-p1)
by Josh Padnick, multi-tier / service-oriented architectures and how one
can use AWS to craft scalable backends.  

# Okay, Enough About Tiers.  Let's Talk Languages

I'm glad you asked!  Up to this point, we've used exclusively `Python` and `Flask`
to develop applications.  But there exists other alternatives, with positives
and negatives.


#### Node.js

`Node.js` isn't a backend framework, but rather a JavaScript runtime, running on
top of `Google Chrome's V8 Engine`.  `Node.js` allows developers to write applications
that run outside of the browser in `JavaScript`, be it command-line utilities,
web-scrapers, you name it.  A large amount of effort has been put into [`Express.js`](https://expressjs.com/),
which is the premier, minimalist `Node.js` web application framework.  `Node.js`
is a solid choice for individuals interested in `asynchronous` programming.  While
`JavaScript` is rarely considered a "functional" language, it has functional
aspects that appeal to individuals who enjoy writing server-side functionality
in an event-driven manner.  `Node.js` has very solid socket support, as well,
so individuals interested in writing applications that utilize sockets, rather than HTTP,
as their primary means of information exchange should look towards `Node.js` as a
possible language to leverage.  In addition, due to its event-driven nature,
`Node.js` is commonly used at the `Load-Balancing Tier` of large backend deployments.
`Node.js` is generally criticized because it involves..... `JavaScript`, which many
programming-language experts find to be a terrible language, for reasons that generally
extend beyond the scope of this article.  However, there's many modern tools that
allow developers to reason about `JavaScript` typing ([`TypeScript`](https://www.typescriptlang.org/),
[`Facebook's Flow Static Type Checker`](https://flow.org/)), `JavaScript` asynchrony ([`Promises`](https://developers.google.com/web/fundamentals/getting-started/primers/promises)),
etc. that make `JavaScript` a lot more comfortable to use than it used to be when `Node.js`
initially became popular.  `Node.js` is used by companies like `LinkedIn`, `Uber`,
`Netflix`, and `Medium`, usually at the App Tier.  

#### Python

`Python` is a very expressive language and has been successfully used for a multitude of
tasks for several years now.  [`Flask`](http://flask.pocoo.org/) is one of the most famous
frameworks to use with `Python` to write modern web applications.  We chose to intro this
course with `Flask` because it is a micro-framework.  It gets out of your way and
lets you express your logic using as much of it or as little of it as you want.  
We enjoy it for that reason.

On the flip side, a framework called [`Django`](https://www.djangoproject.com/), which
a framework that's a lot more opinionated regarding the prerequisite application
architecture.  It's also very reliant on subclassing `Python` classes to service
different types of HTTP requests.  It's a well-supported and solid framework for sure,
and appeals to individuals who enjoy having pre-built solutions for various, day-to-day
web tasks encountered when writing a backend.

`Python` has both synchronous and asynchronous packages for managing I/O-heavy tasks.
It's also very clean and clear to write, making scripting and adding functionality incredibly
quick and easy.  `Python`, however, has a weaker OOP structure because when `Python` as
a language was written, its OOP features were some of the last things added, and this makes
designing software that's dependable a bit harder than writing, say, `Java`, which protects
developers from making certain mistakes due to its strong typing.  In addition, because `Python`
is dynamically typed and interpreted, it can run slower than
statically-typed, compiled languages (languages that have type information established
at compile-time, rather than having to recompute type information dynamically at runtime).  

`Python` is used by companies like `Yelp`, `Quora`, and `Facebook` at the App Tier and
to express database logic involving datastores like `MySQL`, which have very solid and
powerful API drivers in `Python` that are easy to script with.  

#### Java

`Java` is either a language you have learned or will learn in the future.  It's seen by
many as "old school" and "verbose", but `Java` is used by tech giants and start-ups alike
due to its reliability, static-typing, phenomenal support, and commitment as a language
to Object-Oriented methodologies.  Many frameworks written for `Java` take these elements
of the language into account.  Notable frameworks include [`Spring`](https://spring.io/),
which involves a powerful software engineering ideology called
[`dependency injection`](https://www.youtube.com/watch?v=IKD2-MAkXyQ) and has a suite of tools
and an ecosystem that branches outside of the REST API space,
[`Drop Wizard`](http://www.dropwizard.io/1.1.0/docs/), which prides itself in its
extremely performant design and sticks to handling web traffic really, really well, and
[`Play`](https://www.playframework.com/), which is more of a "full-stack" framework than just
a server-side-oriented `Java` tool-kit.  

`Java` is a compiled language, so it's relatively quick in its execution.  It can also
multi-thread like a boss.  In general, it allows programmers to build reliable software that
is built on tried-and-true OO design patterns.  On the flip side, `Java` can be overly
verbose and filled with boilerplate, when compared to other languages that might serve as a
similar means to an end.  What takes 5 lines in a `Python` framework might take 15 in
a `Java` framework.  In addition, due to `Java`'s imperative programming model, it
is easy, as a programmer, to make mistakes regarding the state of a program's execution,
when compared to programming in a functional language that features immutable data-structures
and state.  

Companies like `Amazon`, `Palantir`, `Fitbit`, `Uber`, `Airbnb`, `HubSpot`, and `Riot Games`
use some of the frameworks listed above.  

#### Ruby

[`Ruby`](https://en.wikipedia.org/wiki/Ruby_(programming_language)), as a language,
was made by Yukihiro "Matz" Matsumoto and designed to "make programmers happy."  In general,
`Ruby` is an incredibly flexible language, designed to make it incredibly easy to
perform certain tasks.  In addition, there's usually more than one way to perform certain
tasks in `Ruby`, so the language leaves it up to the programmer to decide how they'd like
to write their code.  [`This post`](https://maori.geek.nz/what-is-ruby-it-is-fun-and-makes-you-happy-337b6f10fa40)
enumerates several more reasons why `Ruby` is awesome.

The premier framework written for `Ruby` is [`Ruby on Rails`](http://rubyonrails.org/),
and this behemoth of a framework has been around forever, meaning there's **a ton** of
support on the Internet, in the form of `StackOverflow` posts, guides, blogs, you name it.
This framework is blazing fast to use and can allow backend developers to go from
0 endpoints to an entire product in no time at all.

But there's a catch, or so we've found.  This velocity of development is met with poor runtime
performance, hidden bugs due to `Ruby`'s dynamically-typed, interpreted nature,
and a lot of ["rails magic"](https://medium.com/ruby-on-rails/demystifying-the-magic-of-rails-416d2195f098)
that can lead to issues down the line.  That being said, if you have a solid hold of
the `Ruby` language and can optimize your `Rails` app to be performant in a modern setting,
this framework will support you throughout that process.  However, many modern companies that
started on `Rails` end up converting their entire codebase down the line.  Said companies include `Square`,
`Airbnb`, `Twitter`, [`Parse`](http://blog.parse.com/learn/how-we-moved-our-api-from-ruby-to-go-and-saved-our-sanity/),

However, `Rails` is awesome for prototyping and can be used in production, regardless of its limits
at scale.  Companies like `Amazon`, `Github`, `Bloomberg`, etc. uses `Rails` to power parts of their site via
a microservices approach.  

#### Others

The above languages are ones we have extensive server-side development experience with.  
Others we did not include, and corresponding frameworks they can be used with, are listed below:

* `Go` with [`Iris`](http://iris-go.com/)
* `PHP` with [`Laravel`](https://laravel.com/) or [`Slim`](https://www.slimframework.com/)
* `Scala` with [`Akka`](http://akka.io/) or [`Play`](https://www.playframework.com/)
* `OCaml` with [`Eliom`](http://ocsigen.org/eliom/)
* `C++` with [`CppCMS`](http://cppcms.com/wikipp/en/page/main), [`TreeFrog`](http://www.treefrogframework.org/), or [`Silicon`](http://siliconframework.org/)
