---
layout: post
title: "Introducing Chattt - an open source CLI chat client"
tags: open-source project github
---

<center>
<img src="https://user-images.githubusercontent.com/4047597/39876787-b682c496-5491-11e8-97ba-a09e50fbc888.png">
</center>

So the past 2 months I was slowly working on my hobby project called "Chattt" and I just did the [v0.2](https://github.com/aviaryan/chattt/releases/tag/v0.2.0) release recently. It's quite ready for public use now so I thought I should share it with everyone. 

## So what is Chattt?

Chattt is an open source terminal based chat system. Its selling point is that it is very simple to use and it doesn't require much technical knowledge. I created it because I always wanted to have a simpler chat system that worked right from the terminal. I know you might say, "IRC does the same thing" but the thing is that I have always found IRC too complex. But that could be because I haven't given it much of a shot. 

Well whatever be the reason here, I went ahead and did this project. I even published it on `npm`. Here is how you can install and use it. 

```sh
$ npm install -g chattt
$ chattt
```
Once installed, it's ready to use. The video below shows a demo. 

<center>
<iframe width="672" height="378" src="https://www.youtube.com/embed/j-MDnWyzKpM?rel=0&amp;controls=0&amp;showinfo=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</center>


## The Development

Chattt was developed entirely in **JavaScript**(JS) which also happens to be my new favorite language. I had to create two GitHub repositories for it since it required both a [backend](https://github.com/aviaryan/chattt-backend) and a [frontend](https://github.com/aviaryan/chattt) part. 

The backend part of Chattt is an ExpressJS server using [socket.io](http://socket.io/) for interacting with chat clients i.e. the Chattt CLI instances. For those who don't know, `socket.io` is a convenient library for using web sockets in your projects. Chattt uses the same concept to implement its core i.e. the chatting feature. 

I also needed a place to host the backend. Since it was just a hobby project and I wasn't too sure about its success, I didn't want to spend money on a compute server. But then I learned about [Glitch](https://glitch.com/) which allows hosting of NodeJS projects for free. I gave it a try and after a few attempts, I was able to host [chattt's backend on it](https://glitch.com/edit/#!/chattt). 

Coming to the frontend part, I had to make a CLI for Chattt. This was a real challenge for me because a CLI chat application requires a very dynamic command-line interface i.e. the command line window should update-in-place. I didn't know how to do such a thing. But then I stumbled upon [blessed](https://github.com/chjj/blessed), a terminal interface library which allowed me to do just that. So excited, I started learning how to use it. Now this was quite challenging since blessed's documentation was massive and I didn't understand the keywords used there (since I hadn't done CLI programming before). But then I came upon [gitter-cli](https://github.com/RodrigoEspinosa/gitter-cli) which was an open-source CLI chat client for [Gitter](https://gitter.im/) and it also used the `blessed` library. So after going through its codebase, I was able to pick up `blessed`'s concepts better and this helped me to finally develop the chattt's user interface(UI). 

Connecting the UI with the backend to display chats was fairly easy once we had the UI so I won't be talking about that. But in the end, I think I did an impressive job on the UI, the result of which it looks like this now. 

<center>
<img src="https://user-images.githubusercontent.com/4047597/39875179-8489d65e-548d-11e8-990a-5a969aeba0ca.png">
</center>


## What's Next?

Glad you asked. Chattt is still in an early phase (only `v0.2`) and I plan to add some more features to it. The most notable ones are -

- Feature to set custom backend server in the client [v0.3]
- Feature to view active user list of a channel before actually joining it [v0.3]

I also plan to generalize the Chattt frontend to support other chat/communication providers like Telegram and Discord. Many users have suggested me this and I will give it a try after publishing `[v0.3]`. If you don't want to miss on these updates, I would suggest [watching](https://github.com/aviaryan/chattt/watchers) the GitHub repo. 


## Conclusion

Chattt is one of the [many](https://github.com/aviaryan?tab=repositories) "hobby" open source projects of mine. I learned a lot of new things developing it. I would really appreciate if you can share your suggestions and ideas regarding it with me on my Discord (**aviaryan#7504**) or GitHub. Seriously, they mean a lot. And thanks for reading this far.  Bye! 😊

👋🏻👋🏻👋🏻
