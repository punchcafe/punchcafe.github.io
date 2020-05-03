---
layout: post
title:  "Growing Pains 01: Skinny Controllers Revisited"
date:   2020-05-03 22:00:00 +0100
categories: entry
---

The project I'm working on has recently undergone a load test, so the current focus has been around scalability. It's my first hands on exposure to a lot of the challenges of scaling an application, and as a result I've found myself rethinking a lot of what I learnt earlier on. Whats interesting to note is that (at least for me) it feels like an entire different domain the world of application development, and yet not factoring scale into your app seems like a recipe for disaster. So I've started this mini series, _growing pains_, (I make absolutely no excuses for the cheesy title) as an outlet to write down some of the musings/reflections/revelations this phase of work is bringing. Without further ado, I present episode one, _Skinny Controllers Revisited_.

In my _very_ early days of Makers, I would often come across the concept of skinny controllers / fat models. At the time I took this as (slightly confusing) gospel and tried to run with it. As with all best practices I've encountered, however, it's not until you've have enough _bad_ experience you realise why it's good.
I also got another piece of sage advice from an old tech lead. We were running a micro service architecture with a _facade_ public facing API. Within our service we had a number of entities, which in turn had statuses. I was tasked with implementing a filter by status functionality, and my initial mistake was trying to do this on the _facade_ level.(Actually, I'm lying. In truth it was on the service level, where I was filtering the result of a Spring Data repository call, but this example works better with some embellishment)
The sage advice was that what I was doing was introducing overhead. Everything the facade has to do which isn't just piping a message somewhere is overhead, and should be avoided. I could see the benefit, let the service handle service stuff and our facade stays nice and dumb. Separation of concerns, blah blah, all that nice Uncle Bob stuff. While this is all true and valid, theres another key advantage which I never considered until earlier this evening while leafing through some _Hazelcast_ documentation.

**Any and all _overhead_ in an application, couples that task to the application.**

In my exampled I cited filtering. The implication of this is that now my facade is coupled to the action of filtering entities on a different service. If demand for these entities is particularly high, as is the number of filter requests, all of a sudden I find I have to scale the instances of not only my Entity service, but the facade service too. This could also cause all sorts of issues (for example in the unlikely situation where the number of facade instances had to be strictly managed).

This also made it much clearer to me why _skinny controllers_ is so important, while also making me think about a concept I hadn't before; the coupling of a _task_ to a service (or instance of that service). Decoupling classes, encapsulation and SRP are wonderful tools for crafting clean code, but these are conceptsa which make my single application tight and well crafted. Considering the bigger picture, the whole flow of a system, which has a concrete number of instances talking to each other in concrete space makes things really a lot more interesting. This challenge of scale, well, It's a whole new ball game.
