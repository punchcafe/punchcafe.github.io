---
layout: post
title:  "Rookie Support Engineering"
date:   2020-05-24 22:00:00 +0100
categories: entry
---

As many companies do, where I work we enact a support rota. Every week one poor bastard is the _support engineer_, and as such it's their responsibility to be the first point of contact and incident response manager to any issues that may arise. Super fun, as you can Imagine. Over the last few months I've been to develop processes and strategies for being on support, and I've been meaning to write these up.

For context, at this point I've been a backend engineer for about 10 months, our stack is a typical managerie of web-back end, with a few cool exceptions (GraphQL I love you even if you are dangerous). I work in Java, which has been my first exposure to a strongly typed language and have had the sheer luck to work with some brilliant minds, so a large part of my initial focus was learning the fundamentals well. As such, I neglected some of the satellite parts of our system, (and bits I mistakenly took for satellite parts) such as Kibana, Kubernetes (the afformentioned mistake), prometheus, Linkerd and Spring Boot Admin. I won't call this a mistake. Underlying principles like good API design, understanding networks, design patterns (and knowing when not to use them), concurrency... (the list goes on), are to my mind, pivotal to building a _working_ knowledge of software engineering as opposed to being good at several technologies. That being said, it sure as hell made the first few shifts of being on support difficult. That brings me to my first point:

## Know your tools
As a support engineer, your role is to support the system currently deployed. That means when it's time to debug an issue, you need some way to interact with or investigate the issue. Systems, of course, have varying degrees of complexity. You could have a lightweight Rails crud application for a single entity, deployed on Heroku, with no real dependencies (except maybe a DB). In that case, most of your debugging will probably be at the application level, and often will be fairly straightforward to reproduce locally. Add the fix, redeploy, baddabing baddaboom it's all good.

Unfortunately, microservices tend to be little bit more complex. If deploying the above application is like putting it in a shop window (where it's easy to access it any time), deploying a microservice can often feel like wrapping your application up nice and tight in a pod, and then subsequently launching it to the fucking International Space Station. Debugging a complex microservice system is an entirely different ball game to debugging a single application running locally. The instincts you have with your IntelliJ debugger or whatever are all of a sudden only applicable at very specific points, to get to which you need a plethora of additional tools.

That's why it's so important to familiarise yourself with your tools.

Most systems will probably use Kubernetes. If yours does, learn it. Learn the eco system. If you're getting unexpected 504 errors, you need to understand what part Kubernetes that could be coming from from. learn how to interact with pods, you need to be able to read, edit, and execute on them. The reality is using kubernetes to deploy your app makes it part of your app. This becomes even more apparent when your system begins to tackle problems of scale, shared caches, load balancers, SSL layers and so on.

You may have a message broker, RDS services, Elastic search, or any number of other pre-existing services. These can be hosted locally, but often will be through AWS. It's worth being able to navigate a little bit of AWS, but it's only worth picking up after you're comfortable with Kubernetes.

Then you need to be comfortable with your supporting tools. Make sure you know how to search through logs. If you use something like Kibana, spend the time to get comfortable with whatever query language it employs. Make sure you know every part of supporting tooling your system has.

The reason it's so important to understand your tools well is that in order to efficiently debug, you need to be able to use your tools as an extension of your instincts. If you have an initial reaction to an error or incident which you have a hunch about, your understanding of the tooling should enable you to begin investigation straight away. It's a different discipline to app development and debugging locally, and it's not as glamorous. You don't have the rewarding feeling of spending time learning Lumine Query Language to then be able to deploy a sexy new feature. That being said, the next time you're on support and stuff goes south, the feeling you get when you spot an error in your monitoring tooling, find the root cause in the logs, use kubernetes to update an environment variable in a deployment, restart the pods and see your hotfix in action is pretty good.


## Use trello, or something like it

## Always be prioritising and re-prioritising

## Find the supporting docs
doesn't exist? be a force for change

## Keep cool
You're the support engineer. You're not there to be blamed for it going wrong. You are a product of your experience, and you're not at fault that issues take a while to get resolved. Keep things prioritised, pass off what you must, refine your practices until they feel like instinct, and keep going. That's all you can do, and all anyone can expect from you.
