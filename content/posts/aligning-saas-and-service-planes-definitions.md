---
title: "Aligning SaaS and Service Planes Definitions"
date: 2023-02-05
draft: false
---

Martin Fowler writes the following about *Evocative Names* in his [Writing Software Patterns](https://martinfowler.com/articles/writingPatterns.html) article:
> One of the valuable features of patterns work is that it develops a vocabulary with which we can talk about how to do things.

Having a clear understanding of what names mean helps communicate intent. When building SaaS applications on AWS, you might stumble into the following terminology:
* SaaS control plane and SaaS application plane
* Service control plane and Service data plane

In this blog post, I describe the definitions of these terms and explain how they align.

## Definitions

### SaaS control plane and SaaS application plane

The [SaaS Architecture Fundamentals](https://docs.aws.amazon.com/whitepapers/latest/saas-architecture-fundamentals/saas-architecture-fundamentals.html) whitepaper suggests organizing the SaaS environment into two distinct planes: control plane and application plane. The [Control plane vs. application plane](https://docs.aws.amazon.com/whitepapers/latest/saas-architecture-fundamentals/control-plane-vs.-application-plane.html) section in the whitepaper defines the SaaS control plane and the SaaS application plane:
* **SaaS control plane** includes services that give you the ability to manage and operate your tenants through a single, unified experience
* **SaaS application plane** includes the services that represent the tenant experience/application for your solution

{{< figure src="/images/aligning-saas-and-service-planes-definitions-1.png" title="Source: https://docs.aws.amazon.com/whitepapers/latest/saas-architecture-fundamentals/control-plane-vs.-application-plane.html" >}}

It is important to note that control plane and application plane describe a logical division of the SaaS environment. It does not prescribe the deployment layout. For example, you could deploy control plane and application plane services in different AWS accounts, or in the same AWS account.

So far, we focused on the SaaS definitions of control plane and application plane. You could also hear AWS using another "planes" terminology: Service control plane and Service data plane. 

### Service control plane and Service data plane

The [Avoiding overload in distributed systems by putting the smaller service in control](https://aws.amazon.com/builders-library/avoiding-overload-in-distributed-systems-by-putting-the-smaller-service-in-control/) article in the Amazon Builders' Library defines the Service control plane and the Service data plane:
* **Service control plane** is responsible for managing and vending customer configuration
* **Service data plane** is responsible for executing customer requests

{{< figure src="/images/aligning-saas-and-service-planes-definitions-2.png" title="Source: https://brooker.co.za/blog/2019/03/17/control.html" >}}

Let's look at [Snowflake](https://www.snowflake.com/)'s [architecture](https://docs.snowflake.com/en/user-guide/intro-key-concepts.html) as an example. It consists of three key layers, including Cloud Services and Query Processing.

{{< figure src="/images/aligning-saas-and-service-planes-definitions-3.png" title="Source: https://docs.snowflake.com/en/user-guide/intro-key-concepts.html" >}}

Cloud Services tie together all of the different components of Snowflake in order to process user requests, from login to query dispatch. This is the Service control plane. Query Processing is responsible for query execution. This is the Service data plane.

[Stripe](https://stripe.com/) is another example. Let's look at its billing functionality. If you want to charge customers based on how much of your service they use, Stripe offers support for [usage-based billing](https://stripe.com/docs/billing/subscriptions/usage-based). Setting up usage-based billing involves actions such as customer sign-up and usage reporting. Customer sign-up is a Service control plane operation - managing and vending customer configuration. Usage reporting is a Service data plane operation - execution of customer requests (metering consumption).

## Aligning SaaS and Service planes definitions

Snowflake customers interact with Cloud Services and Query Processing functionality. Hence, I would organize them into the SaaS application plane. SaaS providers integrate their products with Stripe to provide billing functionality. The billing functionality is an operational component of the SaaS offering, hence I would organize it into the SaaS control plane.

Based on the above, you can see both SaaS control plane and SaaS application plane services may have Service control plane and Service data plane components, as shown in the diagram below. Hence, I suggest to think of SaaS control plane and SaaS application plane as containers for Service control plane and Service application plane.

> SaaS control plane and SaaS application plane are containers for Service control plane and Service application plane

*Note:* Some functionality may span SaaS control plane and SaaS application plane. One example is Authentication service. Authentication control plane would be part of SaaS control plane - e.g. configure authentication requirements. Authentication data plane would be part of SaaS application plane - e.g. perform JWT validation.

{{< figure src="/images/aligning-saas-and-service-planes-definitions-4.png" title="SaaS and Service planes example" >}}

## Conclusion

SaaS and Service planes have both distinct and common concepts. In this blog post, I described the components of each plane, their definitions, and an architectural model for how they align. I hope that will provide you with a vocabulary and a pattern to use when designing your SaaS applications.