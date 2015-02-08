---
layout: post
title: Creating Win Shortcut in Clipjump
category: clipjump
tags: clipjump
---

You may have noticed that creating shortcut having the **Win** key is not possible with the Settings editor.
If you are really keen on utilizing those un-used Win keys , you can use the [ClipjumpCustom.ini](http://clipjump.sourceforge.net/docs/custom.html) feature. 

All you have to do is [get the correspoding label](http://clipjump.sourceforge.net/docs/devList.html#labels) for the feature you need and then follow the underlying example.

```
[win_K]
bind = Win + K
run = channelOrganizer
```

This uses the label `channelOrganizer` to create the shortcut `Win + k` for the 'Channel Organizer'.
Add the above snippet to **ClipjumpCustom.ini** and restart to use `Win + k` as a shortcut for 'Channel Organizer'.

Here is another one for 'Select Channel' window.

```
[some_name]
bind = Win + Shift + C
run = channelGUI
```
