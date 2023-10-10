---
title: Learning Python and FastAPI - 00 - Overview
description: Learning Python and FastAPI - 00 - Overview
date: "2023-06-01 00:00:00"
publishDate: "2023-06-01 00:00:00"
draft: true
---

## Overview

The goal of this series of posts is pretty straight-forward. I know that I personally learn by doing. I have a [project I use]({{< ref "/post/new_tech/project_overview.md" >}} "project I use") to try to learn new tools. It's a bit more complicated than, say, building yet another To-Do app, but it isn't as complicated as architecting a very complex, production-ready system.

I am going to dive into Python and FastAPI, mostly because Python is exploding in popularity again, it's been a while since I've used it professionally, and it's a good toolset to have under the proverbial belt.

It's also important to recognize a few realities. First, the Python 2/3 split was rough, but we are well past that. Python 2 is only for legacy at this point. I won't even touch upon it, except to say that some operating systems still have it as their default. Which means, you may need to get it on your own. Similarly, related tools like `pip` are typically tied to the system, so you will want a clear break (unless you really know what you are doing).

Python also provides the backbone for a bunch of tools I love, such as Ansible and various SDKs. It's an awesome language and is great as a first, or supplemental, language.

That being said, there's a few reasons I don't focus on it, and a lot of it is on me.

## Pros and Cons of Python

First, Python is easy to learn, especially in the beginning. It has a fluent syntax that, for a lot of people, just makes sense. It is easy to spin up a quick script. Important for the modern AI boom, the ability to interoperate with C/C++ code is huge. That's what powers a LOT of the power that Python brings to ML/AI tooling. You get the easy approach of Python and the raw underlying power of C. It's a great combination.

It also has a huge community with great processes (such as PEP) and tons of libraries. Chances are, if you want to get it done, you can find the tool in Python. This is great, because it also means that you don't usually have to reinvent the wheel.

But, it's not my language of choice outside of quick scripts for data analysis or manipulation. Most of this is just personal bias, since I have spent so much time with Go. But the getting started, best practices, and deployment stories are some of the weaker areas. For example, when getting started, hopefully you can get a version of Python that doesn't conflict with the base system installation (if there is one). If Python 3 is in the base, but is an older version, hopefully the new version doesn't have any conflicts. Now, that can be solved with virtual environments, but [up until recently](https://peps.python.org/pep-0668/), which one would you use? Depends on the tutorial (make sure you check those dates!), and it's easy to get confused while learning.

Deployment is not something I enjoyed when I did use it in the past. I am hoping that by working through this project, I'll figure it out and I can cross this out. However, I remember in the past that it was effectively sending the .py files to the target location, ensuring the global Python and `pip` installations were compatible, and running it. I would imagine Docker made this a lot easier, and I am thinking that the recent PEP changes have dramatically improved this, but I have a feeling I will miss the `go build .` simplicity of generating a binary and using it without any need for a runtime on the target.

There's also the fact that Python is a huge community with a long history. Some libraries were really important in the past, but have better ways of doing it now. Others are a matter of preference. It can be overwhelming to know how best to solve a typical solution. I imagine I will have to experiment as I go. For example, I imagine that I won't want to just serialize and deserialize JSON into dicts, so finding the preferred tool could take some experimentation.

All that being said, modern Python likely has shifted dramatically, and those "cons" from prior experience are likely solved, or at least improved. It is a beautiful language and ecosystem that is incredibly popular. I will have to come back and update this section when done.

## FastAPI

I know I wanted something small for the API layer. I know I didn't want a full framework like Django, not that I have anything against that. Django is awesome. But I wanted to start small. I played around with Flask, but someone pointed me to [FastAPI](https://fastapi.tiangolo.com/lo/), and having never used it, it looks interesting. So, why not?

From what I can tell, FastAPI is a bit more cohesive than Flask, which is a minimalist routing library. But it's not as full-framework as Django, as far as I can tell. It reports to be impressively performant, easy to get started, and comes with a lot of cool bells and whistles, like OpenAPI documentation built in.

## Plan

So, my plan is to first get reacquainted with modern Python. I will try to list the most relevant guides and docs I use, but I typically start with the official docs, find a decent tour-style course or video, and then go from there. While I won't link to every possible site or blog post I find, the major ones I rely on I will share as I go.

Next, I will start with brushing up on the official docs for the tools. Given that I plan to focus on the HTTP router first, that likely means the FastAPI official docs.

So, with all that said, let's dive in!

## Quick Aside

Note: At the time of this post, I have [some extra time on my hands]({{< ref "/post/20230531_crashing.md" >}} "some extra time on my hands"). If you'd like to help reduce that time, it would be appreciated!
