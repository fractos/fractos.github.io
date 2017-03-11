---
layout: post
title:  "Extending Inversion - 3: Designing the service container"
date:   2017-03-11 23:10:02
categories: inversion
---

_This article is part of a series describing my various projects extending Inversion - a behavioural micro-framework for .Net_

# Design Goals

## Construction blocks
Behaviours are arranged in a service container and associated with message names. Some behaviours can define their own private lists of other behaviours that have their conditions considered for execution in sequence, e.g. BlockBehaviour, LoopBehaviour or PrototypedConcomitantBehaviour from the [Inversion.Extensibility](https://github.com/fractos/inversion-extensibility) assembly.

If a behaviour is the basic unit that we are dealing with in an Inversion service container, then it is time to start defining what the next highest unit might be.

Each behaviour can have its own set of configuration criteria and because they are completely independent, 

> I want to be able to define construction blocks for elements of pipelines that exist inside a service container. The construction block could encompass multiple behaviours. The output of this system should be a fully configured service container.

## Configuration requirements
Behaviours can require various things from a visiting context. The filters for situations where the requirements are met are off-loaded into the Condition methods that are subscribed to the message bus.

The PrototypedBehaviour class efficiently encapsulates this functionality by only assigning to a Condition method those function lambdas that are necessary. Further, the set of prototypes is extensible by an application which allows custom conditions to be added that can use domain-specific parlance.

* TODO include example of { "control-state", "image-has-roles", "image" } in CustomPrototypeProvider.cs

> I want to be able to declare the requirements of a behaviour in a way that both enforces those requirements in the behaviour code, and also allows those requirements to be aggregated up to the next level of organisational unit.

Context requirements could be parameters, control-state members, flags, evaluations or other service container items, e.g. stores.

Behaviours also have configuration values. These are currently loosely coupled - the behaviour code can assert their existence from within the Condition or Action methods, and the configuration prototype itself is tailored to cover the keys that the behaviour code asserts.

> I want to be able to declare required (and optional) configuration values of a behaviour in a way that is accessible to the behaviour code and that also allows those configuration values to be aggregated up to the next level of organisational unit.


# Inversion.Process.Architect

TaoBehaviour -> Praxis.

Construct -> AggregatePraxis.

"TaoBehaviour" - because I read Christopher Alexander's "The Timeless Way of Building" and the Zen bits of it had an effect on me.


So you have the list of parameters etc that are required by the behaviours.

Glaring problem is that EVERYTHING has to be a TaoBehaviour for this all to work.

