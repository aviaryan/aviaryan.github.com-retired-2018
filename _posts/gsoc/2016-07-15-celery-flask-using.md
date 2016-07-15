---
layout: post
title: Setting up Celery with Flask
category: gsoc
tags: gsoc gsoc16 celery flask
---

In this article, I will explain how to use Celery with a Flask application.
Celery requires a broker to run. The most famous of the brokers is Redis.
So to start using Celery with Flask, first we will have to setup the Redis broker.

Redis can be downloaded from their site [http://redis.io](http://redis.io).
I wrote a script that simplifies downloading, building and running the redis server.

{% highlight bash %}
#!/bin/bash
# This script downloads and runs redis-server.
# If redis has been already downloaded, it just runs it
if [ ! -d redis-3.2.1/src ]; then
    wget http://download.redis.io/releases/redis-3.2.1.tar.gz
    tar xzf redis-3.2.1.tar.gz
    rm redis-3.2.1.tar.gz
    cd redis-3.2.1
    make
else
    cd redis-3.2.1
fi
src/redis-server
{% endhighlight %}

When the above script is ran from the first time, the redis folder doesn\'t exist so it downloads the same, builds it and then runs it. 
In subsequent runs, it will skip the downloading and building part and just run the server.

Now that the redis server is running, we will have to install its Python counterpart.

{% highlight bash %}
pip install redis
{% endhighlight %}

After the redis broker is set, now its time to setup the celery extension. 
First install celery by using `pip install celery`.
Then we need to setup celery in the flask app definition.

{% highlight python %}
# in app.py
def make_celery(app):
	# set redis url vars
	app.config['CELERY_BROKER_URL'] = environ.get('REDIS_URL', 'redis://localhost:6379/0')
    app.config['CELERY_RESULT_BACKEND'] = app.config['CELERY_BROKER_URL']
    # create context tasks in celery
    celery = Celery(app.import_name, broker=app.config['CELERY_BROKER_URL'])
    celery.conf.update(app.config)
    TaskBase = celery.Task
    class ContextTask(TaskBase):
        abstract = True
        def __call__(self, *args, **kwargs):
            with app.app_context():
                return TaskBase.__call__(self, *args, **kwargs)
    celery.Task = ContextTask
    return celery

celery = make_celery(current_app)
{% endhighlight %}

Now that Celery is setup on our project, let's define a sample task.

{% highlight python %}
@app.route('/task')
def view():
	background_task.delay(*args, **kwargs)
	return 'OK'

@celery.task
def background_task(*args, **kwargs):
	# code
	# more code
{% endhighlight %}

Now to run the celery workers, execute

{% highlight bash %}
celery worker -A app.celery
{% endhighlight %}

That should be all. Now to run our little project, we can execute the following script.

{% highlight bash %}
bash run_redis.sh &  # to run redis
celery worker -A app.celery &  # to run celery workers
python app.py
{% endhighlight %}

If you are wondering how to run the same on Heroku, just use the free [heroku-redis](https://elements.heroku.com/addons/heroku-redis) extension. 
It will start the redis server on heroku. Then to run the workers and app, set the Procfile as - 

{% highlight yaml %}
web: sh heroku.sh
{% endhighlight %}

Then set the heroku.sh as - 

{% highlight bash %}
#!/bin/bash
celery worker -A app.celery &
gunicorn app:app
{% endhighlight %}


That's a basic guide on how to run a Flask app with Celery and Redis.
If you want more information on this topic, please see my post 
[Ideas on Using Celery in Flask for background tasks](http://aviaryan.in/blog/gsoc/celery-flask-good-ideas.html). 
