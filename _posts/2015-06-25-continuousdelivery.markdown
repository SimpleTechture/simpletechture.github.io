---
layout: post
title: Creating a Continuous Delivery Platform
tags:
- Continuous Delivery
---

#Fast innovation

![Fast innovation](../../../img/Innovation.jpg)   
Today, companies only survive if they innovate fast enough to create more value than their competitors. Getting to know the needs of the client and delivering it as soon as possible is essential. This means going from a concept to cash in weeks, instead of months or even longer. An agile development method that is highly standardized and with a high degree of automation is vital.

##But what does this mean? A higly standardized and automated environment?

Does this mean that you don't need project managers, developers and testers anymore because they are replace by assembly lines?!  Well, maybe if possible...... but I don't think so. But, what I **do** believe is that companies could really improve the way they execute projects by automating and standardization. 

##What type of automation and standardization are we talking about? That during the start of a project you don't have to:

- Discuss the type of tools to use for a new project 
- Discuss how the development process of a new project should look like
- Discuss the design and development guidelines
- Manually deliver software to our customers

#Continuous Delivery Platform

A Continuous Delivery Platform will unify all previously discussed points. The name Continuous Delivery is not used by accident. [Martin Fowler](http://martinfowler.com/bliki/ContinuousDelivery.html) describes Continuous Delivery as being **a software development discipline where you build software in such a way that the software can be released to production at any time.**. An  Continuous Delivery Platform will actually enable you to deliver software that solves the problem of your customer and can be deployed to production at any time.

#Why is this important?

*Why do you want to continuously delivery software to production?*

The three primary reasons for implementing continuous delivery are:

1. Enable your clients to earn money faster
2. Getting faster feedback from the customer of our client
3. Increase the quality of our solutions

##Enable our clients to earn money faster

As [Joel Spolsky](http://www.joelonsoftware.com/items/2012/01/06.html) describes it **A feature that you built and tested, but didn’t deliver yet because you’re waiting for the next major release, becomes inventory**. Inventory is dead weight: money you spent that’s just wasting away without earning you anything. Sure, 100 years ago, we had these things called “CD-ROMs” and we shipped software that way, so there was an economic reason to bunch up features before we inflict ‘em on the world. But there’s no reason to work that way any more.".

By automating the delivery process, decreasing the size of your deployment and increasing the frequency of deployment we are able to [decrease the cost of a deployment](http://www.alwaysagileconsulting.com/release-more-with-less/)

##Getting Faster Feedback

Speed is the essential component here. Seems logical, but I think speed can be linked to something more important, getting fast feedback! Getting fast feedback is important because it enables our customer to make the correct decisions. If I am talking about fast feedback I mean fast feedback in every aspect of your development process, for example:

- Fast feedback by getting manual feedback from the customers
- Fast feedback by getting automatic feedback from the customer (A/B testing)
- Fast feedback by performing manual acceptance testing by the client
- Fast feedback by performing manual tests
- Fast feedback by running automated UI tests using Selenium
- Fast feedback by deploying the completed feature automatically on the test server (Does it run?)
- Fast feedback by compiling and running tests on a build server, does it integrate?
- Fast feedback by compiling and testing sources locally on a developers workstation (TDD anyone?)

##Increase the quality of our solutions

If we increase the frequency and decrease the size of our deployments we get the additional benefit of a lower failure rate of new releases, shortened lead time between fixes, and faster time to recovery in the event failures. But also the quality of our solutions will increase because we are standardizing our deployments. To standardize our deployments we have to better think about configuration management, automatic database migrations and the architecture of our solution.</span>

#What do we need to make this happen?

To be able to implement continuous delivery we need two things:

- A delivery process in which all parts are completely automated. **A Deployment Pipeline**.
- A culture that enables a collaborative working relationship between everyone involved in the delivery process. **A DevOps culture**.

##Deployment Pipeline

A deployment pipeline is a pipeline which is build up from all the parts that are necessary in the software development process. It runs from left to right. On the left features are being checked-in to the source code repository and on the right features are being deployed and released to production.

![Deployment Pipeline](../../../img/cd-pipeline.png)

##DevOps Culture

DevOps is a combination of Development and Operations. To be able to implement continuous delivery you need close collaboration between developers, operations, testers and anyone else needed to get software into production.

#How to start?

This all looks nice and sounds good but how do you start with implementing this? I think that first of all this means a mind shift in how companies are thinking about projects and make Continuous Delivery a first class citizen of our projects. Every one working in a company in the role of management, developers, project managers, testers and sales should believe and communicate that this is how they deliver projects. Single click deployment should be the norm, not the exception.

![Mind Shift](../../../img/mind_shift.png)

Besides this there should be a focus on our core business of delivering solutions to our clients problems instead of being busy with things which are going to be **commoditized and offered as a service**. For example, small to medium sized companies shouldn't be hosting our own source control server (Git, SVN, or Microsoft TFS Server). Installing, configuring, maintaining or upgrading of source control servers is not their core business. The same goes for many of the other IT services companies are using for development. Most of these services are also offered by external services through a SAS or PAS offering. 

#First Step

A first step a company could take is take a look at the deployment pipeline and start with integrating and automating the first parts of the pipeline. For example by changing your source code repository to a hosted solution such as [GitHub](https://github.com/) or [TFS Online](https://www.visualstudio.com).
