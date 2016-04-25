---
layout: post
title: Emojis offline
tags: python github
---

I am often in need of some [emojis](http://emoji.muan.co/) when committing on my github repos. As I don't have an *always on* Internet connection, I had to sacrifice them when I was not online.
I tried saving the [http://emoji.muan.co/](http://emoji.muan.co/) page but that didn't work. So I wrote a simple Python script to set up a server from the emoji's directory and run the browser with the emoji selector running on localhost.

![Emoji Offline](http://i.imgur.com/ijem3LQ.png)

You can find the script at [https://github.com/aviaryan/pythons/tree/master/EmojiServer](https://github.com/aviaryan/pythons/tree/master/EmojiServer). The instructions in [README](https://github.com/aviaryan/pythons/blob/master/EmojiServer/README.md) should be sufficient to set it up and start using it.