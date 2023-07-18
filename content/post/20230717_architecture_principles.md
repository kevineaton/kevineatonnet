---
title: Architecture Principles
description: The principles that guide my architectural decision making in tech.
date: "2023-07-17 00:00:00"
publishDate: "2023-07-17 00:00:00"
draft: false
---

Now that things have started to settle, I wanted to start to write down my views on software architecture. Most of this has been bottled-up knowledge or hidden behind corporate firewalls of documentation. So, as best I can, I am going to put fingers to keys to put out my overarching software architecture principles and thoughts.

As a software architect, I've had the blessing to work in really good software systems, really challenging software systems, and everything in-between. I've been engineer number 1 at quite a few successful startups, and I've also been engineer 100+ in some very large corporations. The key factor in understanding my approach to software architecture is that what works in one place, may not work in another. Still, these principles have guided me in my tenure, and form my current basis for understanding architecture.

*Note*: I am going to be as general as possible, and use example technologies only to prove a point. At no point should any of my thoughts be construed as talking about any specific team or org I have worked with or at. I will call out specific companies if relevant, but these are general and transcend individual organizations in my career.

## Principles not Laws

The principles mentioned in these posts are guideposts for thinking and decision making. They are not and are not intended to be hard laws or rules. This is meant to be a guide, not a gate-keep. Each should be considered flexible, nuanced, and open to discussion. Be flexible in your thinking and allow your understandings and preferences to shift as relevant.

## Overall Guidance

It's important that, before diving into the principles, a general understanding is established. Something like a thread that weaves the principles together. First, we establish a mission statement, then focus on the priorities for software development. Below are the mission statement and priorities I set for my teams, specifically when running my own organization units at KVSS Technologies and Treelight Software:

### Mission
	 	 	 	
**We will build secure, maintainable software solutions that use industry standards and solve real business problems.**

### Priorities

**Make it Work, Make it Right, Make it Fast**

**Make it Work** – Solve the immediate problem. Figure out how to implement the feature or fix the bug. Get your head around what needs to be done, and take a shot at it. The most beautiful code won’t be worth anything if you don’t Make it Work. Implementations that stay here are NOT generally ready for production UNLESS there is a unique time constraint or business impetus.

**Make it Right** – Once it works, it’s time to Make it Right. Make it Right involves many facets, but at a base level, it touches upon:

- Does it have automated tests?
- Does it have documentation and comments?
- Does it make logical sense while reading the code?
- Is it maintainable / if someone new comes along in 6 months and looks at this, will it make sense?
- Has it been peer-reviewed for quality and standards?

**Make it Fast** – Sometimes, the code must go to production before it makes it here. The reality is that there’s only so much time. If the implementation stops at Make it Right, leave notes about how to Make it Fast, create a backlog ticket, and let your team lead know. Make it Fast means taking a look at improvements to the code. This is where optimizations come in. For example:

- Can you pre-allocate memory instead of growing data structures?
- Can you tighten any hot-path loops?
- Can you tighten up any lookups or I/O?
- Are there in-memory caches or related improvements that can improve performance?

Note that you really need to make sure that you Made it Right before you Make it Fast. The input/output of this step should not change the functionality or requirements (otherwise, you didn’t Make it Work). This is pure optimization.

## Principles

Here are the principles I use for software architecture designs and decisions. Each will have it's own post over the next few days or weeks:

**Knowing What is Happening When and Where is Better than Simple Solutions**

**Industry Open Standards and Tools are Better than Proprietary Tools**

**The Best Tool May Not Be the Best Tool**

**Simple is Better than Complex**

**A Little More Effort Now is Better than A Lot More Effort Later**

**Shepherd the Future**

**Remember the Context**
