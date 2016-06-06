---
layout: post
title: REST API Authentication in Flask
category: gsoc
tags: gsoc gsoc16 flask python
---

Recently I had the challenge of restricting unauthorized personnel from accessing some views in Flask. 
Sure the naive way will be asking the username and password in the json itself and checking the records in the database. The request will be something like this-

{% highlight json %}
{
	"username": "open_event_user",
	"password": "password"
}
{% endhighlight %}

But I wanted to do something better. So I looked up around the Internet and found that it is possible to accept Basic authorization credentials in Flask (sadly it isn't documented). 
For those who don't know what Basic authorization is a way to send plain `username:password` combo as header in a request after obscuring them with base64 encoding. 
So for the above username and password, the corresponding header will be - 

{% highlight json %}
{
	"Authorization": "Basic b3Blbl9ldmVudF91c2VyOnBhc3N3b3Jk"
}
{% endhighlight %}
where the hashed string is base64 encoded form of string "open\_event\_user:password".

Now back to the topic, so the next job is to validate the views by checking the Basic auth credentials in header and call `abort()` if credentials are missing or wrong. 
For this, we can easily create a helper function that aborts a view if there is something wrong with the credentials. 

{% highlight python %}
from flask import request, Flask, abort
from models import UserModel

app = Flask(__name__)

def validate_auth():
	auth = request.authorization
	if not auth:  # no header set
		abort(401)
	user = UserModel.query.filter_by(username=auth.username).first()
	if user is None or user.password != auth.password:
		abort(401)

@app.route('/view')
def my_view():
	validate_auth()
	# stuff on success
	# more stuff
{% endhighlight %}

This works but wouldn't it be nice if we could specify `validate_auth` function as a decorator. 
This will give us the advantage of only having to set it once in a model view with all auth-required methods. Right ? So here we go 

{% highlight python %}
def requires_auth(f):
	@wraps(f)
	def decorated(*args, **kwargs):
		auth = request.authorization
		if not auth:  # no header set
			abort(401)
		user = UserModel.query.filter_by(username=auth.username).first()
		if user is None or user.password != auth.password:
			abort(401)
		return f(*args, **kwargs)
	return decorated

@app.route('/view')
@requires_auth
def my_view():
	# stuff on success
	# more stuff
{% endhighlight %}
I renamed the function from validate\_auth to requires\_auth because it suits the context better. 

At this point, the above code may look perfect but it doesn't work when you are accessing the API through Swagger web UI. 
This is because it is not possible to set base64 encoded authorization header from the swagger UI.
For those who are wondering "what the hell is swagger", I will define Swagger as a tool for API based projects which creates a nice web UI to live-test the API and 
also exports a schema of the API that can be used to understand API definitions.

Now how do we get `requires_auth` to work when a request is sent through swagger UI ? It was a little tricky and took me a couple of hours but I finally got it. 
The trick therefore is to check for active sessions when there are no authorization headers set (as in the case of swagger UI). 
If an active session is found, it means that the user is authenticated. 
Here I would like to suggest using Flask-Login extension which makes session and login management a child's play.
Always use it if your flask project deals with login, user accounts and stuff. 

Now back to the task in hand, here is how we can set the `requires_auth` function to check for existing sessions. 

{% highlight python %}
from flask import request, abort, g
from flask.ext import login

def requires_auth(f):
	@wraps(f)
	def decorated(*args, **kwargs):
		auth = request.authorization
		if not auth:  # no header set
			if login.current_user.is_authenticated:  # check active session
				g.user = login.current_user
				return f(*args, **kwargs)
			else:
				abort(401)
		user = UserModel.query.filter_by(username=auth.username).first()
		if user is None or user.password != auth.password:
			abort(401)
		g.user = user
		return f(*args, **kwargs)
	return decorated
{% endhighlight %}

Pretty easy right !! Also notice that I am saving the user who was currently authenticated in flask's global variable `g`. 
Now the authenticated user can be accessed from views as `g.user`. Cool, isn't it ?
Now if there is a need to add a more secure form of authorization like 'Token' based, you can easily update the `requires_auth` decorator to get the same results. 

I hope this article provided valuable insight into managing REST API authorizations in Flask. I will keep posting more awesome things I learn in my GSoC journey. 

That's it. Sayonara.
