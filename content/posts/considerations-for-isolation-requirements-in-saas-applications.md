---
title: "Considerations for isolation requirements in SaaS applications"
date: 2023-01-19
draft: true
---

A topic that often comes up when building SaaS applications is isolation. 

Isolation requirements have many shades of grey. To drive clarity, I suggest using AWS Well-Architected Framework pillars in the context of isolation requirements. I found this approach helpful to set expectations and boundaries. I suggest that you consider the below areas to determine your isolation requirements:
* Performance isolation (performance efficiency pillar). For example, number of requests per tenant
* Security isolation (security pillar). For example, a user can access data of multiple tenants, but only if they are admin
* Fault isolation (reliability pillar). For example, free tier tenants uptime per month can be X, premium tier should be Y
* Update isolation (operational excellence pillar). For example, update free tier tenant, have a bake time of 1 day, and then update premium tier customers one-by-one
* Cost isolation (cost efficiency pillar). For example, whether a customer in free tier submit endless number of jobs

Having requirements for the above areas will help you to identify the right isolation requirements in your architecture.