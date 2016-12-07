---
layout: post
title:  "Microscaling with load-balancing in Amazon ECS part 1"
date:   2015-11-07 15:00:00
categories: containers microscaling
---
Currently I am involved in the development of a digital imaging platform for libraries and other institutions (the Digital Library Cloud Service, see [http://dlcs.info](http://dlcs.info)) This platform uses a wide palette of Amazon Web Services for the orchestration of moving image assets between storage systems, retrieval of metadata and the presentation of deep-zoom image tiles. It's sorta-kinda an "elastic image server" which supports the [IIIF](http://iiif.io) standard.

The system is split into two layers - Orchestration and Image-Serving. Both of these are intended to auto-scale independently, and this has been achieved with the Orchestration layer by deploying it as an Elastic Beanstalk application. Since this application is (currently) a .Net application that lives in IIS, then this environment suits it well.

The Image-Serving layer is Linux-based, comprising of a lightweight HTTP daemon (lighttpd) and [Ruven Patel's IIPSrv](http://iipimage.sourceforge.net/) with optimised Jpeg-2000 support. This is quite easily deployed as a Docker container and that is what our current prototype uses.

Since we are using Docker anyway, I looked into how Amazon's Elastic Container Service could be used to auto-scale the Image-Serving layer on demand.

What I found was that the concept of a Service in ECS did not cover how I thought it could be used.

### Services under ECS
[ECS documentation about load balancing](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-load-balancing.html)

A Service in ECS represents parts of an application that are distributed across various Docker containers. A cluster of EC2 instances form a group of hosts that a Service may be deployed to. Each host may contain a maximum of one instance of a Service. When scaling occurs, the EC2 cluster can expand or contract with Auto-Scaling Group rules and the ECS Scheduler decides where new instances are placed.

The limitation of a maximum of a single Service instance per host is one that I was not expecting, but can understand given the way Elastic Load Balancers are configured. In ECS, the port numbers of containers in the Service are static because you'll only get one instance of the Service per host, and therefore load balancing can occur across the cluster in a static way (from the Elastic Load Balancer's point of view).

When I saw the setup dialog for a Task in ECS mention the internal and external ports for a Docker container, I had hoped that this implied that the external port on the container could be balanced across if it was left blank, but that turned out not to be the case.

### Microscaling

What I wanted was the ability to work with multiple similar Docker containers populating EC2 cluster members dynamically in response to some metric - e.g. a CloudWatch alarm from our orchestration layer's load balancer. As that Elastic Beanstalk application scales in response to demand, so would the number of deployed containers up to the available space of the hosts in the cluster as defined in ECS.

It turns out that this technique has a term - microscaling - and I've been researching it recently, including the offering from [force12.io](http://force12.io/)

Once the number of containers is under dynamic control, the cluster itself could then expand or contract by responding to a separate Auto-Scaling Group rule.

### Load-balancing

The other issue is load balancing across the cluster. In the scenario I want, the ports on the deployed containers would need to be routed to directly by the load balancer, so that means multiple different ports per host. This is something which would bend the ELB architecture out of shape, I think.


### Use case and solution

Our use case is that I want to have an auto-scaling group of EC2 instances in an ECS cluster which host multiple similar containers that will listen on the same internal port, but are routed to via their external port that is assigned by Docker (e.g. while still appearing to be port 80 inside the container). They will be available to be load-balanced across all the containers that are running in the cluster. These large EC2 instances could host multiple different 'services' comprised of similar containers.

Essentially, we'd like to have a number of large container hosts and fill them on-demand with web servers or other cookie-cutter services, with load balancing going direct to the containers, across a cluster, across AZs.


I have created a system for performing both microscaling and load-balancing within ECS, and I will show the solution in the next blog post.
