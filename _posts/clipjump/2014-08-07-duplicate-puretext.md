---
layout: post
title: Duplicate PureText with Clipjump
category: clipjump
tags: clipjump
---

You can use the pre-distributed [NoFormatting Paste](https://github.com/aviaryan/Clipjump/blob/master/plugins/noformatting_paste.ahk) plugin to paste text 
without formatting. To set up the **Win+V** shortcut combination, you will have to use `ClipjumpCustom.ini`.  
Here is an example code -  

{% highlight ini %}
[paste_without_formatting]
bind = Win+v
run  = API.runPlugin(noformatting_paste.ahk)
{% endhighlight %}

Now pressing Win+V will paste the current clipboard trimming any formatting. If it doesn't work, make sure you have restarted Clipjump.    
  
#### Trimming the Whitespace
The updated version of [NoFormatting Paste](https://github.com/aviaryan/Clipjump/blob/master/plugins/noformatting_paste.ahk) allows trimming whitespace from the start 
and end of the string. It was released after [Clipjump v11.6 (07/08/14)](https://github.com/aviaryan/Clipjump/releases/tag/11.6). 
In case you don't have `NoFormatting Paste` version 0.2, download it from [GitHub](https://raw.githubusercontent.com/aviaryan/Clipjump/master/plugins/noformatting_paste.ahk).  
After downloading and setting it up, you will have to pass 1 as the first parameter of the plugin to trim all whitespaces. Example -

{% highlight ini %}
[paste_without_formatting]
bind = Win+v
run  = API.runPlugin(noformatting_paste.ahk, 1)
{% endhighlight %}
  
<br>
The same feature is available in Common Formats as `TrimWhiteSpaces` but that's a different thing. :-)