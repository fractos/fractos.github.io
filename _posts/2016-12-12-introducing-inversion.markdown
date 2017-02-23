---
layout: post
title:  "Introducing Inversion (draft)"
date:   2016-12-12 20:48:00
categories: inversion
---
*Repository: <https://github.com/guy-murphy/inversion-dev>*

Inversion is a .Net library for writing *behavioural* or *reactive* code that utilises the [Actor Model](https://en.wikipedia.org/wiki/Actor_model) while also allowing isolation and decoupling from an environment in a manner reminiscent of [Hexagonal Architecture](http://alistair.cockburn.us/Hexagonal+architecture) (also known as "Ports and Adapters").

Classes called *behaviours* are registered to a message-bus and can have a set of conditions associated with it. Messages or *events* are 'fired' onto the bus with an attached set of data called the *context*. When a message is received by a behaviour and all its conditions are satisfied by the data in the event / context, then its action will execute. This behaviour is therefore said to 'react' to the current state of the context.

They should be implicitly singleton in nature, so should carry no state of their own between invocations, or indeed between their Condition and Action methods being called when reacting to an event.

Behaviours are usually small components with enough exposed configuration to allow them to be tailored to a variety of situations. This encourages code re-use and simplicity. It embodies the Unix tradition of building complex, reliable systems and pipelines from small tools that each do one thing well.

## The message bus
This is currently implemented using the [.Net Reactive Extensions](https://github.com/Reactive-Extensions/Rx.NET) library. However, any ICollection&lt;Inversion.Process.IProcessBehaviour&gt; could be used, provided it allows the collection to be altered during enumeration. As it stands, the Rx implementation is fast and has proven itself to be extremely reliable in a wide variety of situations.

## The Context
This is the Actor in the Actor model. It carries a set of state with it while it 'visits' instantiated behaviours that are registered to the message bus.

The most commonly used Context is described by the [Inversion.Process.IProcessContext](https://github.com/guy-murphy/inversion-dev/blob/master/Inversion.Process/IProcessContext.cs) interface and is implemented by [Inversion.Process.ProcessContext](https://github.com/guy-murphy/inversion-dev/blob/master/Inversion.Process/ProcessContext.cs). It provides a set of basic methods and members that represent the state of parameters, objects, flags, error messages, cache memory and the service container (which might be specific to this instance). The ProcessContext class is extended sometimes to allow access to things such as web request/response objects.

There are various specialised descendants of ProcessContext, the most notable being those that support web requests by including access to underlying Request and Response objects (e.g. [Inversion.Web.AspNet.AspNetContext.cs](https://github.com/guy-murphy/inversion-dev/blob/master/Inversion.Web.AspNet/AspNetContext.cs)).

### Params
Params is a dictionary of key-value pairs that are purely strings. These are intended for general use.

### Control State
The Control State is a dictionary of live objects that are held by a Context during its lifetime. IProcessContext defines this as a string-keyed dictionary implemented (in Inversion.Process.ProcessContext) by an [Inversion.Collections.DataDictionary&lt;object&gt;](https://github.com/guy-murphy/inversion-dev/blob/master/Inversion/Collections/DataDictionary.cs).

### Flags
The Flags collection is a simple string-keyed dictionary of boolean values. Flags can be set and retrieved. If a flag is not a member of the collection then this equates to a value of false.

### Others

## Events

## Behaviours

## Pipelines

*See [Inversion.Process.Pipeline](https://github.com/fractos/inversion-extensibility/blob/master/Inversion.Extensibility/Process/Pipeline/IPipelineProvider.cs) in [https://github.com/fractos/inversion-extensibility](https://github.com/fractos/inversion-extensibility)*

## Service containers

*[Inversion.Process.IServiceContainer](https://github.com/guy-murphy/inversion-dev/blob/master/Inversion.Process/IServiceContainer.cs) in [https://github.com/guy-murphy/inversion-dev](https://github.com/guy-murphy/inversion-dev)*
*[Inversion.Naiad](https://github.com/guy-murphy/inversion-dev/blob/master/Inversion.Naiad) in [https://github.com/guy-murphy/inversion-dev](https://github.com/guy-murphy/inversion-dev)*

## Isolation / decoupling ("Ports and Adapters")

### Bus meets world

### Inversion.Web

#### Parse, React, ViewState, Render

#### Defining your own lifecycle

#### ControlState as source of truth
i.e. don't trust parameters, but do trust loaded objects.

## Data

### Inversion.IData and the Control State
#### Self-expression in JSON and XML

### Inversion.Data

## Inversion.Extensibility

### Inversion.Messaging.Process.Engine
Running pipeline workloads from queues.

### Inversion.Ultrastructure
Linking pipelines together via 3rd party pub-sub mechanisms.