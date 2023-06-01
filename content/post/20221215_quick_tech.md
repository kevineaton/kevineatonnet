---
title: Tech I'm Using - 2022 Edition
description: Quick description of my preferred tech stack.
date: "2022-12-15 00:00:00"
publishDate: "2022-12-15 00:00:00"
draft: false
---

The benefit of teaching is I get to answer a lot of questions and share a lot of gathered experience. Looking into 2023, I figured it is long overdue to just write down a quick summary of what tech I would use to start a new project and what I am excited about potentially diving into. So, without further ado, let's do it.

## Backend

I'm still a [Go](https://go.dev/) diehard. I've used the language extensively and I absolutely love it. It's what we built 90% of the backend at [Wagz](https://wagz.com) at. It performs remarkably well, is easy to teach to onboarding engineers, is consistent in form, and is just productive.

In 2023, I think I want to check back into [Python](https://www.python.org/). It's having an explosion again, thanks to the boom in ML and AI. I haven't used it for anything non-trivial in awhile, so it will be interesting to jump back into. Specifically, I'm excited to spend some time with FastAPI. I also really want to get back into working with [Elixir](https://elixir-lang.org/). It's a similar story, but I just love the difference of the language, being a functional language from a talented creator.

For data, I still prefer MySQL. Yes, I know it has its problems, but it's familiar territory for me. I still don't enjoy when I have to dive into a MongoDB instance, as querying or retrieving anything but the simplest of data is more work than it should be. In 2023, it's probably time to spend some time with Postgres. It's not paradigm shifting, sure, but it's different enough. Similarly, I really want to dive into a time-series database, as a lot of the locations and metrics data we ingest would probably be better suited for a tool like that.

## Frontend

I would still use React, reluctantly. Unfortunately, I just don't enjoy working with hooks. I think there's too much magic, too much footgunning, and just overall it's less of an experience. I suppose I could technically continue to use my beloved Typescript class based approach to components, but that seems to be a deprecated paradigm, given the lack of support and documentation for non-hook solutions.

However, [Svelte](https://svelte.dev/) is amazing. I am planning to use it extensively in 2023, as I based part of my dissertation work on it. It's fast, dead simple, and super productive. I have the same feelings using Svelte that I did in the early days of React.

## DevOps / Infra

Here's where I try to stay pretty consistent. I enjoy [Terraform](https://www.terraform.io/) for infra, although it's overkill for side projects. [Ansible](https://www.ansible.com/) is my orchestration and configuration tool, and obviously [Docker](https://www.docker.com/) is how I package up anything I build. I use Vault for secrets, and it seems to work really nicely.

I should probably look into some telemetry technologies, and of course there's the, er, [whale-sized elephant in the room](https://kubernetes.io/) I really need to pick up. But, I just haven't hit a situation where Kubernetes was the solution. I also want to look into tools like [etcd](https://etcd.io/).

I also should mention that Docker as a company is... interesting... right now. I probably need to look into [Podman](https://podman.io/) and alternative runtimes. Plus, this [Web Assembly](https://wasmbyexample.dev/home.en-us.html) thing looks to be pretty amazing for packaging apps as well...

## Other Stuff

One of the things a lot of teams struggle with is idea sharing. I have a personal Notion account for my own KM. But communicating shared ideas and ensuring understanding is hard, especially in fully remote teams. One tool I've really started looking at are ways to document and diagram architectures. I was recently told about a new language called [D2](https://github.com/terrastruct/d2) that looks really, really promising.

## Final Thoughts

Technology evolves, and it's important to keep learning. But it's also ok to have your comfortable, go-to stack. I try to avoid [Resume Driven Development](https://rdd.io/), so I have the stacks I'm excited about. Still, it's important to keep your eye on the future so you can stay up to date.
