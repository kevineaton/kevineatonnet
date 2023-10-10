---
title: Architecture Principles - Knowing What is Happening When and Where is Better than Simple Solutions
description: Knowing What is Happening When and Where is Better than Simple Solutions
date: "2023-07-18 00:00:00"
publishDate: "2023-07-18 00:00:00"
draft: false
---

The primary concern here is that Software Engineers are Professionals. Professionals are, with varying degrees of definitions:

- Trained
- Experts
- Knowledgeable
- Communicative
- Collaborative
- Disciplined

In other words, the things that separates a Software Engineer and a Software Developer are the soft skills away from the computer. While I always prefer simple over complex, experiences Software Engineers know that there's a critical skill that "simple solutions" sometimes obfuscate: knowing exactly what is happening where, when, and why.

As engineers, it is critical that we are able to know what is happening when and where with control. When we can’t know an answer off the top of our heads due to complexity or scale, it’s important that information is easily and readily available. This includes:

- Which source code is running on production / develop / staging at any specific point in time; if customers or QA discover bugs, it should not be ambiguous which code was running
- Where data (user profiles, pet profiles, metrics, logs, events) are logically located and how to provide that data to stakeholders
- Predictable request and response patterns (minimal 500s, no 200s disguising server errors) [0]

In other words, it’s worth ensuring we can control and ascertain what is happening in our technology solutions over using “simpler tools.” Whereas a cloud provider may have a tool to make development easier, if it obfuscates where the data is, how the data is managed, or makes it challenging to debug in real time in high-stress scenarios, opt for the more open and industry-known solutions.

### Hypothetical Scenario Vignette

Let's take a hypothetical, that is only hypothetical because I am combining and obfuscating some details to protect the guilty.

There's a billing service, and it's main job is to charge customers, accounting for taxes, fees, and distribution. Sure, it does a lot of other stuff, but that's the main gist of it.

Imagine that a "simple" code development cycle had just a `main` branch. Everything was based off `main`. There's a `develop` environment, a `uat` environment, and a `production` environment. 

Developers checkout code against `main`:

`git checkout main && git pull && git checkout -b feature/awesome`

As developers wrap up their work, they commit and open a PR:

`git add . && git commit -m "awesome sauce" && git push -u origin feature/awesome`

CI runs, code passes, and is merged.

Surely you can see the issue. That code is now in `main` but it *does not reflect any specific running environment*. You don't want to push it straight to main unless you are airtight on your CI pipeline and testing. If so, why bother having `develop` and `uat` environments? So instead, on merge, the code is deployed to `develop` for testing. More commits come in and get deployed. Eventually, you need to push to `uat`. So how do you do it? Well, you could push from `main` *if no other code commits had been merged that hadn't been tested*. If any had, well, good luck. You could roll  back to a specific git commit and try to deploy from there? But that's messy. It's even worse going to production.

But wait! We're having some major issues. It looks like taxes aren't being collected! Well, you can't exactly git pull on main and assume that's the code running on production. Because it isn't. So you would need to hopefully be able to find the last commit of the code that went to production. I'm rooting for you!

Six months pass and it turns out there was a more insidious bug. A customer was repeatedly charged due to some faulty logic. Lawyers get involved, and engineering is demanded to hand over the source code that was running on the server at the exact date and time of the charges. Yes, this has happened. Not to me, but to one of my peers. The company couldn't, and lost the suit.

Now, if the "more complicated" build process was used, we'd have a different story. "What's on production?" "Well, if it's in `main`, it's on prod." "What about develop?" "Same!"

"What about six months ago?" Well, we go look at the merge and push history of the one branch, and because we were disciplined and ensured our branches matched our environments, we were quickly and easily able to find the answer.

*Note*: I can already hear folks yelling "What about tags!" If your group has the discipline to tag each release and map it to the code, and track when it went through each environment, by all means. But again, that's more complicated than the "simple" solution, and still proves the point.

### Other Examples
- Source control should appropriately model, within an hour or so of time, what code is running in which environment	
- User data should be accessible in industry-standard formats for operability and predictability
  - I am going to explicitly call out the cloud vendors here. Yeah, it may be simply to shove something in a proprietary AWS/GCP/Azure tool, but I've never once seen an organization that was being successful not regret that decision when they had to interop outside of engineering.


[0] Some view the 200 as referring to the connection, meaning that if there is an application error but the request completes, it is a 200. This is an anti-pattern. The end result is that what the user wanted to do was not “OK”, so we should not tell the user it is OK. Assume servers and clients treat a 2XX as “Request was handled exactly as expected”. Yes, this is a hill I will fight on.