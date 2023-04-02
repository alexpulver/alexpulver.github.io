---
title: "Methodology for Organizing Application Components into Repositories and Pipelines"
date: 2023-03-31T13:57:23+03:00
draft: true
---

Building applications involves trade-offs around reliability and operational excellence amongst other pillars of AWS Well-Architected Framework. Breaking down an application into smaller components can help reduce blast radius and improve delivery performance. 

In this blog post, I will introduce the recommended methodology for organizing application components into repositories and pipelines. It builds upon the experience embodied in the [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html), and the way Amazon deploys software as described in [Automating safe, hands-off deployments](https://aws.amazon.com/builders-library/automating-safe-hands-off-deployments/) article. I will 1/ provide an overview of the related AWS Well-Architected Framework definitions 2/ introduce the methodology by breaking down an example application into components 3/ discuss trade-offs in splitting source repositories and pipelines.

The AWS Well-Architected Framework uses the following terminology:

* A **component** is the code, configuration, and AWS Resources that together deliver against a requirement. A component is often the unit of technical ownership, and is decoupled from other components.
* The term **workload** is used to identify a set of components that together deliver business value. A workload is usually the level of detail that business and technology leaders communicate about.
* The **architecture** is how components work together in a workload. How components communicate and interact is often the focus of architecture diagrams.

I’ll use [DocumentsApp](https://github.com/alexpulver/adf/blob/main/examples/documentsapp/README.md) as example. The Trivia Game is a workload, because it delivers a certain business value to its customers. It is composed of multiple components as you will see later in the blog post. As mentioned above, a component is often the unit of technical ownership, and is decoupled from other components. 

[Image: Image.jpg][Image: Workload.drawio.png]

I recommend for each component to have its own version control repository and a deployment pipeline. As mentioned in the [Automating safe, hands-off deployments](https://aws.amazon.com/builders-library/automating-safe-hands-off-deployments/#Source_and_build) article, having a separate pipeline for each component helps deploy changes to production faster. Backend component code changes that fail integration tests and block the backend pipeline don’t affect other pipelines. For example, they don’t block frontend code changes from reaching production in the frontend pipeline. All the pipelines for the same workload tend to look very similar. For example, a configuration pipeline uses the same safe deployment techniques as the backend code pipeline, because a bad configuration change can have an impact on production just as a bad backend code change can.

*TODO:* Add description of trade-offs between 1/ repository per workload and pipeline per component 2/ repository per workload and a pipeline per workload 3/ repository per component and a pipeline per component.

Backend component consists of the following logical units: API, database, and monitoring. 

*TODO:* Add details and code examples.
[Image: Image.jpg]

Conclusion

This blog post brings together multiple aspects of developing, deploying and operating a workload. I described a methodology for organizing application components into repositories and pipelines. The methodology includes trade-offs and what it would take to address the shortcomings of each approach. I also mentioned the advantages of each component having its own version control repository and a deployment pipeline. I hope it should help you do build applications at scale, while reducing blast radius and increasing time to value.
