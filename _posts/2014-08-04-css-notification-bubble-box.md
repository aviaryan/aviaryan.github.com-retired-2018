---
layout: post
title: Simple CSS notification boxes without using any icon
tags: css
---

This post will show you how to create message/notification boxes using CSS without using a image/icon/font icon.  
So for creating icons, we will use CSS `border-radius` property and some unicode text if needed. The four icons in question are <span class="symbol icon-info"></span> 
<span class="symbol icon-error"></span> <span class="symbol icon-tick"></span> and <span class="symbol icon-excl"></span>.  
Here is the style to create these 4 icons. You will notice that I have used specific fonts where needed.

{% highlight css %}
.symbol {
	font-size: 0.9em;
	font-family: Times New Roman;
	border-radius: 1em;
	padding: .1em .6em .1em .6em;
	font-weight: bolder;
	color: white;
	background-color: #4E5A56;
}

.icon-info { background-color: #3229CF; }
.icon-error { background: #e64943; font-family: Consolas; }
.icon-tick { background: #13c823; }
.icon-excl { background: #ffd54b; color: black; }

.icon-info:before { content: 'i'; }
.icon-error:before { content: 'x'; }
.icon-tick:before { content: '\002713'; }
.icon-excl:before { content: '!'; }
{% endhighlight %}

For creating containers i.e. message boxes, we will use the following CSS code -

{% highlight css %}
.notify {
	background-color:#e3f7fc; 
	color:#555; 
    border:.1em solid;
	border-color: #8ed9f6;
    border-radius:10px;
    font-family:Tahoma,Geneva,Arial,sans-serif;
    font-size:1.1em;
    padding:10px 10px 10px 10px;
    margin:10px;
    cursor: default;
}

.notify-yellow { background: #fff8c4; border-color: #f7deae; }
.notify-red { background: #ffecec; border-color: #fad9d7; }
.notify-green { background: #e9ffd9; border-color: #D1FAB6; }
{% endhighlight %}

Use the `.notify` class with `<div>` tag to create a *streched* container. Then use the `.symbol` class to create the icon and add the message text later. Here is the 
code for the following 4 boxes (in screenshot).

{% highlight html %}
<div class="notify"><span class="symbol icon-info"></span> A kind of a notice box !</div> 
<div class="notify notify-red"><span class="symbol icon-error"></span> Error message</div> 
<div class="notify notify-green"><span class="symbol icon-tick"></span> A positive/success/completion message</div> 
<div class="notify notify-yellow"><span class="symbol icon-excl"></span> A warning message</div>
{% endhighlight %}

<!-- image -->
<img src="http://i.imgur.com/WCoo9za.png">

<div class="notify"><span class="symbol icon-info"></span> To have the message box not strech to full width of the page, use span instead of div tag. </div> 
See the <a href="https://rawgit.com/aviaryan/4125787eaec46348268e/raw/7e2dfdc223f9d3adf08e97355c5756f3ad8c7692/css-box-noimage.html">working example</a> on Raw-Github ! And the <a href="https://gist.github.com/aviaryan/4125787eaec46348268e">gist's source</a>.  
Don't hesitate to ask if you face problems.
