---
layout: post
title: Small Docker images using Alpine Linux
category: gsoc
tags: gsoc gsoc16 docker
---

Everyone likes optimization, small file sizes and such.. Won't it be great if you are able to reduce your Docker image sizes by a factor of 2 or more. 
Say hello to [Alpine Linux](https://hub.docker.com/_/alpine/).
It is a minimal Linux distro weighing just 5 MBs. It also has basic linux tools and a nice package manager APK. APK is quite stable and has a considerable amount of 
packages.

{% highlight bash %}
apk add python gcc
{% endhighlight %}

In this post, my main motto is how to squeeze the best out of AlpineLinux to create the smallest possible Docker image. So let's start.


### Step 1: Use AlpineLinux based images

Ok, I know that's obvious but just for the sake of completeness of this article, I will state that prefer using Alpine based images wherever possible. 
[Python](https://hub.docker.com/_/python/) and Redis have their official Alpine based images whereas NodeJS has good unoffical Alpine-based images. 
Same goes for Postgres, Ruby and other popular environments.


### Step 2: Install only needed dependencies

Prefer installing select dependencies over installing a package that contains lots of them. For example, prefer installing `gcc` and development libraries over buildpacks.
You can find listing of Alpine packages on their [website](https://pkgs.alpinelinux.org/packages).

**Pro Tip** - A great list of Debian v/s Alpine development packages is at [alpine-buildpack-deps](https://hub.docker.com/r/praekeltfoundation/alpine-buildpack-deps/) Docker Hub page (scroll down to Packages). It is a very complete list and you will always find the dependency you are looking for.  


### Step 3: Delete build dependencies after use

Build dependencies are required by components/libraries to build native extensions for the platform. Once the build is done, they are not needed.
So you should delete the build-dependencies after their job is complete. Have a look at the following snippet.

{% highlight bash %}
RUN apk add --virtual build-dependencies gcc python-dev linux-headers musl-dev postgresql-dev \
    && pip install -r requirements.txt \
    && apk del build-dependencies
{% endhighlight %}

I am using `--virtual` to give a label to the pacakages installed on that instance and then when `pip install` is done, I am deleting them.


### Step 4: Remove cache

Cache can take up lots of un-needed space. So always run `apk add` with `--no-cache` parameter.

{% highlight bash %}
RUN apk add --no-cache package1 package2
{% endhighlight %}

If you are using npm for manaing project dependencies and bower for managing frontend dependencies, it is recommended to clear their cache too.

{% highlight bash %}
RUN npm cache clean && bower cache clean
{% endhighlight %}


### Step 5: Learn from the experts

Each and every image on Docker Hub is open source, meaning that it's Dockerfile is freely available. Since the official images are made as efficient as possible, 
it's easy to find great tricks on how to achieve optimum performance and compact size in them. So when viewing an image on DockerHub, don't forget to peek into its 
Dockerfile, it helps more than you can imagine. 


## Conclusion

That's all I have for now. I will keep you updated on new tips if I find any. In my personal experience, I found AlpineLinux to be worth using. 
I tried deploying [Open Event Server](https://github.com/fossasia/open-event-orga-server) on Alpine but faced some issues so ended up creating a Dockerfile 
using [debain:jessie](https://hub.docker.com/_/debian/). 
But for small projects, I would recommend Alpine.
On large and complex projects however, you may face issues with Alpine at times. That maybe due to lack of packages, lack of library support or some other thing.
But it's not impossible to overcome those issues so if you try hard enough, you can get your app running on Alpine.
