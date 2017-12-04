---
layout: post
title:  "Extending Inversion - 1: Extended prototypes"
date:   2017-03-11 14:24:48
categories: inversion
---

_This article is part of a series describing my various projects extending Inversion - a behavioural micro-framework for Microsoft .Net_.

_Inversion on GitHub: [https://github.com/guy-murphy/inversion-dev](https://github.com/guy-murphy/inversion-dev)_

_Inversion.Extensibility on GitHub: [https://github.com/fractos/inversion-extensibility](https://github.com/fractos/inversion-extensibility)_

## Goals

The aim of this article is to demonstrate the current set of prototype criteria available in Inversion, how these have been extended in Inversion.Extensibility, and how they can be further extended with custom logic.

## Inversion.Process - Prototype defaults

The IConfiguration mechanism for a PrototypedBehaviour (see [Inversion.Process](https://github.com/guy-murphy/inversion-dev/tree/master/Inversion.Process)) allows a bag of string tuples to be associated with an instantiation of a behaviour, such that they can carry criteria for the behaviour's Condition and also configuration items that are accessable during the Condition or Action methods. These can be defined in whatever service container you are using. For example, here is a simple setup for ParameterisedSequenceBehaviour using Inversion.Naiad:

<script src="https://gist.github.com/fractos/8a1b83ab057b3f1565dd50d41af9cb91.js"></script>

So, a list of behaviours is being added to the service container with the name "event-behaviours". The scenario here is that an execution process would pull that list from the service container by name, register the list of behaviours with a context and then fire an event with the message name "my-message".

The list contains one item - ParameterisedSequenceBehaviour - which will respond to an event with the message name "my-message". That's all well and good, but we'd like to be a little bit more selective about when the behaviour will execute. In this case, I only want to execute the behaviour if these things are true:

- The event name is "my-message"
- The context has a flag set called "enable-next-message"
- The context's control state contains an object called "user"

All items in the configuration are available to the behaviour's instance. This means that they can introspect the set of stanzas and retrieve configuration values very easily. The Inversion.Process.Configuration class includes various access methods to retrieve items from the (frame, slot, name, value) structure, and these are extended in the Inversion.Extensibility.Extensions.IConfigurationEx class (see below).

* TODO link above to IConfigurationEx section, below.

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
These criteria are defined with instances of Case objects, as defined in the Prototype class. When any behaviour that inherits from PrototypedBehaviour (or PrototypedWebBehaviour) is constructed, the bag of tuples that make up its Configuration is examined and any Case that matches is added to an internal list of the instantiated behaviour. The list is checked during the Condition method after the base version of Condition has determined if the message name matches. Because the list of Case functions is kept to only those that match the Configuration tuples, this cuts down hugely on the logic that each behaviour needs to perform during their Condition evaluation.

So, in this example:

* TODO include gist of simple configuration of prototypedbehaviour here

Only the following Case objects will be added to the instance of the PrototypedBehaviour's Configuration state:

* TODO include gist of applicable Case stanzas

## Inversion.Extensibility.Prototypes

The Inversion.Extensibility repo contains the Prototypes class which can be used to extend the list of Cases that are available to configure PrototypedBehaviour instances with.

The new prototypes make heavy use of an extension method called GetEffectiveStringResult ([ControlStateEx.cs](https://github.com/fractos/inversion-extensibility/blob/master/Inversion.Extensibility/Extensibility/Extensions/ControlStateEx.cs)) that will attempt to render a string value for an item in the control state. The name of the item can be suffixed with a JSONPath which will address a particular JSON property of an IData-based object. TextData-based objects will simply render as their value. 

By calling the static method AddPrototypes() during application startup, the following Case methods will become available to PrototypedBehaviour Configuration blocks:

| Frame | Slot | Name | Value | Description |
| ----- | ---- | ---- | ----- | ----------- |
| control-state | equals | (x) | (y) | TRUE if GetEffectiveStringResult( (x) ) == (y) |
| control-state | not-equals | (x) | (y) | TRUE if GetEffectiveStringResult ( (x) ) != (y) |
| control-state | empty | (x) | | TRUE if String.IsNullOrEmpty( GetEffectiveStringResult( (x) ) ) |
| control-state | not-empty | (x) | | TRUE if !String.IsNullOrEmpty( GetEffectiveStringResult( (x) ) ) |
| control-state | value-contains | (x) | (y) | TRUE if GetEffectiveStringResult( (x) ).Contains( (y) ) |
| control-state | value-does-not-contain | (x) | (y) | TRUE if !GetEffectiveStringResult( (x) ).Contains( (y) ) |
| object-cache | includes | (x) | | TRUE if context.ObjectCache contains an object called (x) |
| object-cache | excludes | (x) | | TRUE if context.ObjectCache does not contain an object called (x) |
| context | type | (x) | (y) | TRUE if context.Params[ (x) ] value can be safely converted to type (y) |
| context | empty | (x) | | TRUE if String.IsNullOrEmpty( context.Params[ (x) ] ) |
| context | not-empty | (x) | | TRUE if !String.IsNullOrEmpty( context.Params[ (x) ] ) |
| context | contains | (x) | (y) | TRUE if context.Params[ (x) ].Contains( (y) ) |
| context | not-contains | (x) | (y) | TRUE if !context.Params[ (x) ].Contains( (y) ) |
{:.mbtablestyle}
<br />
These give a great deal more flexibility in behaviour configuration, allowing the following to be reacted to:

- values and partial content of control-state items
- values of JSONPath-addressed properties of control-state items
- type and partial content of context parameters
- object cache membership

<script src="https://gist.github.com/fractos/ab1e57e841276c2fb3c14bdf9e237c4c.js"></script>

* TODO add ("control-state", "type") to Inversion.Extensibility prototypes
* TODO application-custom prototypes ("image", "has-roles")
* TODO IConfigurationEx extension methods
* TODO evals
