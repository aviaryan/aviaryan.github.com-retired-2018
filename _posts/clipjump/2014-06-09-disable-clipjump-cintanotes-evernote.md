---
layout: post
title: Disable Clipjump for a particular shortcut
category: clipjump
tags: clipjump
---

The plugin [CJ Disabled Shortcut](http://clipjump.sourceforge.net/downloads/plugins/cjdisabledShortcut.ahk) can be used to disable clipjump when pressing shortcuts like **Clip 
Note** in CintaNotes and similar features in Evernote, OneNote and other note-taking applications.  
Basically this plugin disables clipjump around the span of pressing a shortcut which alters clipboard and thus puts unneeded entries to Clipjump.  
  
To use this plugin, you will have to set a separate ClipjumpCustom binding for the shortcut you want to disable clipjump for. 
Here I will take example of [CintaNotes](http://cintanotes.com/) and its *Clip Text Hotkey* (Ctrl+F12) shortcut.  
After downloading the above plugin, put this code in ClipjumpCustom.ini .

{% highlight ini %}
[cintanotes_clip_text]
bind = Win + F12
run = API.runPlugin(cjdisabledShortcut.ahk, Ctrl+F12, 1200)
{% endhighlight %}

The `CJdisabledShortcut` plugin has two parameters.  

1. The `shortcut_key` - The key for which you want Clipjump blocked for. Here "Ctrl+F12"
2. The `delay` - The delay between pressing the "blocked shortcut" and re-enabling Clipjump. Here "1200"  
  
The *bind* key sets up "Win + F12" as the shortcut for this process.
Now you can press <kbd>Win</kbd> + <kbd>F12</kbd> to duplicate the feature of <kbd>Ctrl</kbd>+<kbd>F12</kbd> without invoking Clipjump.  
Same can be done for other applications.  
One good idea will be to change the CintaNotes <kbd>Ctrl</kbd>+<kbd>F12</kbd> shortcut to something *un-usable* like <kbd>Ctrl+Alt+Shift+F12</kbd> so that you
don't waste a *usable* shortcut space. This is just a suggestion, just what I have done here with the clipping shortcuts of CintaNotes and Evernote.  
  
#### Update
Well, you can do this with ClipjumpCustom alone, no need to use a plugin -   

{% highlight ini %}
[cn_clip_text]
bind = win + f12
run = API.blockMonitoring(1)
zsomevar = %HParse(Ctrl+F12, 1, 1, 1)%
send = %zsomevar%
sleep = 1200
run = API.blockMonitoring(0)
{% endhighlight %}

Please ask if you face problems.. 