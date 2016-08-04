---
layout: post
title: Dynamically marshalling output in Flask Restplus
category: gsoc
tags: gsoc gsoc16 flask-restplus flask
---

Do you use [Flask-Restplus](https://github.com/noirbizarre/flask-restplus) ? Have you felt the need of dynamically modifying API output according to condition. 
If yes, then this post is for you. 

In this post, I will show how to use decorators to restrict GET API output. So let's start.

This is the basic code to create an API. Here we have created a `get_speaker` API to get a single item from Speaker model.

{% highlight python %}
from flask_restplus import Resource, Model, fields, Namespace
from models import Speaker

api = Namespace('speakers', description='Speakers', path='/')

SPEAKER = Model('Name', {
	'id': fields.Integer(),
	'name': fields.String(),
	'phone': fields.String()
})

class DAO:
	def get(speaker_id):
		return Speaker.query.get(speaker_id)

@api.route('/speakers/<int:speaker_id>')
class Speaker(Resource):
    @api.doc('get_speaker')
    @api.marshal_with(SPEAKER)
    def get(self, speaker_id):
        """Fetch a speaker given its id"""
        return DAO.get(speaker_id)
{% endhighlight %}

Now our need is to change the returned API data according to some condition. Like if user is authenticated then only return `phone` field of the SPEAKER model. 
One way to do this is to create condition statements in `get` method that marshals the output according to the situation. But if there are lots of methods which require this, 
then this is not a good way. 

So let's create a decorator which can change the `marshal` decorator at runtime. It will accept parameters as which models to marshal in case of authenticated and non-authenticated cases. 

{% highlight python %}
from flask_login import current_user
from flask_restplus import marshal_with

def selective_marshal_with(fields, fields_private):
    """
    Selective response marshalling. Doesn't update apidoc.
    """
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            if current_user.is_authenticated:
                model = fields
            else:
                model = fields_private
            func2 = marshal_with(model)(func)
            return func2(*args, **kwargs)
        return wrapper
    return decorator
{% endhighlight %}

The above code adds a wrapper over the API function which checks if the user is authenticated. If the user is authenticated, `fields` model is used for marshalling else 
`fields_private` is used for marshalling. 

So let's create the private model for `SPEAKER`. We will call it `SPEAKER_PRIVATE`. 

{% highlight python %}
from flask_restplus import Model, fields

SPEAKER_PRIVATE = Model('NamePrivate', {
	'id': fields.Integer(),
	'name': fields.String()
})
{% endhighlight %}

The final step is attaching the `selective_marshal_with` decorator to the get() method. 

{% highlight python %}
@api.route('/speakers/<int:speaker_id>')
class Speaker(Resource):
    @api.doc('get_speaker', model=SPEAKER)
    @selective_marshal_with(SPEAKER, SPEAKER_PRIVATE)
    def get(self, speaker_id):
        """Fetch a speaker given its id"""
        return DAO.get(speaker_id)
{% endhighlight %}

You will notice that I removed `@api.marshal_with(SPEAKER)`. This was to disable automatic marshalling of output by flask-restplus. To compensate for this, I have added 
`model=SPEAKER` in `api.doc`. It will not auto-marshal the output but will still show the swagger documentation. 

That concludes this. The get method will now switch `marshal` field w.r.t to the authentication level of the user.
As you may notice, the `selective_marhsal_with` function is generic and can be used with other models and APIs too. 
