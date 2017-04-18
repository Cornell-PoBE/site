---
layout: post
---

# (2 Weeks)

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
* We want **todo-list items** that belong to one **todo-list**

The following sections overview different aspects of `SQL` (leveraging `MySQL` paradigms)
by referring to the above example system.  We demonstrate how to use `SQL` to craft
constraints and relationships that set ourselves up well for data storage according to
the business-case needs of this todo-list application.  

# Syntax for Creation and Insertion

Let's start with the todo-lists.  Todo-lists have *a lot* of information associated with them:
users who own them, items that they own, and metadata information.  For now, we would like to
focus on the metadata information associated with them, and will slowly introduce the "glue"
that allows us to connect users and todo-list items to these todo-lists in the near future.

What information do `todo-lists` have?  

* Some unique identifier for the todo-list
* The name of the todo-list
* When the todo-list was created

The syntax used to establish the above constraints would be:

{% highlight sql %}
CREATE TABLE todo_lists (
  id INT NOT NULL AUTO_INCREMENT,
  name VARCHAR(255) NOT NULL,
  created_at DATETIME NOT NULL DEFAULT GETDATE()
);
{% endhighlight %}

Let's unpack the above.

As we can see, we've attempted to nail all three fields that are required of us in order to
represent the `todo-list` relation.  We've defined an `AUTO_INCREMENT` `id` field, meaning
that every new tuple in the `todo-list` relation is assigned an `id` field that is strictly
greater than every other `todo-list` tuple to exist in the database previously (so it's
guaranteed to be unique).  The `name` field is defined as a `VARCHAR` data-type with a maximum of
255 characters allowed.  The `created_at` field is defined as a `DATETIME` data-type.  Its
default value, if unspecified on creation, grabs the datetime at which the tuple is being created
(a.k.a. the system's clock value on whatever system is running the `SQL` database).  In addition,
we have slapped on some `NOT NULL` constraints to prevent poorly-formatted tuples with `NULL` values
for important, prerequisite fields like `name`.  As we can see, given the above relation structure,
we can only provide the name of our todo-list and the database does the rest in terms of data-storage,
field assignment, and mild error-handling, all for free!  

To add a single `todo-list` tuple to the database, we can write the following:

{% highlight sql %}
INSERT INTO todo_lists (name)
VALUES ('My Awesome Todo-List!');
{% endhighlight %}

# Primary Keys

You might think, why do we need a unique identifier for this todo-list?  Isn't the name enough?
It is here that we will introduce our first major concept of good database design: **primary keys**.
I'd like to cite Cornell's `CS 4320` course materials in order to define a primary key:

A **primary key** is a set of 1 or more fields.  An invariant that *must* be maintained amongst
the tuples in your database is:
* No two distinct tuples can have same values in all key fields

This allows one to uniquely identify tuples in your database, which is useful for **querying** and
**relating** different relations, which we'll see in the next section.  Now, we mentioned that
the name of the todo-list might be a good unique identifier for a todo-list, and it very well could
be depending on our application's logic.  However, given the logistics of the app, uniquely identifying
a todo-list via name would mean that only one grouping of people who maintains that todo-list will ever
get the todo-list name "Grocery Shopping", and "Wedding Todos", etc.  We shouldn't restrict one
subset of users from accurately expressing their todo-list needs via its name, just because some other
arbitrary subset of people had that todo-list need first!  Therefore, we need a more global, better
measure of uniqueness.  This could be a combination of fields that must guarantee unqiueness, or one
numerical- or string-based field that is generated each time a new todo-list item is created.

The above example of creating the `todo-lists` relation establishes an absolutely unique primary key
field called `id`.  We can formally declare it as the primary key of the relation via the following
syntax:

{% highlight sql %}
ALTER TABLE todo_lists
ADD PRIMARY KEY(id);
{% endhighlight %}

Obviously our `id` field is already unique because it leverages `AUTO_INCREMENT`, however, the above
code allows the database to formally categorize the `id` field as the **key** for `todo-list` tuples,
as well error-handle any potential inserts that accidentally specify a previously used `id` field,
rather than letting the database assign the `id` itself.  

# Foreign Keys - Linking Tuples Together

Great!  Now we've set up our `todo-lists` relation.  Let's next tackle the `todo-list-items` relation.

What information do `todo-list-items` have?

* A unique identifier
* A description
* A `true / false` means of articulating whether this item is completed or not
* When the todo-list-item was created
* A means of referring to the owning `todo-list`

Before we talk about any of the *informational* needs `todo-list-items`, let's first discuss how we'd
manage to create a reference to the owning `todo-list` tuple of an arbitrary `todo-list-item` tuple.

In this situation, each `todo-list-item` tuple belongs to exactly *one* `todo-list` tuple, meaning
that we can safely have *one* field in our `todo-list-item` relation representing that `todo-list` tuple.
We'd like a way to capture exactly what `todo-list` tuple we're referring to in a concise and clean way.
We can use the `todo-list` tuple's **key** to refer to which `todo-list` tuple owns the `todo-list-item`
tuple.  We can, therefore, define a **foreign key**, which contains the primary key of the `todo-list` tuple
that owns the `todo-list-item` tuple.

We can define the `todo-list-items` relation in the following way:

{% highlight sql %}
CREATE TABLE todo_list_items (
  id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  description VARCHAR(255) NOT NULL,
  completed BOOLEAN NOT NULL DEFAULT FALSE,
  created_at DATETIME NOT NULL DEFAULT GETDATE(),
  FOREIGN KEY (todo_list_id) REFERENCES todo-lists (id)
);
{% endhighlight %}

# Data Relationships

The ideas of foreign keys is very powerful, as it is the crux of linking together data in a `SQL`-system.
Via foreign keys, we can establish *relationships* between tuples.  We've seen the situation articulated
above, which involved a `todo-list-item` tuple *belonging to* a `todo-list` tuple.  This is a `one-to-many`
relationship, where **one** `todo-list` tuple is matched up to [potentially] **many** `todo-list-item`
tuples.  There are two other common relationships that exist in relational databases: `one-to-one` and  
`many-to-many`.  Below we list all 3 and describe the relational patterns used to create these relationships:

* `one-to-one`: A relationship where tuple **a** of relation *A* is associated with one
and only one tuple **b** of relation *B*, and vice versa.  Therefore, relation *A* has a
foreign key `B_ID` referring to *B* and relation *B* has a foreign key `A_ID` referring to *A*.  
  * **Example**: [`country` and `capital city`](https://en.wikipedia.org/wiki/One-to-one_(data_model))
* `one-to-many`: A relationship where tuple **a** of relation *A* is associated with one or more
tuples of relation *B*, but tuple **b** of relation *B* is associated with one and only one tuple from
relation *A*.  Therefore, *B* has a foreign key `A_ID` referring to *A*.  
  * **Example**: `todo-list` and `todo-list-item`
* `many-to-many`: A relationship where tuple **a** of relation *A* is associated with
one or more tuples of relation *B*, and tuple **b** of relation *B* is associated with
one or more tuples of relation *B*.  Since no side of this relationship paired with
exactly one tuple of the other side, it does not make sense to have a single foreign
key field present on either side of the relationship, as this does not express to fact
that the both sides can be associated with *many* of the other side.  Therefore,
we must define a third relation *C* called a **JOIN relation / table** or **relationship
table**, which has two foreign keys: `A_ID` and `B_ID`.  For every tuple **c** of
relation *C*, there is a relationship between a given **a'** of *A* and **b'** of *B*.  
By having this third relation, we can cleanly express *A*'s relationship with *B*.  
  * **Example**: `beers` and `bars` (a beer is sold at many bars and a bar sells many beers).

As we can see, the above techniques are very powerful in terms of establishing relationships
between data, as a lot of the models in an application's business logic are related to each
in the above 3 ways.  

# Querying and Synthesizing Data

Now that we have some basic ideas behind how we'd model data and have articulated our full
`todo-list` example database **schema** (our relation structure and whatnot), we can move
onto the actual syntax and semantics behind grabbing our data once we've figured out how
to model it.  

This section is not meant to enumerate all the features of `SQL` querying, but rather expose
the reader to *topics* related to `SQL` querying.  In the **Further Reading** section, we have
listed some of our favorite learning resources regarding `SQL` querying.  

The classic `SQL` query structure is the following:

{% highlight sql %}
SELECT [DISTINCT] target-list
FROM relation-list
WHERE conditions
{% endhighlight %}

Let's decompose the above:

* `SELECT` allows one to choose which attributes we're extracting from the query.   
* `DISTINCT` allows one to only return single tuples of a particular structure (a.k.a.
if multiple tuples have the same set of fields being selected on, only one of each
will be returned).
* `target-list` refers to the list of attributes we're extracting (e.g. `username,
email, phone_number`, or `*` for all returned attributes).
* `FROM` allows one to choose which relations we're grabbing information from.
* `relation-list` refers to the list of relations we're getting info from (e.g.
`todo-lists, todo-list-items`).
* `WHERE` allows one to specify boolean conditions on which data is returned.
* `conditions` is a series of boolean conditions (e.g. `users.first_name = 'John'
AND users.age > 25`)

If we only care about information from one relation, `relation-list` only has one
relation listed.  However, a common need in an application's logic is to find
"matching pairs" from two relations (e.g. return an entire country's information
and its capital's information as one big tuple).  In order to synthsize relations
that are bound together via `foreign key` relationships, one needs to understand
the concept of **SQL JOINs**.  **JOINs** quite literally "join" data together.  

Below is an example of a `JOIN` query that lists tuples of `todo-list` information
and `todo-list-item` information combined into "super" tuples.  This, of course,
refers back to our `todo-list` example articulated above:

{% highlight sql %}
SELECT *
FROM todo_lists tl, todo_list_items tli,
WHERE tl.id = tli.todo_list_id;
{% endhighlight %}

**NOTE:** `tl` and `tli` serve as *aliases* for the relations so we don't have to
write the entire relation name every time we refer to an attribute within
the relation.  

One can think of the above as performing the cross-product of `todo-list` tuples
and `todo-list-item` tuples (a.k.a. every possible tuple pairing) and then only
returning the resultants where `tl.id = tli.todo_list_id`.  

This is a very basic `SQL JOIN` example.  More exist [here](https://www.w3schools.com/sql/sql_join.asp)
and online  (see our **Further Reading** section).  

# Referential Constraints, Indexing, and Other Bells and Whistles

As we can see, `SQL` and relational databases are very powerful in terms of data
storage, data modeling, and querying.  But what else do they give us?  As we saw
previously when declaring tables, `SQL` provides us with constraint-based error-detection.  
We can flag attributes as `NOT NULL`, given attributes `DEFAULT` values, etc.  We
can use these constraints to maintain attribute-level invariants, but also
application-wide invariants.  

For example, it is desirable to enforce *referential constraints* involving the
relationships present in your application.  What does this exactly mean?  
Well, say the following happens: a `todo-list` (from our running example)
gets deleted.  What happens to all its `todo-list-items`?  Now all those `todo-list-items`
refer to a `todo-list` that doesn't exist.  What happens if we didn't have `AUTO_INCREMENT`
attached to the `todo-list` relation's `id` field and another `todo-list` tuple is
made with the old, deleted `todo-list`'s `id`?  Do all those old `todo-list-items` now
belong to this new `todo-list` tuple?  As we can see, this situation gets pretty messy when
these `todo-list-items` are left swinging in the wind.  

`SQL` lets us specify actions, given an occurrence like above, where the relationship
between data breaks:

1. The default is `NO ACTION`
2. `CASCADE`, a.k.a. delete tuples that refer to the deleted one (via foreign key)
3. `SET NULL / SET DEFAULT`, a.k.a. set the foreign reference to `NULL` or some default value

This very powerful, as we can pass this type of logic reliably off `SQL`.  Anything
that involves less application-level logic for us, as backend developers, is good

Outside of specifying constraints, `SQL` lets us **index** particular attributes in
order to increase the velocity at which we get are returned results on executing
particular queries.  TODO 
