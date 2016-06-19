---
layout: post
title: Paginated APIs in Flask
category: gsoc
tags: gsoc gsoc16 flask python
---

Week 2 of GSoC I had the task of implementing paginated APIs in [Open Event](https://github.com/fossasia/open-event) project. 
I was aware that [DRF](www.django-rest-framework.org/) provided such feature in Django so I looked through the Internet to find some library for Flask.
Luckily, I didn't find any so I decided to make my own. 

A paginated API is page-based API. This approach is used as the API data can be very large sometimes and pagination can help to break it into small chunks. 
The Paginated API built in the Open Event project looks like this - 

{% highlight json %}
{
    "start": 41,
    "limit": 20,
    "count": 128,
    "next": "/api/v2/events/page?start=61&limit=20",
    "previous": "/api/v2/events/page?start=21&limit=20",
    "results": [
    	{
    		"data": "data"
    	},
    	{
    		"data": "data"
    	}
    ]
}
{% endhighlight %}

Let me explain what the keys in this JSON mean - 

1. `start` - It is the position from which we want the data to be returned.
2. `limit` - It is the max number of items to return from that position.
3. `next` - It is the url for the next page of the query assuming current value of `limit`
4. `previous` - It is the url for the previous page of the query assuming current value of `limit`
5. `count` - It is the total count of results available in the dataset. Here as the 'count' is 128, that means you can go maximum till start=121 keeping limit as 20. Also when 
 you get the page with start=121 and limit=20, 8 items will be returned. 
6. `results` - This is the list of results whose position lies within the bounds specified by the request.

Now let's see how to implement it. I have simplified the code to make it easier to understand.

{% highlight python %}
from flask import Flask, abort, request, jsonify
from models import Event

app = Flask(__name__)

@app.route('/api/v2/events/page')
def view():
	return jsonify(get_paginated_list(
		Event, 
		'/api/v2/events/page', 
		start=request.args.get('start', 1), 
		limit=request.args.get('limit', 20)
	))

def get_paginated_list(klass, url, start, limit):
    # check if page exists
    results = klass.query.all()
    count = len(results)
    if (count < start):
        abort(404)
    # make response
    obj = {}
    obj['start'] = start
    obj['limit'] = limit
    obj['count'] = count
    # make URLs
    # make previous url
    if start == 1:
        obj['previous'] = ''
    else:
        start_copy = max(1, start - limit)
        limit_copy = start - 1
        obj['previous'] = url + '?start=%d&limit=%d' % (start_copy, limit_copy)
    # make next url
    if start + limit > count:
        obj['next'] = ''
    else:
        start_copy = start + limit
        obj['next'] = url + '?start=%d&limit=%d' % (start_copy, limit)
    # finally extract result according to bounds
    obj['results'] = results[(start - 1):(start - 1 + limit)]
    return obj
{% endhighlight %}

Just to be clear, here I am assuming you are using SQLAlchemy for the database. The `klass` parameter in the above code is the SqlAlchemy db.Model class on which you want 
to query upon for the results. The `url` is the base url of the request, here '/api/v2/events/page' and it used in setting the *previous* and *next* urls. 
Other things should be clear from the code. 

So this was how to implement your very own Paginated API framework in Flask (should say Python). I hope you found this post interesting. 

Until next time.

Ciao
