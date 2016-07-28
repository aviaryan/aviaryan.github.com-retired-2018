---
layout: post
title: Downloading Files from URLs in Python
category: gsoc
tags: gsoc gsoc16 python
---

This post is about how to efficiently/correctly download files from URLs using Python. 
I will be using the god-send library [requests](docs.python-requests.org/) for it. I will write about methods to correctly download binaries from URLs and set their filenames. 

Let's start with baby steps on how to download a file using requests -- 

{% highlight python %}
import requests

url = 'http://google.com/favicon.ico'
r = requests.get(url, allow_redirects=True)
open('google.ico', 'wb').write(r.content)
{% endhighlight %}

The above code will download the media at [http://google.com/favicon.ico](http://google.com/favicon.ico) and save it as google.ico.

Now let's take another example where url is [https://www.youtube.com/watch?v=9bZkp7q19f0](https://www.youtube.com/watch?v=9bZkp7q19f0). 
What do you think will happen if the above code is used to download it ?
If you said that a HTML page will be downloaded, you are spot on. This was one of the problems I faced in the Import module of Open Event where I had to download media from 
certain links. When the URL linked to a webpage rather than a binary, I had to not download that file and just keep the link as is. 
To solve this, what I did was inspecting the headers of the URL. Headers usually contain a `Content-Type` parameter which tells us about the type of data the url is linking to.
A naive way to do it will be - 

{% highlight python %}
r = requests.get(url, allow_redirects=True)
print r.headers.get('content-type')
{% endhighlight %}

It works but is not the optimum way to do so as it involves downloading the file for checking the header. 
So if the file is large, this will do nothing but waste bandwidth. 
I looked into the requests documentation and found a better way to do it. That way involved just fetching the headers of a url before actually downloading it. 
This allows us to skip downloading files which weren't meant to be downloaded.

{% highlight python %}
import requests

def is_downloadable(url):
    """
    Does the url contain a downloadable resource
    """
    h = requests.head(url, allow_redirects=True)
    header = h.headers
    content_type = header.get('content-type')
    if 'text' in content_type.lower():
        return False
    if 'html' in content_type.lower():
        return False
    return True

print is_downloadable('https://www.youtube.com/watch?v=9bZkp7q19f0')
# >> False
print is_downloadable('http://google.com/favicon.ico')
# >> True
{% endhighlight %}

To restrict download by file size, we can get the filesize from the `Content-Length` header and then do suitable comparisons.

{% highlight python %}
content_length = header.get('content-length', None)
if content_length and content_length > 2e8:  # 200 mb approx
	return False
{% endhighlight %}

So using the above function, we can skip downloading urls which don't link to media.


#### Getting filename from URL

We can parse the url to get the filename. 
Example - [http://aviaryan.in/images/profile.png](http://aviaryan.in/images/profile.png). 

To extract the filename from the above URL we can write a routine which fetches the last string after backslash (/).

{% highlight python %}
url = 'http://aviaryan.in/images/profile.png'
if url.find('/'):
	print url.rsplit('/', 1)[1]
{% endhighlight %}

This will be give the filename in some cases correctly. However, there are times when the filename information is not present in the url. 
Example, something like `http://url.com/download`. In that case, the `Content-Disposition` header will contain the filename information. 
Here is how to fetch it.

{% highlight python %}
import requests
import re

def get_filename_from_cd(cd):
    """
    Get filename from content-disposition
    """
    if not cd:
        return None
    fname = re.findall('filename=(.+)', cd)
    if len(fname) == 0:
        return None
    return fname[0]


url = 'http://google.com/favicon.ico'
r = requests.get(url, allow_redirects=True)
filename = get_filename_from_cd(r.headers.get('content-disposition'))
open(filename, 'wb').write(r.content)
{% endhighlight %}

The url-parsing code in conjuction with the above method to get filename from `Content-Disposition` header will work for most of the cases.
Use them and test the results. 

These are my 2 cents on downloading files using requests in Python. Let me know of other tricks I might have overlooked. 
