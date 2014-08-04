---
layout: page
title: Talk
tagline: Interscript communication provider
ghlink: https://github.com/aviaryan/autohotkey-scripts/blob/master/Functions/talk.ahk
highlight: 1
brushes: ['autohotkey']
hltheme: github
---

[Ahk Topic](http://www.autohotkey.com/board/topic/94321-)  
[Download Examples](https://dl.dropboxusercontent.com/u/116215806/Products/MoreAHK/talk_examples.7z)

`talk()` is a class to help share data and facilitate control between scripts as easy as it can get.  
Uses the SendMessage example from help file as a base to do the job.  
Works with compiled exe's too.  

<pre class="brush: ahk;">
client := new talk("script1.ahk")
</pre>