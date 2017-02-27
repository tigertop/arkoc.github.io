---
layout: post
title: .NET Reverse Enginering - Part 1
---

Do you ever have the feeling that you are a piece of shit and you are just a man who knows combining source codes found from StackOverflow?

No ? Go fuck yourself. I periodically feel that.

This feeling challenges me to learn new things, find ways to test my brain and Reversing was a thing that I chose to start feeling developer (something a bit different from shit) again.

It's my secret of learning new things:

1. Feel like a shit.
2. Do some "cool" stuff.
3. Wait 15 minutes and go to point 1. ( Don't use gotos in source codes. )

Ok, when your mood is ready, let's start the main topic. 

We have obfuscated crypter that is written with VB.NET. Our main goal is to find out the logic behind of encrypting. It is the hardest part of reversing when you need to figure out logic, not just write a patch or find out a secret key in the program. 

In this part:

1. Remove any anti-reversing protections.
2. Make program decompilable and runnable.

<!--more-->

Here can be a huge text about trying multiple deobfuscators with no luck, let's just skip this step and let's try with our hands. (right hand is most productive)
u
Let's load our assembly with [dnSpy](https://github.com/0xd4d/dnSpy).
Let's look through our module and classes and what we notice is that methods defined in the assembly aren't decompiled. 

![messed methods](http://arkoc.github.io/images/re_part1_1.png)

It means that our methods are encrypted in some hidden section (not in IL section) and when the program runs it decrypts from a section and put them in right place. To be sure let's open our assembly with CFF Explorer and navigate to PE sections.

![cff explorer](http://arkoc.github.io/images/re_part1_2.png)

Yes, we are right. Now we must find a place in the source code where functions decryption and fixing occurs. It is obvious that it is made in `<Module>.ctor`. This constructor is called before main application Entry Point, it means that it is perfect place for such kind of work,

Reopen dnSpy, Right Click on Assembly -> Go to <Module>.cctor.

![module ctor](http://arkoc.github.io/images/re_part1_3.png)

Because of there are functions in `<Module>` class that are not decompiled as well, we must think that methods fixing must be accrued in the first call, let's dive in.

Here we see a call to the function that is imported from kernel32.dll `VirtualProtect`. This function is setting access attributes in memory blocks.  Ok, what it gives to us: the program gets sections address (where are real methods) sets `PAGE_EXECUTE` attribute on it and....we don't want to know more. We find places where methods are fixed, that's it.

![virtual protect](http://arkoc.github.io/images/re_part1_4.png)

Let's set a breakpoint after that call and dump assembly from memory.

![dump assembly](http://arkoc.github.io/images/re_part1_5.png)

Save it and open newly saved module with dnSpy.
Now we see that our methods are decompiled and are feeling good. We do it!

But here is a problem, our .exe is broken, because we forget to remove "methods fixing function". The program still thinks that he need to fix his methods (but methods are already fixed ) and completely is messing up the code.

Let's remove that function and save assembly again.

![saving assembly](http://arkoc.github.io/images/re_part1_6.png)
![saving assembly](http://arkoc.github.io/images/re_part1_7.png)

Then click on module then File -> Save Module.
Don't forget to set MD Writer options like this:

![md_writer](http://arkoc.github.io/images/re_part1_8.png)

Ok, now we have runnable .exe with decompiled methods.

Here is [ReverseEnginering](https://www.youtube.com/watch?v=Itt1nn9aWz0) music for you.

In the next part we will try to hack activation.




