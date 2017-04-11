---
layout: post
---

## Relational Databases

# A Short History

In any sort of meaningful application, various entities that exist within the logic of
your app have *relationships* with other entities.  Whether it be a user's friendship
with another user, a section belonging to an overall course, etc., any means of
representing one's application logic will involve addressing these relationships.
Relational databases make these said "connections" between different entities that
can be stored as data on disk their primary focus.  

Relational databases emerged in the 1970s and have been a maturing technology at the
heart of academia and industry since then.  Research regarding relational databases
and their capabilities ability to optimizing for particular use-cases is being
performed all across the globe.  The `Introduction to Databases` (`CS 4320`)
course here at Cornell makes them and their academic principles the focus of the first
~60% of the course.  Companies like `Facebook`, `Pinterest`, `Netflix`, `NASA`, etc. use
these data stores for various use-cases due to their robustness, velocity of reading /
writing information, general technological maturity, and a host of other reasons.  

# Different Kinds

In general, the idea of a "relational database system" is a model, a paradigm for
management of data in a particular way.  Several large companies in the tech world
have implemented that model in their own ways to achieve solutions that, while similar
in their base representation of a what a relational database system is and how user's
interact with it, differ slightly in their execution and overall utility.  Some of these
implementations include `MySQL`, `PostgreSQL`, `MariaDB`, `Microsoft SQL Server`,
`Amazon SimpleDB`, etc.  We will be focusing primarily on `MySQL` and `PostgreSQL` in
this course, as they, in our opinion, are the easiest systems to download and immediately
start being productive with.  

# Wait... What is SQL?  

Now, in the two databases we have selected to focus on, we see the acronym `SQL` - what
does this stand for?  `SQL` stands for "**S** tructured **Q** uery **L** anguage".  Essentially,
when you're dealing with a `SQL`-based system, there is a **structured** way to write data
-insertion,data-deletion, and data-reading operations via this `SQL` language.  In
deconstructing the acronym we noticed an important term: "query".  "Query" describes any
means of expressing a desire for a particular type of information.  "Is Matt Damon
married?" is a query.  "How many years of programming experience does David Gries have?"
is a query.  "I want all names that begin with 'A'" is a query.  It's this language `SQL` mixed
with the relational-database ideology (which we are about to discuss) that make storage of
data in a SQL system so logical and pleasant from a user's perspective.  

# Overall Relational Structure

Relational databases are very easy to understand from a high-level if you've used spreadsheet
programs like `Excel` or equivalent.  An app's information is stored in a **database**, with
**tables** representing different entities in the application.  For example, one might have a
database called `my_twitter_clone` with tables `users` and `tweets`.  These **tables** can also
be dubbed **relations**, and we will prefer the latter term for the remainder of our
discussion of relational databases.  These tables have both **columns** and **rows**.  The
**columns** describe the attributes that relations have, such as `name, email, address`
for our `users` relation.  The **rows** describe the data in the table itself.  With every
row is a new `user`, or a new `tweet`, or whatever your relation is called.  **Rows** can also
be dubbed **tuples**, and we will prefer the latter term for the remainder of our discussion, as
well.

# A Clarifying Example

Let's walk through an example to better understand the relational-database model and `SQL`.  
Let's say I'm remaking the todo-list app that was assigned for `A1`, but with some extended
functionality.  Let's express that functionality in full here:

* We want **todo-lists**
* We want **todo-list items** that belong to one todo-list
* We want **users**
* We want todo-lists to be **owned** by 1 or more users (I could potentially own
  one todo-list with my friends, our "group todo list")

Let's start with the todo-lists.  Todo-lists have *a lot* of information associated with them:
users who own them, items that they own, and metadata information.  For now, we would like to
focus on the metadata information associated with them, and will slowly introduce the "glue"
that allows us to connect users and todo-list items to these todo-lists in the near future.

What metadata do todo-lists have?  

* The name of the todo-list
* When the todo-list was created
* Some unique identifier for the todo-list

You might think, why do we need a unique identifier for this todo-list?  Isn't the name enough.  
It is here that we will introduce our first major concept of good database design: **primary keys**.
