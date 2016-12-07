---
layout: post
title:  "Configuring a containerised system"
date:   2016-12-07 16:47:00
categories: containers
---
There are five basic methods of passing configuration into a container.

## 1. Building in configuration files
At build time, copy configuration files into the container image.

- inflexible

## 2. Environment variables
Discrete environment variable values injected at container instantiation time.

- flexible - can re-configure per instance
- good integration with Docker, ECS, shell

## 3. Holding configuration in S3
Pass S3 address to container by either an environment variable or a command parameter. Scripts or apps then retrieve dynamic configuration from the S3 bucket. Can be private, with ambient credentials assumed from an associated EC2 role on the container host, or by AWS credentials that are passed by environment variables.

- allows dynamic configuration during container lifetime

## 4. Attach a configuration volume
A specific volume that holds configuration files can be attached to the container during instantiation.

- allows dynamic configuration during container lifetime
- have to start managing volumes across container hosts
- Kubernetes uses this type of solution

## 5. Use a configuration server/service
Pass server details into the container at instantiation time. Scripts or apps will retrieve configuration from a service such as etcd, Redis, a database, etc.

- client dependencies
- container contents may have to be adapted to integrate with a configuration server
- allows dynamic configuration during container lifetime