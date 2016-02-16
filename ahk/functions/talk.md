---
layout: page
title: Talk
tagline: Interscript communication provider
ghlink: https://github.com/aviaryan/autohotkey-scripts/blob/master/Functions/talk.ahk
---

[Ahk Topic](http://www.autohotkey.com/board/topic/94321-)  
[Download Examples](http://bit.ly/215YECi)

`talk()` is a class to help share data and facilitate control between scripts as easy as it can get.  
Uses the SendMessage example from help file as a base to do the job.  
Works with compiled exe's too.  

{% highlight autohotkey %}
client := new talk("script1.ahk")
{% endhighlight %}