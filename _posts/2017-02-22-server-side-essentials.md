---
layout: post
---

# (1 Week)

## Client-Server Model

# Request-Response

The client-server model partitions tasks or workloads between the providers of a resource or service, called servers, and service requesters, called clients.
A server host runs one or more server programs which share their resources with clients. Clients themselves do not share any of their resources, but request a server's content or service function and starts communication sessions with servers which await incoming requests. The World Wide Web leverages this style of distributed application architecture.

![Client-Server Architecture](
https://upload.wikimedia.org/wikipedia/commons/thumb/c/c9/Client-server-model.svg/500px-Client-server-model.svg.png)

The relationship itself can be described by saying that the server provides a function or service to one or many clients, who initiate requests for these services. Servers are unique and can be described by the function that they serve. A web server for instance serves web pages while a file server serves files. The sharing of resources with the client is what makes these abstractions: services.

What is important to note is that the service functions as a black-box abstraction where the client does not need to know how the server handles the request and response. As such, the client only needs to understand the input format of the request and the output format of the returned response. This format is defined by an application communication protocol which we will elaborated upon later in the lecture.

Clients and servers communicate via the exchange of messages in a request-response messaging pattern where the client sends a request and awaits a response from the server. This communication exchange is a subset of inner-process communication which is the method which unique and seperate processes share data. The request-response messaging pattern leveraged by the client-server model is quite ubiquitous and is commonly seen in distributed computing. However, as expected, it is not the only communication model that exists under IPC. We will be studying others as well, for example Pub-Sub. For eager readers, [here](http://www.mdpi.com/1424-8220/12/6/7648/pdf) is an interesting article that reviews both models and their appropriate advantages.

# Communication Protocols

Just as we humans must have common semantics and syntax to speak with each other, for computers to communicate they too must have a common language and an established set of rules so that both the client and the server know what to expect. These sets of rules describe what a client and server expect in their communciation with each other. This "language" and these rules are defined by the communication protocol.

The communication protocol is a system of strict rules that would allow a set of communication systems to trasmit information in previously agreed upon well-defined format. There are multiple protocols that are combined to describe a single communication. These group of protocols are combined to make up the protocol suite, where we will be focusing on the protocol stack (the software implementation). The following figure shows an example computer networking protocol suite in increasing rank of abstraction:


| Protocol      | Layer           |
| --------------|-----------------|
| HTTP	        | Application     |
| TCP/UDP       | Transport       |
| IP	          | Internet/Network|
| Ethernet	    | Data Link/Link  |
| IEEE 802.3u	  | Physical        |


In this class, you will only need to understand three protocols within the transport and application layer, at a very high level. However if you would like to dig deeper into networking, definitely check out the networking slides from CS4410, 2015fa, [here](http://www.cs.cornell.edu/courses/cs4410/2015fa/slides/08-networking.pdf).

In this class we will focus on the Internet Protocol (IP) which is a principal protocol communication stack for sending messages, called datagrams, across the network. This routing of messages is fundamental for the internet and is the basis for internetworking. You can review the IP protocol suite [here](https://en.wikipedia.org/wiki/Internet_protocol_suite). The three key ones we will review are User Datagram Protocol (UDP) and Transmission Control Protocol (TCP) from the transport layer, and Hypertext Transfer Protocol (HTTP) from the application layer.

User Datagram Protocol is a protocol defining the method of sending messages or datagrams across an IP network. Datagrams are just transfer units structured in header and payload sections that are sent across a packet-switched network. These datagrams provide connectionless communication by loading important header information like destintation/source and checksums in the header and the data in the payload. You can read more about this in the slides linked [above](http://www.cs.cornell.edu/courses/cs4410/2015fa/slides/08-networking.pdf). One important thing to note is that UDP does not have handshaking, the back-and-forth communication which is necessary for reliability in the network. As such, there is no guarantee of delivery, ordering, or duplicate protection. If we are to seek such reliability via handshaking this can be provided via TCP.

Transmission Control Protocol is a protocol defining the method of sending different types of messages (we will focus on UDP-like datagrams) with the inclusion of a handshaking routine to provide reliable, ordered, and error-checked delivery of datagrams being sent across an IP network. TCP is a fairly complicated and intricate protocol that is well-addressed in these [videos](https://www.youtube.com/playlist?list=PLB7C6Y9smovkPRzGLsa_GaKc5DOvr6UDd), if you are interested in looking into it further (optional for this class).

Hyptertext Transfor Protocol is the main application protocol for exchanging and transferring hypertexts (logical links between nodes) that acts as the overarching framework for the request-response protocol in the client-server computing model described above. Because HTTP is an application layer protocol it leverages the use of some form of underlying and reliable transport layer protoocl like TCP. As such, HTTP treats the lower level layers, like the TCP transport layer, as black boxes that provide a stable network connection across which to communicate. HTTP defines the methods to indicate the desired action to be performed when the client is making a request and awaiting a response from the server. We will walk through each of the possible requests and the request-response lifecycle.

# HTTP Request-Response Lifecycle

We know that HTTP is a common protocol to pass around HTML amongst other forms of data i.e. JSON by providing stateless data exchange, between the client and the server. This data exchange is achieved via request-response communication. HTTP defines request methods that each uniquly indicate the desired action to be performed on the identified resource. This method specification is either in the form: HTTP/1.0, which define the GET, POST, and HEAD methods, or: HTTP/1.1 which define the OPTIONS, PUT, DELETE, TRACE, and CONNECT methods. Recently there was an additional HTTP method added: PATCH.

| Request       | Description     |
| --------------|-----------------|
| GET	        | requests a representation of the resource; data retrieval |
| HEAD          | identical to GET w/e the response body  |
| POST          | requests that server accepts an entity enclosed in the request |
| PUT           | requests that the entity enclosed in th request be stored under the UI |
| DELETE        | requests to delete a specified resource |
| TRACE         | requests to echo the recieved request to see changes to servers |
| OPTIONS       | requests to see all server-supported HTTP methods |
| CONNECT       | request that converts request connection to a TCP/IP tunnel for SSL |
| PATCH         | request that applies some form of modification on the resource |


It is important to note that only the PUT and DELETE requests are defiend to be idempotent. Idempotence means that the operation will produce the same result if it is executed once or multiple times. For methods : GET, HEAD, OPTIONS, and TRACE to be declared idempotent they must be prescribed as safe. POST requests naturally won't be always idempotent as sending multiple identitical POST requests could further alter the state of the resource which may or may not be desirable.

It is necessary to understand GET,POST, and PATCH as those will be leveraged heavily in your APIs throughout the class.

The request format will be sent as:

* A request line with the above formats (i.e. GET /your_api/endpoint)
* Request header fields
* Empty line
* Message body (optional data or request entity)

To complete the request-response communication you will obviously recieve a response from the server. The response contains a status code which allows the user-agent to handle the response primiarily based on the status code and secondarily on the other response header fields. Response header fields are responsible for telling the user-agent the format in which the message body or data sent will come in: text/json, text/html, etc. HTTP status codes are dividied into five groups:

1. Informational: 1XX
2. Successful: 2XX
3. Redirection: 3XX
4. Client Error: 4XX
5. Server Error: 5XX

The response format will be recieved by the client as:

* A status line which includes the status code and a reason message (i.e. HTTP/1.1 200 OK SUCCESFUL)
* Response header fields (i.e. Content-Type: text/html)
* Empty line
* Message body (optional data sent over from server)

Now that you have knowledge of the formatting and structure of an HTTP request and response, I will walk through the request-response lifecycle and handling which can be broken down into 5 steps. I will show an example of an every-day get request you would do when access a web-page on a browser.

#### 1. Parsing the URL

Firstly, a URL is typed into the address bar of a web browser. URL is an alias to a unique IP address which you can read all about [above](http://www.cs.cornell.edu/courses/cs4410/2015fa/slides/08-networking.pdf). The matching IP address to the alias is a unique number sequence which is the server you are trying to collect resources from, this target server can be called the Host.

The browser next will check for the IP in its browser cache. The browser cahce lets you set aside a section of your computer’s hard disk to store representations that you’ve seen. This cache is used when users hit the “back” button or click a link to see a page they’ve just looked at. Also, if you use the same navigation images throughout your site, they’ll be served from browsers’ caches almost instantaneously. To learn more about caches read [this](https://www.mnot.net/cache_docs/). If the IP is not found in the browser cache it sends a request to DNS. the DNS or Domain Name System is a comprehensive directory network translating domain names into its unnique IP sequence. The browser then parses the URL to the find the Protocol Type, Host, Port, and request-URI path in the form: "[protocol]://[host]/[request-URI]". The Protocl Type would be HTTP in this case. The HOST would be the server which the browser will contact. The Request-URI or Universal Resource Identifier would be used by the server to identify the resource location.

The browser then forms a HTTP request given the URL. Lets take a simple example: typing in: www.google.com. This would be a  GET /URI HTTP/1.1 to the host www.google.com.

#### 2. Sending the request

A soceket connection is now opened from the user to the IP address where an HTTP request is sent to the host and the user waits to get a response back. The web server would recieve the request at this stage.

#### 3. The server response

The Host or target server checks the request format, the headers and the request method type. If the request is static the server locates the html filename and sends that file back over the internet. If the request is dynamic, all necessary files are processed containing changeable sections depending on variable values given to the page on the server end. The dynamic data and the requested file will be combined into a long string of text (HTML, .txt, or XML) before its output is sent back over the internet. The response the server sends wil have the status code illustrating whether or not the server successfully processed the request.

#### 4. Browser rendering

The browser now receieves the response with the HTML document that it will then parse into a Document Object Model or DOM. It does this parsing by translating HTML elements and attributes into nodes with the "Document Object" set as the root node of the tree. When external scripts, images, and style-sheets are parsed, new requests are made to the server, with applicable styles getting attached to the appropriate corresponding node in the DOM tree. Javacsript files are parsed and executed next with the DOM nodes updated (but this information is not necessary for this class). In parallel to the DOM tree being consturcted, the browser would construct a "render tree" that acts as the visual representation of the node elements which the browser will use to render the page.

#### 5. HTTP persistent connection

If the HTTP request includes "Connection: Keep-Alive" in the header field there is a specific series of steps that must be undergone after the browser renders the page. It will initiate a single TCP connection for sending and recieving multiple HTTP requests-responses, instead of opening a new connection every single request-respone pair.  With this header set in the initial browser request it informs the server to not drop the connection, so that when the client sends another request it uses the same connection until the client or server decides to drop the connection.

# Model-View-Controller
Model-View-Controller (MVC) is a software architectural pattern for implementing user interfaces that divides an application into three interconnected parts in order to separate internal representations of information from the ways that information is presented to and accepted from the user. This design pattern decouples these major components to allow for efficient code reusability and parallel development. The three components are the following:

Model:
The Model component corresponds to all the data-related logic that the user works with. This can represent either the data that is being transferred between the View and Controller components or any other business logic-related data. For example, a Customer object will retrieve the customer information from the database, manipulate it and update it data back to the database or use it to render data.

Controller:
Controllers act as an interface between Model and View components to process all the business logic and incoming requests, manipulate data using the Model component and interact with the Views to render the final output. For example, the Customer controller will handle all the interactions and inputs from the Customer View and update the database using the Customer Model. The same controller will be used to view the Customer data.

View:
The View component is used for all the UI logic of the application. For example, the Customer view will include all the UI components such as text boxes, dropdowns, etc. that the user interacts with.

I will go through an example flow below:
![MVC Design](
https://betterexplained.com/wp-content/uploads/rails/mvc-rails.png)

* Browser makes a request
* Web server receives the request
* Routes finds out which controller to use: the default route pattern is “/controller/action/id"
* The web server then uses the dispatcher to create a new controller, call the action and pass the parameters.
* Controllers do the work of parsing user requests, data submissions, cookies, sessions, ... etc 
* Model talks to the database, in this case MySQL, store and validate data, and perform the business logic
* Views are what the user sees: HTML, CSS, XML, Javascript, JSON
* Controller returns the response body (HTML, XML, etc.) & metadata (caching headers, redirects) to the server. The server combines the raw data into a proper HTTP response and sends it to the user.

I will now elaborate on how we will be using the MVC pattern design within a Flask app. 

# Flask App

Flask uses a concept of blueprints for making application components and supporting common patterns within an application or across applications. Blueprints can greatly simplify how large applications work and provide a central means for Flask extensions to register operations on applications. A Blueprint object works similarly to a Flask application object, but it is not actually an application. Rather it is a blueprint of how to construct or extend an application.

# Accounts Blueprint

Now we will be diving into writing `Models` and `Controllers` for an `accounts` blueprint that will serve as reusable `users-sessions` module that can be added to any application desiring a sign-up system.  We make some short-cuts along the way in order to increase this guide's brevity, while still providing meaningful sample code and explanations for the different components of the system.  

We must create our module within `app`, such that it contains the following structure:

{% highlight bash %}
.
├── __init__.py
├── controllers
│   ├── __init__.py
│   ├── sessions_controller.py
│   └── users_controller.py
└── models
    ├── __init__.py
    ├── session.py
    └── user.py
{% endhighlight %}

Let's start with `./app/accounts/__init__.py`.  This file contains a couple of lines of information specifying the `Flask Blueprint` information of this module, as well import `controllers`:

{% highlight python %}
from flask import Blueprint

# Define a Blueprint for this module (mchat)
accounts = Blueprint('accounts', __name__, url_prefix='/accounts')

# Import all controllers
from controllers.users_controller import *
from controllers.sessions_controller import *
{% endhighlight %}

In addition, register your new blueprint in `./app/__init__.py` by changing the lines:

{% highlight python %}
# Import + Register Blueprints
# WORKFLOW:
# from app.blue import blue as blue_print
# app.register_blueprint(blue_print)
{% endhighlight %}

to:

{% highlight python %}
# Import + Register Blueprints
from app.accounts import accounts as accounts
app.register_blueprint(accounts)
{% endhighlight %}

## Models

The models created should correspond, field-by-field, to the database tables that are setup as a result of running a series of migration commands. An example database setup using `PostgreSQL` is explained below:

With `PostgreSQL` setup with the database created and dependencies met you can now write a migration script. This migration script will be used to capture changes you make over time to the schemas of your various models, that will propagate changes to your database tables. In this class, rather than writing raw-SQL, we will utilize [`SQLAlchemy`](http://flask-sqlalchemy.pocoo.org/2.1/) (specifically, `Flask-SQLAlchemy`) as a database `Object-Relational-Model` (`ORM`, for short).  In addition, for the purposes of serialization (turning these database entities into organized [`JSONs`](http://www.json.org/) that we can send over the wire) and deserialization (turning a `JSON` into a entity once again), we will use [`Marshmallow`](https://marshmallow-sqlalchemy.readthedocs.io/en/latest/) (specifically, `marshmallow-SQLAlchemy`). These topics will be furthered covered next week. 

Here is an example migration script: 
{% highlight python %}
import os
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand
from app import app, db

migrate = Migrate(app, db)
manager = Manager(app)

manager.add_command("db", MigrateCommand)

if __name__ == "__main__":
  manager.run()
{% endhighlight %}

In the above, `app` refers to the module we created above. `db`refers to our reference to the database connection that we have yet to define in the `app` module.  

This script can be used in the following way to migrate your database, on changing your models:

{% highlight bash %}
# Initialize migrations
python manage.py db init
# Create a migration
python manage.py db migrate
# Apply it to the DB
python manage.py db upgrade
{% endhighlight %}

### Base Model

In `./models/__init__.py`, (accessible to the entire `models` module) we should write the following:

{% highlight python %}
from app import db # Grab the db from the top-level app
from marshmallow_sqlalchemy import ModelSchema # Needed for serialization in each model
from werkzeug import check_password_hash, generate_password_hash # Hashing
import hashlib # For session_token generation (session-based auth. flow)

class Base(db.Model):
  """Base PostgreSQL model"""
  __abstract__ = True
  id         = db.Column(db.Integer, primary_key =True)
  created_at = db.Column(db.DateTime, default    =db.func.current_timestamp())
  updated_at = db.Column(db.DateTime, default    =db.func.current_timestamp())

{% endhighlight %}

The above imports modules and objects necessary for use in your models, as well as defines a `Base` model class that every model should extend to inherit the book-keeping and necessary fields, `id`, `created_at`, and `updated_at`. These fields are critical for each model created.

### Example Models

Let's build an example `User` model. This `User` model in `./models/user.py` will consist of the following:

{% highlight python %}
from . import *

class User(Base):
  __tablename__ = 'users'

  email           = db.Column(db.String(128), nullable =False, unique =True)
  fname           = db.Column(db.String(128), nullable =False)
  lname           = db.Column(db.String(128), nullable =False)
  password_digest = db.Column(db.String(192), nullable =False)

  def __init__(self, ** kwargs):
    self.email           = kwargs.get('email', None)
    self.fname           = kwargs.get('fname', None)
    self.lname           = kwargs.get('lname', None)
    self.password_digest = generate_password_hash(kwargs.get('password'), None)

  def __repr__(self):
    return str(self.__dict__)


class UserSchema(ModelSchema):
  class Meta:
    model = User

{% endhighlight %}

The above is pretty self-explanatory.  It outlines several `db` fields, declares a constructor, defines a default string-based representation of the `User` model, and declares a `UserSchema` class that will be used to serialize and deserialize the `User` model to / from `JSON`. As you can see, this class inherits the `Base` class and as a result wil have the `id`, `created_at`, and `updated_at` fields by default.  

Let's build an example `Session` model. This `Session` model in `./models/session.py` will consist of the following:

{% highlight python %}
from . import *

class Session(Base):
  __tablename__ = 'sessions'

  user_id       = db.Column(db.Integer, db.ForeignKey('users.id'), unique =True, index =True)
  session_token = db.Column(db.String(40))
  update_token  = db.Column(db.String(40))
  expires_at    = db.Column(db.DateTime)

  def __init__(self, ** kwargs):
    user = kwargs.get('user', None)
    if user is None:
      raise Exception() # Shouldn't be the case

    self.user_id       = user.id
    self.session_token = self.urlsafe_base_64()
    self.update_token  = self.urlsafe_base_64()
    self.expires_at    = datetime.datetime.now() + datetime.timedelta(days=7)

  def __repr__(self):
    return str(self.__dict__)

  def urlsafe_base_64(self):
    return hashlib.sha1(os.urandom(64)).hexdigest()


class SessionSchema(ModelSchema):
  class Meta:
    model = Session

{% endhighlight %}

As we can see, the session model will belong to the user, and be part of a [token-based authentication flow](http://stackoverflow.com/questions/1592534/what-is-token-based-authentication).  Everything in the above code is self-explanatory, and serves as an example session implementation.  


# Controllers

We expose our models to the app via the Controller. As a start we must import our models into our app in order to have them be exposed to our `manage.py` script which is responsible for migrating our `db` to match our programmatic schema. As such, in `./controllers/__int__.py`, we would add the following (we'll throw in the imports while we're at it):

{% highlight python %}
from functools import wraps
from flask import request, jsonify, abort
import os

# Import module models
from app.accounts.models.user import *
from app.accounts.models.session import *

{% endhighlight %}

Now you can safely run the following in the root of your project to migrate your database:

{% highlight bash %}
# Initialize migrations
python manage.py db init
# Create a migration
python manage.py db migrate
# Apply it to the DB
python manage.py db upgrade
{% endhighlight %}

Now if you connect to your `Postgres` database, you should see two new tables, `users` and `sessions`!  The migration also creates an index on foreign-key `user_id` in `sessions`, for fast access of sessions by their owning user's `id`.  

The controller logic is more developed and will be CONTINUEDDDD..... 
