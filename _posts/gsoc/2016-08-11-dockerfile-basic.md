---
layout: post
title: Writing your first Dockerfile
category: gsoc
tags: gsoc gsoc16 docker
---

In this tutorial, I will show you how to write your first `Dockerfile`. 
I got to learn Docker because I had to implement a [Docker](http://docker.com) deployment for our GSoC project [Open Event Server](https://github.com/aviaryan/open-event-orga-server).

First up, what is Docker ? 
Basically saying, Docker is an open platform for people to build, ship and run applications anytime and anywhere. Using Docker, your app will be able to run on any 
platform that supports Docker. And the best part is, it will run in the same way on different platforms i.e. no cross-platform issues. 
So you build your app for the platform you are most comfortable with and then deploy it anywhere.
This is the fundamental advantage of Docker and why it was created.

So let's start our dive into Docker. 

Docker works using Dockerfile ([example](https://github.com/fossasia/open-event-orga-server/blob/master/Dockerfile)), a file which specifies how Docker is supposed to build your application.
It contains the steps Docker is supposed to follow to package your app. Once that is done, you can send this packaged app to anyone and they can run it on their system with 
no problems.

Let's start with the project structure. You will have to keep `Dockerfile` at the root of your project. A basic project will look as follows -

{% highlight bash %}
- app.py
- Dockerfile
- requirements.txt
- some_app_folder/
-   some_file
-   some_file
{% endhighlight %}

Dockerfile starts with a base image that decides on which image your app should be built upon. Basically "Images" are nothing but apps. 
So for example you want your run your application in Ubuntu 14.04 VM, you use [ubuntu:14.04](https://hub.docker.com/_/ubuntu/) as the base image.

{% highlight bash %}
FROM ubuntu:14.04
MAINTAINER Your Name <your@email.com>
{% endhighlight %}

These are usually the first two lines of a Dockerfile and they specify the base image and Dockerfile maintainer respectively. 
You can look into [Docker Hub](https://hub.docker.com/) for more base images.

Now that we have started our Dockerfile, it's time to do something. Now think, if you are trying to run your app on a new system of Ubuntu, what will be the first step you 
will do... You update the package lists.

{% highlight bash %}
RUN apt-get update
{% endhighlight %}

You may possibly want to update the packages too. 

{% highlight bash %}
RUN apt-get update
RUN apt-get upgrade -y
{% endhighlight %}

Let's explain what's happening. `RUN` is a Docker command which instructs to run something on the shell. Here we are running `apt-get update` followed by `apt-get upgrade -y` 
on the shell. There is no need for `sudo` as Docker already runs commands with root user previledges. 

The next thing you will want to do now is to put your application inside the container (your Ubuntu VM). `COPY` command is just for that.

{% highlight bash %}
RUN mkdir -p /myapp
WORKDIR /myapp
COPY . .
{% endhighlight %}

Right now we were at the root of the ubuntu instance i.e. in parallel with /var, /home, /root etc. You surely don't want to copy your files there.
So we create a 'myapp' directory and set it as WORKDIR (project's directory). From now on, all commands will run inside it. 

Now that copying the app has been done, you may want to install it's requirements.

{% highlight bash %}
RUN apt-get install -y python python-setuptools python-pip
RUN pip install -r requirements.txt
{% endhighlight %}

You might be thinking why am I installing Python here. Isn't it present by default !? Well let me tell you that base image 'ubuntu' is not the Ubuntu you are used with. It just contains the bare essentials, not stuff like python, gcc, ruby etc. So you will have to install it on your own.

Similarly if you are installing some Python package that requires gcc, it will not work. When you are struck in a issue like that, try googling the error message and most 
likely you will find an answer. :grinning:

The last thing remaining now is to run your app. With this, your Dockerfile is complete. 

{% highlight bash %}
CMD python app.py
{% endhighlight %}


#### Building the app

To build the app run the following command. 

{% highlight bash %}
docker build -t myapp .
{% endhighlight %}

Then to run the app, execute `docker run myapp`.


#### Where to go next

Refer to the [official Dockerfile reference](https://docs.docker.com/engine/reference/builder/) to learn more Dockerfile commands. 
Also you may find my post on [using Travis to test Docker applications](http://aviaryan.in/blog/gsoc/docker-test.html) interesting if you want to automate testing of your Docker application.

I will write more blog posts on Docker as I learn more. I hope you found this one useful.

