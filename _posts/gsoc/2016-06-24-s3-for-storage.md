---
layout: post
title: Using S3 for cloud storage
category: gsoc
tags: gsoc gsoc16 python
---

In this post, I will talk about how we can use the [Amazon S3](docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html) (Simple Storage Service) for cloud storage. 
As you may know, S3 is a no-fuss, super easy cloud storage service based on the IaaS model. 
There is no limit on the size of file or the amount of files you can keep on S3, you are only charged for the amount of bandwidth you use. 
This makes S3 very popular among enterprises of all sizes and individuals.

Now let's see how to use S3 in Python. Luckily we have a very nice library called [Boto](http://boto.cloudhackers.com/en/latest/) for it. 
Boto is a library developed by the AWS team to provide a Python SDK for the amazon web services. 
Using it is very simple and straight-forward. Here is a basic example of uploading a file on S3 using Boto - 

{% highlight python %}
import boto
from boto.s3.key import Key
# connect to the bucket
conn = boto.connect_s3(AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)
bucket = conn.get_bucket(BUCKET_NAME)
# set the key
key = 'key/for/file'
file = '/full/path/to/file'
# create a key to keep track of our file in the storage
k = Key(bucket)
k.key = key
k.set_contents_from_filename(file)
{% endhighlight %}

The above example uploads a `file` to s3 bucket `BUCKET_NAME`. 

Buckets are containers which store data. 
The `key` here is the unique key for an item in the bucket. Every item in the bucket is identified by a unique key assigned to it. 
The file can be downloaded from the url `BUCKET_NAME.s3.amazonaws.com/{key}`.
It is therefore essential to choose the key name smartly so that you don't end up overwriting an existing item on the server. 

In the [Open Event project](https://github.com/fossasia/open-event-orga-server/), I thought of a scheme that will allows us to avoid conflicts. It relies on using IDs of items for distinguishing them and goes as follows - 

* When uploading user avatar, key should be 'users/{userId}/avatar'
* When uploading event logo, key should be 'events/{eventId}/logo'
* When uploading audio of session, key should be 'events/{eventId}/sessions/{sessionId}/audio'

Note that to store user 'avatar', I am setting the key as `/avatar` and not `/avatar.extension`. This is because if user uploads pictures in different formats, we will end up 
storing different copies of avatars for the same user. This is nice but it's limitation is that downloading file from the url will give the file without an extension. 
So to solve this issue, we can use the Content-Disposition header.

{% highlight python %}
k.set_contents_from_filename(
	file,
	headers={
		'Content-Disposition': 'attachment; filename=filename.extension'
	}
)
{% endhighlight %}

So now when someone tries to download the file from that link, they will get the file with an extension instead of a no-extension "Choose what you want to do" file. 

This covers up the basics of using S3 for your Python project. You may explore [Boto's S3 documentation](boto.cloudhackers.com/en/latest/s3_tut.html) to find other interesting 
functions like deleting a folder, copy one folder to another and so. 

Also don't forget to have a look at the [awesome documentation](https://github.com/fossasia/open-event-orga-server/blob/master/docs/AMAZON_S3.md) 
we wrote for the Open Event project. 
It provides a more pictorial and detailed guide on how to setup S3 for your project.
