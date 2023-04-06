---
layout: post
title: MacOS Thread Injection UNFINISHED
---

If you have ever done DLL injection on Windows, you know that it is a relatively trivial task given the ```CreateRemoteThread``` function. Library injection on MacOS is done similarly to Windows but with a few more hoops to jump through. I was largely inspired by [this post](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html) and used part of the shellcode they provided.

## Thread Injection:
There are a few hurdles to injecting a thread in a running process on mac.  

## AssaultCube:
If you want to practive writing injectable code AssaultCube is the perfect game/software for this, however on arm macs it is only emulatable through rosetta.  
If we go ahead and run our thread injection code on the AssaultCube process it immediately crashes and produces no log anywhere (that I can find).

My initial prediction was the program was being protected with SIP because it is emulated with rosetta. However this is clearly not the case, everything up to thread_resume() works without error. 

## Rosetta Emulation 
Rosetta actually maps and translates x86_64 executables ahead of time and produces .AOT executable files in /var/db/oah/. When you execute the x86 executable it actually goes ahead and executes its matching arm64 .AOT file. 

Lets addach lldb and see what happens when we run the code injection. We were able inject the thread and code without crashing, however if we try to interrupt the process, it becomes uniterruptable and hangs forever until we restart our computer.  

We know that the program crashes immediately from ```thread_resume()```, but at what point is it able to run the shellcode? If we inject our loop assembly with lldb attached we can see how many theads the process has without it immediately crashing. If restart the process and then we then inject our bootstrap assembly we can see that we have one more thread. This means that our bootstrap assembly successfully creates the pthread. Since we use our mach thread to bootstrap a fully built pthread, and leave the mach thread running, my prediction is that translated program has no idea how to deal with the a bare mach thread.  

Lets try calling ```thread_terminate``` immediately after ```thread_resume``` in our c injection code. The code now injects without crashing and we are now able to attach lldb and interrupt as we please.  


### References


![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
