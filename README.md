# PoBE Site


## Setup

On initially cloning:

````bash
bundle
````

## Running Locally

````bash
jekyll serve
````

## Making Posts

The top of post's file must be the following:

````
YYYY-MM-DD-my-post-title.md
````

The top must have the following:

````
---
layout: post
---
````

Syntax highlighting is achieved through:

````
{% highlight LANGUAGE-NAME %}
....
{% endhighlight %}
````
