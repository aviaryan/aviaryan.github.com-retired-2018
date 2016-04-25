---
layout: post
title: C# Sublime Build
tags: sublime-text
---

If you want to code in C# using Sublime Text, then this post is for you. After this post you will be able to use `Ctrl+B` to build a .cs file and `Ctrl+Shift+B` to run the exe through 
the terminal.  

## The Steps

1. Find the C-Sharp Compiler on your system. Normally it's in `C:\Windows\Microsoft.NET\Framework\` folder. For me it is `C:\Windows\Microsoft.NET\Framework\v3.5\csc.exe`

2. Add csc.exe's path to Environment Variables. The name of the variable should be `csc.exe` . The screenshot should assist you. ![csc.exe-setting](http://i.imgur.com/wEhvkFn.png)

3. Then add the csc.exe's directory to path variable. For me the directory is `C:\Windows\Microsoft.NET\Framework\v3.5`. Just append the directory in `PATH` with a preceding semi-colon (;).

4. The create this build file for Sublime Text. Name it something like `C#.sublime-build` and store it in Data\Packages\User directory.

{% highlight json %}
{
  "selector"  : "source.cs",
  "cmd"       : "gmcs $file_name",
  "shell"     : true,

  "osx" : {
    "path" : "/usr/local/bin:$PATH"
  },
  "windows" : {
    "cmd"  : "csc.exe $file_name"
  },

  "variants"  : [
    {
      "cmd"   : "mono $file_base_name.exe",
      "name"  : "Run",
      "shell" : true,

      "windows" : {
        "cmd": ["start", "cmd", "/k", "${file_path}/${file_base_name}.exe"]
      }
    }
  ]
}
{% endhighlight %}
  
  
**Disclaimer** - I took the base of .sublime-build from [this repo](https://github.com/chrokh/csharp-build-singlefile-sublime-text-2). I submitted a [pull request](https://github.com/chrokh/csharp-build-singlefile-sublime-text-2/pull/3) too with the enhancements.
  
Hope this helps !