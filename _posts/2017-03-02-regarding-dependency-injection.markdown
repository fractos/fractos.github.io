---
layout: post
title:  "Regarding Dependency Injection"
date:   2017-03-02 07:56:00
categories: coding
---

A colleague of mine recently posted on an internal Slack channel about this blog article: [How not to do dependency injection - the static or singleton container](http://www.devtrends.co.uk/blog/how-not-to-do-dependency-injection-the-static-or-singleton-container).

Reading the article, I became aware of something that [a friend of mine had written about](http://guy-murphy.github.io/2014/11/24/service-locator-vs-dependency-injection/), namely the absolute vitriol that many coding blogs and generally good books seem to have against Service Location. For example, I'm reading [Adaptive Code via C#](https://www.amazon.co.uk/gp/product/0735683204/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&camp=1634&creative=6738&creativeASIN=0735683204&linkCode=as2&tag=readingpile-21) at the moment and in that Gary McLean Hall has a small _meltdown_ about it, which is a shame because it's a pretty good book for people to learn about refactoring and use of basic patterns. However it also commits the cardinal sin of saying that the 'D' in SOLID is for Dependency _Injection_ and not Dependency _Inversion_, and that betrays the architectural bias of the author.

The upshot being:
- Anyone who tells you service location is an anti-pattern isn't fully aware of the problems that it is supposed to be an answer for.
- Dependency Injection moves away from the original point - separating the configuration of services from their use.
- Dependency Injection vastly increases the surface area of an object via abuse of the constructor which isn't bound by interface contract, thereby avoiding abstracting dependencies fully and leading to constructors being the medium by which relationships between objects are communicated - (this is not a virtue)
- By making assumptions about state, Dependency Injection turns architectural _uses-a_ relationships into _has-a_ - which blocks the use of singletons and a bunch of other architectural patterns where they would be appropriate.
- Further, it means that although relationships between data entities are modelled, behavioural relationships between objects are not because they become one great big dependency ball.

[Guy Murphy - Service Locator vs Dependency Injection](http://guy-murphy.github.io/2014/11/24/service-locator-vs-dependency-injection/)

Another in-depth article Guy wrote about Inversion of Control can be found here:

[Guy Murphy - I come not to burn IoC, but to praise it](http://guy-murphy.github.io/2014/11/27/I-come-not-to-bury-IoC-but-to-praise-it/)