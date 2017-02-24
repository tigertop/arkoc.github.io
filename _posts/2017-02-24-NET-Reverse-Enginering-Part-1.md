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

This program has remote-activation and this post will be about removing it.

1. Remove any anti-reversing protections.
2. Make program decompilable and runnable.
3. Find out a place where activation is made and remove it.

<!--more-->
