---
layout: post
title: Testing Docker Deployment using Travis
category: gsoc
tags: gsoc gsoc16 docker
---

Hello. This post is about how to setup automated tests to check if your application's docker deployment is working or not. I used it extensively while working on the Docker 
deployment of the Open Event Server. 
In this tutorial, we will use Travis CI as the testing service. 

To start testing your github project for Docker deployment, first add the repo to [Travis](https://travis-ci.org/). 
Then create a `.travis.yml` in the project's root directory. 

In that file, add docker to services. 

{% highlight yaml %}
services:
  - docker
{% endhighlight %}

The above will enable docker in the testing environment. It will also include `docker-compose` by default.

Next step is to build your app and run it. Since this is a pre-testing step, we will add it in the `install` directive.

{% highlight yaml %}
install:
  - docker build -t myapp .
  - docker run -d -p 127.0.0.1:80:4000 --name myapp myapp
{% endhighlight %}

The 4000 in the above text is assuming your app runs on port 4000 inside the container. Also it is assumed that the `Dockerfile` is in the root of the repo. 

So now that the docker app is running, it's time to test it. 

{% highlight yaml %}
script:
  - docker ps | grep -i myapp
{% endhighlight %}

The above will test if our app is in one of the running docker processes. It is a basic test to see if the app is running or not.  

We can go ahead and test the app's functionality with some sample requests. Create a file `test.py` with the following contents.

{% highlight python %}
import requests

r = requests.get('http://127.0.0.1/')
assert 'HomePage' in r.content, 'No homepage loaded'
{% endhighlight %}

Then run it as a test.

{% highlight yaml %}
script:
  - docker ps | grep -i myapp
  - python test.py
{% endhighlight %}

You can make use of the unittest module in Python to bundle and create more organized tests. The limit is the sky here. 

In the end, the `.travis.yml` will look something like the following 

{% highlight yaml %}
language: python
python:
  - "2.7"

install:
  - docker build -t myapp .
  - docker run -d -p 127.0.0.1:80:4000 --name myapp myapp

script:
  - docker ps | grep -i myapp
  - python test.py
{% endhighlight %}

So this is it. A basic tutorial on testing Docker deployments using the awesome Travis CI service. 

Feel free to share it and comment your views. 
