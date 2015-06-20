---
layout: post
title: Set max clip limits for any channel in Clipjump
category: clipjump
highlight: 1
---

Clipjump by default only allows channel 0 (Default) to have limits on maximum number of clips that can be accomodated in it. In all channel other than 0, unlimited clips can be stored without resrictions. This post explains how one can impose a maximum clip limit on any channel.   
  
We will have to use [ClipjumpCustom.ini](http://clipjump.sourceforge.net/docs/custom.html) for the purpose. The variable we are going to use is `cn.totalclipsN` where the 
last ***N*** is the number of channel. For example, `cn.totalclips3` is for channel number 3.  
  
Now know that 'totalclips' is the maximum number of clips that can be contained in a channel. For the default channel 0, it is "Minimum number of active clipboards" + 
"Clipboard Threshold" i.e 20+10=30 in a default installation.   
If you set `cn.totalclips3 = 40` for the channel, it means that maximum clips that will finally exist in the channel (here 3) is 40 and the minimum number of active 
clipboards for that channel will be `totalclips - threshold` that is 40-10=30.  

<p class="message"> 
Note that you will have to set the value of <b>cn.totalclipsN</b> in a <a href="http://clipjump.sourceforge.net/docs/custom.html#not_autorun">auto-executing section</a> .
</p>
  
### Example
{% highlight ini %}
;Customizer File for Clipjump
;Add your custom settings here
    
[AutoRun]
; cn.totalclips1 = 30
cn.totalclips3 = 20
{% endhighlight %}
  
Note that this just a *hack* so it may have some shortcomes. For example, clips will be trimmed according to the limit applied only when a new clip is 
added in the channel because that is the time when compacting of database happens. 
  