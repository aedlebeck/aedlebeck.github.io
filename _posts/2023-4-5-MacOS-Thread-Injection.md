---
layout: post
title: Thread Injection in Rosetta 2 Emulated Process UNFINISHED
---

If you have ever done DLL injection on Windows, you know that it is a relatively trivial task given the ```CreateRemoteThread``` function. Dynamic library injection on MacOS is done similarly to Windows but with a few more hoops to jump through.  

ARM64 Injection:
I was following along a very interesting article by Knight https://knight.sc/malware/2019/03/15/code-injection-on-macos.html. After a few small modifications I was able to get the code to inject into any Mach-O 64-bit executable arm64 file that doesn't have SIP.  

AssaultCube:
A popular game for learning about and practicing code injection is AssaultCube. However the current version of AssaultCube is not supported for ARM. If we run file on the AssaultCube launcher we can see: 
```
➜ file ./launcher
./launcher: Mach-O 64-bit executable x86_64
```
If we go ahead and run our thread injection code on the AssaultCube process it immediately crashes and produces no reports anywhere. My initial prediction was the program was being protected with SIP like all apple processes. However this is clearly not the case, as everything up to thread_resume() works without error. After searching through forums posts on this specific issue you would be lead to believe that the x86 code is loaded into memory and emulated and executed in real time. So ofcourse the program would crash while trying to execute arm instructions??? NO, this is not the case at all. Rosetta actually maps and translates x86_64 executables ahead of time and produces a .AOT executables file in /var/db/oah/. All of these files are protected by SIP, but if you remove SIP 


![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
