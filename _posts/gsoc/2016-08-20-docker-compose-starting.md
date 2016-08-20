---
layout: post
title: Getting started with Docker Compose
category: gsoc
tags: gsoc gsoc16 docker
---

In this post, I will talk about running multiple containers at once using [Docker Compose](https://github.com/docker/compose). 


#### The problem ?

Suppose you have a complex app with Database containers, Redis and what not. How are you going to start the app ?
One way is to write a shell script that starts the containers one by one. 

{% highlight bash %}
docker run postgres:latest --name mydb -d
docker run redis:3-alpine --name myredis -d
docker run myapp -d
{% endhighlight %}

Now suppose these containers have lots of configurations (links, volumes, ports, environment variables) that they need to function. You will have to write those parameters 
in the shell script.

{% highlight bash %}
docker network create myapp_default
docker run postgres:latest --name db -d -p 5432:5432 --net myapp_default
docker run redis:3-alpine --name redis -d -p 6379:6379 --net myapp_default \
	-v redis:/var/lib/redis/data
docker run myapp -d -p 5000:5000 --net myapp_default -e SOMEVAR=value --link db:db \
	--link redis:redis -v storage:/myapp/static
{% endhighlight %}

Won't it get un-manageable ? Won't it be great if we had a cleaner way to running multiple containers. Here comes docker-compose to the rescue. 


#### Docker compose

[Docker compose](https://docs.docker.com/compose/) is a python package which does the job of handling multiple containers for an application very elegantly. 
The main file of docker-compose is `docker-compose.yml` which is a YAML like syntax file with the settings/components required to run your app. 
Once you define that file, you can just do `docker-compose up` to start your app with all the components and settings. Pretty cool, right ?

So let's see the docker-compose.yml for the fictional app we have considered above.

{% highlight yaml %}
version: '2'

services:
  db:
    image: postgres:latest
    ports:
      - '5432:5432'

  redis:
    image: 'redis:3-alpine'
    command: redis-server
    volumes:
      - 'redis:/var/lib/redis/data'
    ports:
      - '6379:6379'

  web:
    build: .
    environment:
      SOMEVAR: value
    links:
      - db:db
      - redis:redis
    volumes:
      - 'storage:/myapp/static'
    ports:
      - '5000:5000'

volumes:
  redis:
  storage:
{% endhighlight %}

Once this file is in the project's root directory, you can use `docker-compose up` to start the application. 
It will run the services in the order in which they have been defined in the YAML file.

Docker compose has a lot of commands that generally correspond to the parameters that `docker run` accepts. 
You can see a full list on the official [docker-compose reference](https://docs.docker.com/compose/compose-file/). 


#### Conclusion 

It's no doubt that docker-compose is a boon when you have to run complex applications. It personally use Compose in every dockerized application that I write. 
In GSoC 16, I dockerized [Open Event](https://github.com/fossasia/open-event-orga-server). 
Here is the [docker-compose.yml](https://github.com/fossasia/open-event-orga-server/blob/development/docker-compose.yml) file if you are interested. 

PS - If you liked this post, you might find my [other posts on Docker](http://aviaryan.in/blog/tags.html#docker) interesting. Do take a look and let me know your views. 

