---
layout: post
title:  "Extending Inversion - 1: Extended prototypes"
date:   2017-03-11 14:24:48
categories: inversion
---

_This article is part of a series describing my various projects extending Inversion - a behavioural micro-framework for .Net_

Inversion on github: [https://github.com/guy-murphy/inversion-dev](https://github.com/guy-murphy/inversion-dev)

Inversion.Extensibility on github: [https://github.com/fractos/inversion-extensibility](https://github.com/fractos/inversion-extensibility)

# Extended condition prototypes

## Default available criteria

The IConfiguration mechanism for a PrototypedBehaviour (see [Inversion.Process](https://github.com/guy-murphy/inversion-dev/tree/master/Inversion.Process)) allows a bag of string tuples to be associated with an instantiation of a behaviour, such that they can carry criteria for the behaviour's Condition and also configuration items that are accessable during the Condition or Action methods. These can be defined in whatever service container you are using. For example, here is a simple setup for ParameterisedSequenceBehaviour using Inversion.Naiad:

```
Inversion.Naiad.ServiceContainer.Instance.RegisterNonSingleton("event-behaviours",
    container => new List<IProcessBehaviour>
    {
        new ParameterisedSequenceBehaviour("my-message",
            new Configuration.Builder {
                {"context", "flagged", "enable-next-message", "true"},
                {"control-state", "has", "user"},
                {"fire", "next-message"}
            })
    });
```

So, a list of behaviours is being added to the service container with the name "event-behaviours". The scenario here is that an execution process would pull that list from the service container by name, register the list of behaviours with a context and then fire an event with the message name "my-message".

The list contains one item - ParameterisedSequenceBehaviour - which will respond to an event with the message name "my-message". That's all well and good, but we'd like to be a little bit more selective about when the behaviour will execute. In this case, I only want to execute the behaviour if these things are true:

- The event name is "my-message"
- The context has a flag set called "enable-next-message"
- The context's control state contains an object called "user"

Out of the box, the default [Inversion.Process.Behaviour.Prototype](https://github.com/guy-murphy/inversion-dev/blob/master/Inversion.Process/Behaviour/Prototype.cs) class will let you use the following default criteria in conditions:

| Frame | Slot | Name | Value | Description |
| ----- | ---- | ---- | ----- | ----------- |
| event | has | (x) | | TRUE if event.Params contains (x) |
| event | match | (x) | (y) | TRUE if event.Params[(x)] == (y) |
| event | excludes | (x) | | TRUE if event.Params does not contain (x) |
| context | has | (x) | | TRUE if context.Params contains (x) |
| context | match | (x) | (y) | TRUE if context.Params[(x)] == (y) |
| context | match-any | (x) | (y) | TRUE if any context.Params[(x)] == (y) across all match-any rules<br />e.g. if specified twice for different Values, then TRUE if any match. |
| context | match-none | (x) | (y) | TRUE if context.Params[(x)] != (y) |
| context | flagged | (x) | | TRUE if context.Flags[(x)] is set |
| context | flagged | (x) | (y) | TRUE if context.Flags[(x)] == (y) (where y = "true" \|\| "false") |
| context | excludes | (x) | | TRUE if context.Params does not contain (x) |
| control-state | has | (x) | | TRUE if context.ControlState contains (x) |
| control-state | excludes | (x) | | TRUE if context.ControlState does not contain (x) |
{:.mbtablestyle}
<br />
These criteria are defined as being instances of Case objects, as defined in the Prototype class. When any behaviour that inherits from PrototypedBehaviour (or PrototypedWebBehaviour) is constructed, the bag of tuples that make up its Configuration is examined and any Case that matches is added to an internal list of the instantiated behaviour. The list is checked during the Condition method after the base Condition has determined if the message name matches. The only Case functions that are added are those which match the Configuration tuples, which cuts down hugely on the logic that each behaviour needs to perform during their Condition evaluation.

