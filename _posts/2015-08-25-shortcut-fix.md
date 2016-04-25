---
layout: post
title: Shortcut virus fix
tags: autohotkey
---

In recent days, a lot of my classmates had problems with their pendrives where a virus seem to have taken over and all of the drive contents where converted to shortcuts.
I saw the hidden system files and concluded that this was a work of a windows script (.WsF). So I decided to write up a script to fix these issues. As it seems to be a Windows only virus, I chose AutoHotkey as the programming language.

The virus worked in 2 steps.

- It marked all the files/folders in the root of the pendrive as hidden and system. So they automatically disappeared.
- Then it created shortcuts to all the items at the root. The `target` of these shortcuts was specified such that opening them executed a script which copied the virus to the system and also created an autorun entry for it. Now that system virus ran in the background and infected everything that got mounted.

So what my script does is that it iterates through all the files at the pendrive root, deletes the shortcut files and removes the system-hidden attributes from the original files. Additionally I also made it delete any .WsF and .vbs file that lied at the root.

The code can be found at my [autohotkey-scripts repo](https://github.com/aviaryan/autohotkey-scripts/blob/master/Tools/shortcut_fix.ahk) and the executable can be downloaded from my [dropbox](http://pastebin.com/raw/0a34it7y).


> One obvious limitation of my script is that it can't disinfect the system. So if you accidentally activated the virus in the pendrive, your system will get infected and will infect all future drives that plug into it. 

To fix system virus infection, I generally use *Task Manager* + *[Everything](http://www.voidtools.com/)* . I look in the Task Manager for the virus and then find it via Everything and delete it. 

For the shortcut virus, you can look in Task Manager for something like `wscript.exe` but that won't help as it is just the interpreter. Instead use something like [AutoRuns](https://technet.microsoft.com/en-in/sysinternals/bb963902.aspx) to see the startup entries and check for instances of some unknown weirdly named script ending in `.wsf` , `.vbs` or `.bat`. Then use Everything to find and delete it.

