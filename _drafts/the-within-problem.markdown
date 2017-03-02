---
layout: post
title:  "The &quot;within&quot; problem"
date:   2016-12-07 23:20:00
categories: iiif
---
# The "Within" Problem

[@MattMcGrattan](https://twitter.com/MattMcGrattan) and I were chatting about how to approach the problem of searching OCR text that belongs to images that are annotations upon IIIF canvases in potentially many IIIF manifests that are themselves within potentially many IIIF collections.

Searching them isn't in itself the problem and it will result in an obedient list of canvas IDs produced from ElasticSearch. The problem comes when trying to find the manifest and collections that they belong to. The search documents themselves can't reference the manifest and collections as it would become practically impossible to associate canvases with more IIIF resources, let alone move a canvas from one manifest to another.

The idea has been mooted of storing the relationships between Canvas, Manifest and Collection in a graph database which would allow the easy retrieval and update of those relationships. It is apparently possible to include queries to a graph database that conforms to a particular interface from within an ElasticSearch query.

I think there are actually two parts of the problem in the storage side - firstly, the association of manifests to collections, and secondly the association of canvases to manifests. A collection could be entirely arbitrary or transient, and to be more about a facet of manifest metadata or the general contents of manifests being associated together. I believe that the relationship of a manifest within a collection, or a collection being within another collection are likely to change independently of a canvas changing its association with a manifest or increasing the cardinality of the manifests of which it is associated.

I suspect that it will be more efficient to perform a query about the results of the ElasticSearch operation in two stages - firstly to collect the manifests that the canvas is associated with, and secondly to collect the collections that those manifests are associated with.

This could be represented in a graph database, however I believe that it is more pragmatic, optimal and efficient to store the relationships within a fast key-value store like Redis.