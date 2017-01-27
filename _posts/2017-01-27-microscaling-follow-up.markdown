---
layout: post
title:  "Microscaling follow-up"
date:   2017-01-27 15:12:00
categories: containers microscaling
---

Hello. Long time no follow-up blog about this.

My [original post](http://circle-theory.blogspot.com/2015/11/microscaling-with-load-balancing-in.html) detailed the problem that I was seeing with using Amazon's EC2 Container Service to manage a cluster of identical containers that required to have traffic routed to them individually from a load balancer. At the time, this couldn't be performed without extra tooling around the system, and I hinted that I had come up with a solution that would allow the process of [_microscaling_](http://microscaling.org) to be achieved on EC2.

I had a part two of that article sitting in draft for well over a year; various things happened - we bought our first house, my partner became pregnant and our daughter was born back in October, and part one even got cited by [force12.io](http://force12.io) on their [microscaling.org](http://microscaling.org) page in their collection of papers - so, you will perhaps forgive me for being a little distracted and not getting around to finishing that bit off.

Meantime, things moved along in the world of ECS.

The Application Load Balancer feature arrived and after a couple of months of me studiously ignoring its existence, I can now say that it does exactly what I needed Warden to do in the first place, namely that an ECS Service is now not limited to static port routing and a new Service instance with a dynamically allocated external port on the container host can be routed to automatically by the ALB. (Previously, it had meant that adding a new Service instance would require a new container host to be added to a cluster as the "Classic" Elastic Load Balancer could not route automatically to the dynamic ports.)

However, I will document what I'd come up with that filled the gap.

## Warden

Warden ([https://github.com/fractos/warden](https://github.com/fractos/warden)) was my hack for this, which enabled me to run multiple identical containers on an ECS cluster and have them load-balanced. It is the Go version of a set of Perl scripts that I had created which managed the ECS cluster of image servers.

It was impossible to manage the level (number of instances) of an ECS Service if its definition required a static container port and you wanted to re-use the same host for as many container instances as possible. So, instead of defining an ECS Service, Warden's Manager would manage the number of Tasks that were programatically being run across the cluster, increasing or decreasing them according to a metric (e.g. number of Elastic Beanstalk instances in an upstream system that calls the image servers I'm working on). ECS would then schedule the new Tasks across the cluster and the Registrar process would detect a change in the running container instances on a host, updating the host's local Nginx routing setup to match and enrolling or removing the host from an associated Elastic Load Balancer if required.
Each container host in the cluster would have two extra containers running:
- Redx ([https://github.com/CBarraford/docker-redx](https://github.com/CBarraford/docker-redx))
Which is a modified version of Nginx that takes its routing configuration from a live Redis database.
- Redis
To serve as Redx's configuration database. Lua code in the Nginx config allows the routing configuration to be read dynamically from Redis, so front-ends and back-ends can be added or removed without having to restart the Nginx process.

The cluster would need a central Redis instance that would serve as the service database and competition platform for Warden instances.
1. Synchronised the list of currently active containers on a host with a local nginx configuration held in Redis - "Registrar"
2. Managed the number of Tasks running across an ECS cluster for a particular Task Definition according to a connected metric - "Manager"

### The Registrar
The Registrar periodically examines the list of Docker containers that are running on the host and, for those it recognises, it inspects the container, pulling out the local IP address that Docker has assigned. The IP addresses and the exposed port numbers are used to update the Nginx configuration held in a local Redis. Each Service has an Nginx front-end which is on a well-known port, unique for that Service. The separate container instances are added as Nginx back-ends associated with the Service front-end. Arriving traffic is therefore load-balanced internally across all the matching containers on the container host. If a Service has any container instances on the host then Warden ensures the EC2 instance is enrolled in the assigned Elastic Load Balancer, and removes it from the ELB if there are zero container instances.

### The Manager 
This process would also run on each container host, but the instances would hold a centralised competition for a leader. Each instance detects if the currently presiding leader's Availability Zone is listed as currently active and checks for a recent heartbeat message from the leader. If there is a problem, then a competition is held and instances roll random numbers as their entry. The winner is picked from a Redis sorted set and they become the leader. A kill message is lodged for the previous leader to pick up if they return from whatever disaster befell their AZ.

Meanwhile, the new leader emits heartbeats and measures how many Tasks for a Service need to be running by using a specified metric. I'd hard-coded this in the Perl version to look at the number of Elastic Beanstalk instances were running for a particular application and then using a set of files in S3 to map between the metric and the number of Tasks that should run, like a very simple static database. Manager uses the AWS SDK to increase or decrease the running number of Tasks for a particular Task Definition across the ECS cluster before waiting and looping its lifecycle.

### Afterwards

I never got around to writing a clean way to define a Service in such a way that meant Warden could pick up its configuration dynamically. Also, how to abstract away the metrics that Manager would use to decide on Task numbers. I'd like to look into how to produce a plug-in architecture for Go that is friendly to being completely agnostic for both of these factors.

Further, the network topology that the project moved towards made it unnecessary to run Warden's Manager. Originally there was one ECS cluster that spread across the three Availability Zones in the eu-west-1 Region and there was no notion of an ECS Service to keep a desired number of container instances running across the cluster, so Manager filled that gap. Then there were changes that were made in response to realising that Elastic File System, once it eventually emerged from Preview, was more expensive to run than separate NFS volumes and servers per AZ. This meant splitting up the system into verticals - one per AZ - so that each layer of the system would only deal with one NFS volume and one ECS cluster per AZ. Warden's Registrar still ran on each container host to synchronise the load balancing across the image server containers, but the number of containers running was managed by an ECS Service.

It was a good exploration of what was possible with a bit of scripting and glue. Experience of producing tooling in Go that was reactive to system conditions has also been invaluable.

For comparison, this is the original version of Warden which was written in Perl and effectively is just glue between the Redis, Docker and AWS CLIs: [https://github.com/fractos/warden-perl](https://github.com/fractos/warden-perl).

Go was a natural choice for making a better, more solid version of Warden, mainly because of its Redis and AWS SDK access via libraries.