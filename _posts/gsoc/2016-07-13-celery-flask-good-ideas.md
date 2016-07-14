---
layout: post
title: Ideas on using Celery in Flask for background tasks
category: gsoc
tags: gsoc gsoc16 celery flask
---

Simply put, Celery is a background task runner. It can run time-intensive tasks in the background so that your application can focus on the stuff that matters the most.
In context of a Flask application, the stuff that matters the most is listening to HTTP requests and returning response. 

By default, Flask runs on a single-thread. 
Now if a request is executed that takes several seconds to run, then it will block all other incoming requests as it is single-threaded.
This will be a very bad-experience for the user who is using the product. So here we can use Celery to move time-hogging part of that request to the background.

I would like to let you know that by "background", Celery means another process. 
Celery starts worker processes for the running application and these workers receive work from the main application.
Celery requires a broker to be used. Broker is nothing but a database that stores results of a celery task and provides a shared interface between main process and worker processes. 
The output of the work done by the workers is stored in the Broker. The main application can then access these results from the Broker.

Using Celery to set background tasks in your application is as simple as follows -

{% highlight python %}
@celery.task
def background_task(*args, **kwargs):
    # do stuff
    # more stuff
{% endhighlight %}

Now the function `background_task` becomes function-able as a background task. To execute it as a background task, run - 

{% highlight python %}
task = background_task.delay(*args, **kwargs)
print task.state  # task current state (PENDING, SUCCESS, FAILURE)
{% endhighlight %}

Till now this may look nice and easy but it can cause lots of problems. This is because the background tasks run in different processes than the main application.
So the state of the *worker application* differs from the *real application*. 

One common problem because of this is the lack of request context. Since a celery task runs in a different process, so the request context is not available. 
Therefore the request headers, cookies and everything else is not available when the task actually runs. 
I too faced this problem and solved it using an excellent snippet I found on the Internet.

<!-- gist -->
<!-- <script src="https://gist.github.com/aviaryan/8620390d832678f0de4528bbc4b4b272.js"></script> -->
{% highlight python %}
"""
Celery task wrapper to set request context vars and global
vars when a task is executed
Based on http://xion.io/post/code/celery-include-flask-request-context.html
"""
from celery import Task
from flask import has_request_context, make_response, request, g

from app import app  # the flask app


__all__ = ['RequestContextTask']


class RequestContextTask(Task):
    """Base class for tasks that originate from Flask request handlers
    and carry over most of the request context data.
    This has an advantage of being able to access all the usual information
    that the HTTP request has and use them within the task. Pontential
    use cases include e.g. formatting URLs for external use in emails sent
    by tasks.
    """
    abstract = True

    #: Name of the additional parameter passed to tasks
    #: that contains information about the original Flask request context.
    CONTEXT_ARG_NAME = '_flask_request_context'
    GLOBALS_ARG_NAME = '_flask_global_proxy'
    GLOBAL_KEYS = ['user']

    def __call__(self, *args, **kwargs):
        """Execute task code with given arguments."""
        call = lambda: super(RequestContextTask, self).__call__(*args, **kwargs)

        # set context
        context = kwargs.pop(self.CONTEXT_ARG_NAME, None)
        gl = kwargs.pop(self.GLOBALS_ARG_NAME, {})

        if context is None or has_request_context():
            return call()

        with app.test_request_context(**context):
            # set globals
            for i in gl:
                setattr(g, i, gl[i])
            # call
            result = call()
            # process a fake "Response" so that
            # ``@after_request`` hooks are executed
            # app.process_response(make_response(result or ''))

        return result

    def apply_async(self, args=None, kwargs=None, **rest):
        self._include_request_context(kwargs)
        self._include_global(kwargs)
        return super(RequestContextTask, self).apply_async(args, kwargs, **rest)

    def apply(self, args=None, kwargs=None, **rest):
        self._include_request_context(kwargs)
        self._include_global(kwargs)
        return super(RequestContextTask, self).apply(args, kwargs, **rest)

    def retry(self, args=None, kwargs=None, **rest):
        self._include_request_context(kwargs)
        self._include_global(kwargs)
        return super(RequestContextTask, self).retry(args, kwargs, **rest)

    def _include_request_context(self, kwargs):
        """Includes all the information about current Flask request context
        as an additional argument to the task.
        """
        if not has_request_context():
            return

        # keys correspond to arguments of :meth:`Flask.test_request_context`
        context = {
            'path': request.path,
            'base_url': request.url_root,
            'method': request.method,
            'headers': dict(request.headers),
        }
        if '?' in request.url:
            context['query_string'] = request.url[(request.url.find('?') + 1):]

        kwargs[self.CONTEXT_ARG_NAME] = context

    def _include_global(self, kwargs):
        d = {}
        for z in self.GLOBAL_KEYS:
            if hasattr(g, z):
                d[z] = getattr(g, z)
        kwargs[self.GLOBALS_ARG_NAME] = d
{% endhighlight %}

To run a task in Request context mode, do - 

{% highlight python %}
@celery.task(base=RequestContextTask, bind=True)
def background_task(self, *args, **kwargs):
    # do stuff
    # more stuff
{% endhighlight %}

If you are wondering what the RequestContextTask class does, it simply stores all request context vars and global vars when a background task is called (`task.delay()`) 
and then unpacks those values to their proper places when the task is about to be run. The above snippet can be easily extended to store any value. 

Another challenge that some people may face is the occasional Parsing/Serialization error. 
This happens because the data being sent to/from a function that is to be background executed is too complex. 

Serialization is the process of converting complex data structures and objects into a plain string.
Serialization of data is necessary because the background tasks and the main thread run in different processes.
Now think how will the main thread communicate the celery thread to do some task. 
This is done using serialization of the concerned data. 
So to avoid serialization errors, it is recommended that you make background tasks such that they require only simple arguments to run and they return only simple data.

So basically keeping small and simple tasks is recommended when using Celery. Follow this golden rule and you will not run into any problems.
