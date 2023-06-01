---
title: Learning New Tech - A Project Overview
description: Why This Approach
date: "2023-04-30"
publishDate: "2023-04-30"
---

## Background

Learning a new technology can be a challenge. Everyone learns differently and has different preferences. I prefer to learn by doing, but there's a catch. At some point, the guides and tutorials you find give a gap. A lot focus on introductions to the technology, and maybe get up to something as complex as a To-Do clone; then you have the really advanced examples or cookbooks that show how to do one thing and assumes a lot of prior knowledge. The gap between those can be frustrating.

I find that I learn by leveraging what I've already learned and applying those concepts with the new tech I am learning. Although I've worked in a lot of different roles on a lot of different software types, my preference is working on HTTP APIs that "do something." So rather than just building a basic endpoint that doesn't do much, let's do something non-trivial.

Obviously, this project really assumes I am learning a tech for this purpose or similar (so, no, this probably isn't a great way to learn tools like D2Lang...) but it is useful for more general learning paths. THe nice thing is that if you have one of these running, you can target that with a client-side technology. This is how I taught myself Svelte, for example.

## What Is It?

The demo project I like to build after I get the basics of a language is a basic HTTP API, most often RESTful and JSON, that allows a user to do the following:

- Sign up as a pending user that needs to verify - Includes critical proper password encryption!
- Verifies their account
- Can view their profile only if authorized
- Can edit their profile only if authorized
- The data is stored in some sort of data store
- Automated tests
- Configuration and centralized logging / debugging

Obviously, there's a lot of implications here. The solution needs to figure out HTTP routing, encryption, data serialization / deserialization, authorization checks (probably through middleware or something similar), and testing.

## Component Explanations

With the above high-level goals, here's what the final "project" will need to look like, in the general order I would approach in a technology agnostic way:

### Configuration and Logging

The first step is getting set up. I would try to find the simplest way, the community-driven way, and find the major differences, if any, between the two. Don't over-complicate this step, as it will likely change as I learn more. For this, I would just be looking to:

- Get the tech installed and working
- Install any needed editor plugins for QOL
- Figure out if there's anything special about package management (like Python's new venv approach!)
- Set up basic logging to either STDERR or a file

With that, it depends on what the technology is to decide the next steps. The next two components could be done in either order.

### HTTP

What is the preferred HTTP routing approach? Is there a batteries-included solution? Is there a bunch of different libraries? Does the tool prefer full frameworks? Pick one and go with it.

With the selection, implement the basic endpoints that just output basic, ephemeral data. I would create the following endpoints to just make sure I get the correct routing:

- `POST /users` - Register a new account, no auth
- `POST /users/verify` - Verify a user account; could be expanded for email address changes; no auth
- `POST /login` - Login a user; I prefer not to have this on the `/users` route, but that's purely preference.
- `GET /me` - Get the user's profile, needs the auth
- `PATCH /me` - Update the user's profile, needs the auth
- `POST /me/refresh` - Refresh a user's access token
- `GET /users/{number}` - An arbitrary endpoint to look up a user's profile; if authorized then they will get theirs if the ids match; if that isn't their id OR there is no auth, they get an error

Things like password resets, account deletion, and other nice-to-haves will be left for further experimentation. At this point, these endpoint don't "do" anything, other than return placeholder data.

It's critical to note that things like crons to expire tokens, setting up cookies (so web clients don't store tokens in local storage), and other critical features aren't included. This isn't designed for production, it's designed for learning.

This is also a bit controversial, but there's no automated testing at this point. I assume that things will change. I believe tests are critical, but I am not a proponent of writing the tests when I don't even know what the output will look like. Now, if I was conforming to a spec, such as an OAS3 spec, that would be different!

### Data Model

Typically, I would want to store the data for persistence. Unless I am specifically learning a new data technology, I would default to MySQL or Postgres. For this step, establishing an open and persistent DB connection with basic CRUD operations would be the goal. The end result is being able to write the following to storage, in outline:

- User
  - id
  - firstName
  - lastName
  - email
  - password -- ENCRYPTED!!!
  - status -- one of `pending`, `active`, `inactive`, `locked`
  - createdOn
  - lastLoggedInOn
- Tokens
  - id
  - userId
  - tokenType -- used for verification and authorization, so one of `register`, `refresh`
  - token
  - expiresOn

The tokens are used initially for the registration verification but are then used for authorization. On login, a user will be given a short lived `access_token` that will be a `JWT` that can be decrypted for identity. They will be given a `refresh_token` that is NOT a `JWT`, and will be stored in the DB. The user can exchange the `refresh_token` for a new `access_token` as long as the `refresh_token` is still valid.

### Encryption with bcrypt

For obvious reasons, you cannot store passwords unencrypted and unhashed. Also, `md5` and `sha1` are effectively plaintext. Use them for checksums, but nothing secure.

If starting fresh on something I want to hit the real world, just start with `argon2`. However, there's a lot of systems that use `bcrypt`. Both are good, with `argon2` being slightly better for a variety of reasons that are beyond the scope of this post. However, since I'm likely to encounter both, we take this in two steps to also show migrating from one to the other.

For this step, we would simply encrypt the password with bcrypt and then verify the stored password against the output. It's pretty simple, but it demonstrates a few key concepts:

- Likely, there will be a need for a library or module
- We will likely want to ensure that the password field is never returned to the user
- We need to make sure we never ever store the password in plaintext, so our serializers (or equivalent) need to be validated

### Encryption with argon2

For this step, we would look at implementing `argon2` instead. For new user sign ups, we would need to start saving the password with `argon2` applied. However, that leads to an interesting problem we intentionally created (assuming we don't clear the DB before implementing this!). Namely, existing users with passwords created using `bcrypt` will not be able to log in!

So, we will need to get the password from the DB and determine which algorithm was used. You COULD theoretically store that in the user profile, but that's probably a bad idea. For example, you really don't want to leak the salts, iterations, algorithm, etc out to the world even accidentally. So it's best to do the check on the existing data, whenever possible.

If the found password is already using `argon2`, then you are golden. If it is using `bcrypt`, then you would need to replace that field with the `argon2` generated version.

In a real-world app, you would also probably want to do things like have a script that checks how many users are using the old algorithm, or marks users who have the old algorithm as locked and they need to change it, or other data security approaches to hasten the migration so you can remove the additional cycles. But that is beyond this learning project!

### Wiring

At this point, we are almost done with the logic! With the data manipulation ready to be safely stored, we can go ahead hook up the CRUD operations for Users and Tokens to the respective end points. Again, we are still doing manual testing, so having something like a Postman set up to test could be helpful. With spot checks, we will have a "functioning" API that is woefully dangerous if left as it. Why? Anyone can be anyone! Let's fix that.

### Auth

This is the last of the feature functionality we will build. In this portion, we will require access tokens to authorize the user on endpoints that need to be locked down. This could be tricky, as some technology stacks will use middlewares, others leave it up to an external library or tool that needs to be hooked up. But the end result is that authorization checks are appropriately securing the system.

We also need to worry about expired tokens. We want the access tokens to be very short lived. Users then renew them with a longer-lived refresh token. This is akin to a stripped down, non-spec-compliant oAuth2 implementation. We could actually implement oAuth2, but that would be a bit bigger than we would want to chew off here. So instead, we will just use those two tokens.

The implication, of course, is that the refresh token endpoint cannot require an active access token, but must validate the user. I prefer sending the old access token and the refresh token, which both can be used to identify and renew the user session.

### Testing

So, now we are at the finish line. Almost.

I can't in good conscience consider the experiment "done" without having automated tests. My mantra is, and always has been,

*Make It Work, Make It Right, Make It Fast*

So, in this case, we've "made it work", now it's time to "make it right". Right means many things to many people, but one of the key components for me is that it is tested with automated tests. In this final section, we implement basic automated testing. Whether we focus on unit testing or integration testing is a matter of preference. I personally prefer integration testing over unit testing, if I have to choose.

Depending on framework and tool, we do not need to worry about 100% coverage. This is mostly to get a feel for what testing looks like. How do you test the DB interaction? What about making authorized and unauthorized API calls? Here we find the patterns and answers. We may need to refactor some of our code to be more testable. Still, I put this at the end because, while learning a stack, I don't want to be worrying about ensuring I hit a perfect design and flow before I get into the weeds. I am giving grace to grow and change the implementation

## Potential Improvements

Since the goal is to learn, we intentionally cut a few corners that you would want in a production system. Some, but not all, of the additional components or features you would want could include:

- Dockerization / deployment strategies
- Scheduled jobs
- Emails and notifications
- Caching
- Monitoring / Observability / Traceability
- Performance benchmarks
- Database migrations
- Automated documentation

## Final Thoughts

When learning a new technology, try to minimize the number of new technologies you introduce. If you decide to try a new language, with a new data backend, using a new protocol, you may have a tough time figuring out what is the cause of a bug or error. I would keep it to 1 new technology at a time, or 2 if you want to really explore but have a decent understanding of the tradeoffs. Besides, you can always build on the final product after. Started with MySQL but want to learn Postgres? Well, you've already got something working, so it should be easier to swap out the data layer, for example.

Feel free to steal this idea as your own. If it works for you, awesome! If it doesn't, or you have any feedback, let me know!

### Quick Warning

This is not meant to be a fulltime job. As I do this, I am not caring about perfect code that is production ready, resilient, and performant. Heck, there's no observability, status checks, etc. That being said, it's also a bit more than just a couple hours of work. So please **do not use this as a take home test for applicants. If you want to break it apart or assign pieces of it, I can't stop you, but please respect the time of your potential future employees.
