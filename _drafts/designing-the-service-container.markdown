---
layout: post
title:  "Designing the service container"
date:   2017-02-22 23:10:02
categories: inversion
---

Inversion behaviours are aligned with Agile project techniques because:
- Small unit of code
- Unit testable

## Construction blocks
Behaviours are arranged in a service container and associated with message names. Some behaviours can define their own private lists of other behaviours that are generally considered for execution in sequence.

If a behaviour is the basic unit that we are dealing with in an Inversion service container, then it is time to start defining what the next highest unit might be.

> I want to be able to define construction blocks for elements of pipelines that exist inside a service container. The construction block could encompass multiple behaviours. The output of this system should be a fully configured service container.

## Context requirements
Behaviours can require various things from a visiting context. Filtering out situations where the requirements are not met can be off-loaded into the attached configuration - e.g. ```{ "control-state", "has", "image" }``` will add that criteria to the behaviour's Condition list. 

> I want to be able to declare the requirements of a behaviour in a way that both enforces those requirements in the behaviour code, and also allows those requirements to be aggregated up to the next level of organisational unit.

Context requirements could be parameters, control-state members, flags, evaluations or other service container items, e.g. stores.

## Configuration values
Behaviours also have configuration values. These are currently loosely coupled - the behaviour code can assert their existence from within the Condition or Action methods, and the configuration prototype itself is tailored to cover the keys that the behaviour code asserts.

> I want to be able to declare required (and optional) configuration values of a behaviour in a way that is accessible to the behaviour code and that also allows those configuration values to be aggregated up to the next level of organisational unit.

