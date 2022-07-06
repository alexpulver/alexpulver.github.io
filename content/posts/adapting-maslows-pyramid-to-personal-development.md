---
title: "Adapting Maslow's Pyramid to Personal Development"
date: 2020-05-23
---

The software engineering world is constantly increasing in breadth and depth. There is so much to learn, and it is very challenging to prioritize what and when. Recently, I looked at [Maslow's hierarchy of needs](https://en.wikipedia.org/wiki/Maslow%27s_hierarchy_of_needs) in its pyramid form, and then an idea stroke me - personal development might be very much represented in a similar manner. The missing link for me was adding time sensitivity to the needs (i.e. priorities). One famous example of this approach is a [TestPyramid](https://martinfowler.com/bliki/TestPyramid.html):

{{< figure src="/images/adapting-maslows-pyramid-to-personal-development-1.png" width="50%" >}}

The idea is that you should have many more unit tests than UI tests, partly because they take less time to write and maintain. This got me thinking - there is a wide spectrum of topics, some are quick to go through and some can take days or weeks. As with tests, the knowledge portfolio should have a certain balance. After scribbling on paper (there is still something great to paper and pencil for that purpose :blush:), you can see the digital version of the `PersonalDevelopmentPyramid` I came up with below:

![](/images/adapting-maslows-pyramid-to-personal-development-2.png)

I am a firm believer that learning should be both theoretical and practical. It is not enough to "just get to code", because you will probably be producing a less maintainable system that will have a higher total cost of ownership (TCO) down the road [[1](https://medium.com/@ryancohane/financial-cost-of-software-bugs-51b4d193f107)]. On the other hand, reading all whitepapers and books will not teach you all the nitty-gritty details in the ever-evolving software tools, frameworks, and libraries unless you use them. You can see that I have a mix of theoretical and practical activities in the pyramid to align with that approach. Now I will discuss each layer more in-depth.

**Recent Announcements**

Whatever domain you are working in, there are certainly some new releases or updates happening every day. Find some good feed that provides a summary of these, and read it every morning. That will help you to keep on top of new developments in your area, bearing the cost of a few minutes.

**Reading Blogs**

I like posts that walk you through a task or use case with implementation details included. This lets you "feel" how things are working without spending a significant amount of time to do it yourself. Some topics worth spending the additional time and get your hands dirty - for example when some details are omitted or not described in detail enough. Most of the posts take anywhere between 5 to 30 minutes to read, so you can read one or two per day.

---

_**Your 20% Day**_

I put the following three layers in the "*Your 20% Day*" group. Before diving into them, I will try to explain why first. Personally, I need uninterrupted "in the zone" time to learn topics in depth. Usually it means two hours or more. All of the following activities require (in my opinion) at least that amount of time to be done effectively. Some of them can be broken down into chunks, but each chunk should still be of that minimum size. Hence, I try to (with various degrees of success) pick a day or two half-days per week to focus explicitly on these activities. Preferably, there should be a rotation and a mix of theoretical and practical activities - for example, work on a project and then read an article. With that (important) interlude, let's go back to the next layers.

---

**Article/Video or Proof of Concept**

Articles, such as [Avoiding insurmountable queue backlogs](https://aws.amazon.com/builders-library/avoiding-insurmountable-queue-backlogs/), go one step further and often offer a broader view on a topic. This is important because as engineers, we should not only learn *how* to do something but also what are the different considerations, approaches, and trade-offs involved. Videos complement the articles because often speakers add examples or details that are usually not captured in writing. For the hands-on part, I try to run small proof of concepts (POCs) that are good enough to solidify understanding of the topic but do not unfold into longer projects.

**Whitepaper or Workshop**

Now we are getting to the "heavyweight" activities. These can take anywhere between one to two days (including additional research). This is the place to start getting picky - for example, choose ones that align with your area of depth or interest. Whitepaper gives a much deeper view into a domain as compared to articles/videos. For example, see [Machine Learning Lens](https://aws.amazon.com/blogs/architecture/introducing-the-well-architected-framework-for-machine-learning/), which aims to help you design your machine learning (ML) workloads following cloud best practices. Workshops allow you to uncover the nitty-gritty details I mentioned earlier in this post, and potentially bring up additional questions to dig further into. For example, see [AWS App Mesh Workshop](https://www.appmeshworkshop.com/) - it is only when going through every step and deployment, the different considerations for choosing service discovery method revealed themselves to me.

**Course or Project**

Everything we covered until now provides us with "building blocks" of knowledge. A course or project takes these further and lets you enrich your knowledge with an end-to-end story. It can be a course in design patterns, development of an open-source library, public speaking, and more.

**Book**

Even after you built the "end-to-end" knowledge platform, it is still rather specific. Books allow you to take a birds-eye view (while sometimes flying very close to the ground) and cut through different areas, potentially uncovering new ideas and concepts. You should have at least one book to read and progress through (almost) every day.

---

**Conclusion**

Acquiring knowledge is hard (I would even dare to say it is [NP-hard](https://en.wikipedia.org/wiki/NP-hardness)). The above framework served me quite well for some time, so I hope it can be of use to others. Of course there are times when I deviate from the pyramid, but it serves me as a guiding principle I try to follow. Like Winston S. Churchill said, *"Success is stumbling from failure to failure with no loss of enthusiasm."*