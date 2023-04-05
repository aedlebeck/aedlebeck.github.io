---
layout: post
title: Thread Injection in Rosetta 2 Emulated Process UNFINISHED
---

If you have ever done DLL injection on Windows, you know that it is a relatively trivial task given the ```CreateRemoteThread``` function. Library injection on MacOS is done similarly to Windows but with a few more hoops to jump through. This post documents my trials and tribulations of trying to inject code into a x86 process emulated by rosetta. I was largely inspired by [this post](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html) and used part of the shellcode they provided.

<!-- ## Thread Injection:
I have already covered the details of thread injection on mac [here](). -->

## AssaultCube:
I want to be able to practice writing injectable code and AssaultCube is the perfect game/software for this, however it is only emulatable through rosetta on arm macs.  
If we go ahead and run our thread injection code on the AssaultCube process it immediately crashes and produces no log anywhere (that I can find).

My initial prediction was the program was being protected with SIP because it is emulated with rosetta. Rosetta is apple software and apple protects all apple software with SIP. However this is clearly not the case, we are not injecting into rosetta, and everything up to thread_resume() works without error. 
## Rosetta Emulation 
After searching through forums posts on this specific issue you would incorrectly be lead to believe that the x86 code is loaded into memory and emulated and executed in real time. So ofcourse we have to inject x86 code???

NO, this is not the case at all. Rosetta actually maps and translates x86_64 executables ahead of time and produces a .AOT executable files in /var/db/oah/. When you execute the x86 executable it actually goes ahead and executes its matching arm64 .AOT file. So why is it crashing then if the process is simply just an arm executable?  

## Debugging
Lets addach lldb and see what happens when we run the code injection. Huh? We were able inject the thread and code without crashing. I have no idea whats going on here. Lets try to interrupt the process. The process is now uniterruptable and hangs forever until we restart our computer. I'm even more confused now. If we run lldb on the target process without injecting, we are able to interrupt and continue the process as many times as we like without problems.  

We know that the program crashes immediately from ```thread_resume()```, but at what point is it able to run the shellcode? If we inject our loop assembly with lldb attached we can see how many theads the process has without it immediately crashing. If restart the process and then we then inject our bootstrap assembly we can see that we have one more thread. This means that our bootstrap assembly is able to be executed. Weird... our shellcode in the bare thread does get executed, but immediately crashes. Since we use our mach thread to bootstrap a fully built pthread, and leave the mach thread running, my prediction is that .AOT file has no idea how to deal with the a the bare mach thread.  

Lets try calling ```thread_terminate``` immediately after ```thread_resume``` in our c injection code. Lets test it out on AssaultCube without lldb...
Whoah our program no longer crashes and we are now able to attach lldb and interrupt as we please.  

When I have the time to dive deeper figure out the issue with how rosetta is handling the mach threads I will update this post.

### References


![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
