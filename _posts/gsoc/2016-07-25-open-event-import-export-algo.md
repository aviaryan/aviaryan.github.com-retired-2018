---
layout: post
title: Import/Export feature of Open Event - Challenges
category: gsoc
tags: gsoc gsoc16
---

We have developed a nice import/export feature as a part of our GSoC project [Open Event](https://github.com/fossasia/open-event).
It allows user to export an event and then further import it back.

Event contains data like tracks, sessions, microlocations etc.
When I was developing the basic part of this feature, it was a challenge on how to export and then further import the same data.
I was in need of a format that completely stores data and is recognized by the current system.
This is when I decided to use the APIs.

API documentation of Open Event project is at [http://open-event.herokuapp.com/api/v2](http://open-event.herokuapp.com/api/v2). We have a considerably rich API covering most 
aspects of the system.
For the export, I adopted this very simple technique.

1. Call the corresponding GET APIs (tracks, sessions etc) for a database model internally.
2. Save the data in separate json files.
3. Zip them all and done.

This was very simple and convenient. Now the real challenge came of importing the event from the data exported.
As exported data was nothing but json, we could have created the event back by sending the data back as POST request.
But this was not that easy because the data formats are not exactly the same for GET and POST requests.

Example - 

Sessions GET -- 

{% highlight json %}
{
	"speakers": [
		{
			"id": 1,
			"name": "Jay Sean"
		}
	],
	"track": {
		"id": 1,
		"name": "Warmups"
	}
}
{% endhighlight %}

Sessions POST --

{% highlight json %}
{
	"speaker_ids": [1],
	"track_id": 1
}
{% endhighlight %}

So the exported data can only be imported when it has been converted to POST form. Luckily, the only change between POST and GET APIs was of the related attributes where 
dictionary in GET was replaced with just the ID in POST/PUT. 
So when importing I had to make it so such that the dicts are converted to their POST counterparts. For this, all that I had to do was to list all dict-type keys 
and extract the `id` key from them.
I defined a global variable as the following listing all dict keys and then wrote a function to extract the ids and convert the keys.

{% highlight python %}
RELATED_FIELDS = {
    'sessions': [
        ('track', 'track_id', 'tracks'),
        ('speakers', 'speaker_ids', 'speakers'),
    ]
}
{% endhighlight %}


#### Second challenge

Now I realized that there was even a tougher problem, and that was how to re-create the relations. 
In the above json, you must have realized that a session can be related to speaker(s) and track. These relations are managed using the IDs of the items. 
When an event is imported, the IDs are bound to change and so the old IDs will become outdated i.e. a track which was at ID 62 when exported can be at ID 92 when it is imported.
This will cause the relationships to break.
So to counter this problem, I did the following - 

1. Import items in a specific order, independent first
2. Store a map of old IDs v/s new IDs. 
3. When dependent items are to be created, get new ID from the map and relate with it.

Let me explain the above -

The first step was to import/re-create the independent items first. Here independent items are tracks and speakers, and the dependent item is session. 
Now while creating the independent items, store their new IDs after create. Create a map of old ids v/s new ids and store it.
This map will hold a clue to *what became what* after they were recreated from the json.
Now the key final step is that when dependent items are to be created, find the indepedent *related* keys in their json using the above defined *RELATED_FIELDS* listing.
Once they are found, extract their IDs and find the new ID corresponding to their old ID. 
Link the new ID with the dependent item and that would be all.

This post covers the main challenges I faced when developing the import/export feature and how I overcame them. 
I hope it will provide some help when you are dealing with similar problems. 

