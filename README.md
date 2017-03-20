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

The file must be in the `_posts` directory and its name must be the following:

````
YYYY-MM-DD-my-post-title.md
````

The top of the file must have the following:

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
