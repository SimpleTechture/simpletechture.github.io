---
layout: post
title: Improve your Software Architecture
tags:
- Software Architecture
---

Use the following 10 tips to improve your **Software Architecture Design**. I use these 10 tips or guidelines daily and they have helped me creating high quality Software Architectures. Describing your software architecture design is useful for any type of project, it will share the design of the system among your stakeholder.

#1. Based on requirements

![Requirements](../../../img/Requirements1.jpg)

You should base your software architecture design on the requirements of your stakeholders. An architecture focuses on **non-functional requirements**. I see many software architecture designs based on purely technical motives. Each part of your design should be based on business requirements. You as an architect should translate these requirements into the **architectural design decisions**. If the stakeholder values **maintainability**, you could use the **layer pattern** to separate several parts of the application. If performance is important, maybe layering is not a good solution. An exhaustive list of non-functional requirements can be found at [ISO 9126](http://en.wikipedia.org/wiki/ISO/IEC_9126) and at [QUINT](http://www.serc.nl/quint-book/). If you do not use Non-functional requirements in your organization but want to introduce them, take a look at a [presentation](http://www.slideshare.net/kalkie/letsgrow-nonfunctional-requirements) I did previously.

From the Non-functional requirements or quality attributes you have to create the right design. While you could create this from scratch there are many examples in the form of design patterns or architectural patterns. A design or architectural pattern expresses a relation between a problem and a solution. Although we often think that our problem is unique this is often not the case. If you take a step back you will see that many of our problems already have been solved using existing patterns. Two books that I can recommend are *[Pattern-Oriented Software Architecture](http://www.amazon.com/Pattern-Oriented-Software-Architecture-System-Patterns/dp/0471958697)* and *[Design Patterns](http://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)*. Both books contain a catalog full of patterns. Each pattern describes the problem it solves and in which context it can be used. There are also many online pattern source on the web such as [this one on Wikipedia]("http://en.wikipedia.org/wiki/Architectural_pattern_(computer_science)) and [this from The OpenGROUP](http://www.opengroup.org/architecture/togaf8-doc/arch/chap28.html)

#2. Rationale, rationale, rationale

![Rationale](../../../img/Rationale1.jpg)

The most important aspect of your architecture description is the recording of your **rationale** behind design decisions. It is important for a reader of the architecture description to understand the reason why you made a specific decision. Make your assumptions explicit and add them to the description. Assumptions may be invalid now or later but at least it will be clear how you came to that decision. It make communicating with your team much easier if you share you rationale.

Recording your rationale becomes much easier if your non-functional requirements are explicit. It will be much clearer if you describe that you created several components to increase the **testability** because testability is the an important requirements. Do describe the **Why** and **How** in your** software architecture design**

#3. Don’t Repeat Yourself (DRY)

![Repetition](../../../img/Repetition.jpg)

**Don’t Repeat Yourself (DRY)** or **Duplication Is Evil (DIE)** come from software-engineering in general. The DRY principle is stated as “Every piece of knowledge must have a **single**, unambiguous, authoritative representation within a system”. You can apply this principle on many levels; Architecture, Design, Testing, Source Code and Data. For me this is one of the most difficult things to uphold. You have to fight the repetition because it will slow you and your project down. The difficult part of this Repetition Creep as I call it is that it is introduced very slowly. The first repetition won’t hurt you directly, it will even gain some time. You are able to release the first version of the application somewhat quicker, but as I found it always shows up later and makes something else more difficult. At that moment you regret the decision to introduce repetition.

If you absolutely must add another copy of information make sure that you automatically generate that copy of the information. It will make your live so much easier in the future. One thing that helps to fight repetition is to store the data where it belongs. This seems logical and is the basis of **object oriented design** but I often see this violated with regards to system architectures. For example take packaging an application for deployment. The process in which you filter the build of your software to include the components that are necessary in a package. Where would you store the information which component should be included in the package? You could create a list that includes the names of the components that should be packaged. That means you introduce your first repetition. You now have two places where component names are mentioned. A better solution would be to add that information to the component itself. When the first list in any format shows up in or around an application, **alarm bells** should sound and you should be on the lookout for repetition

#4. Slicing the cake

![Slicing the cake](../../../img/Slicingthecake.png)

I struggled with naming this, but found **Slicing the cake** as it is called in Agile development the best description. By slicing the cake I mean that you design your architecture iterative in vertical slices. An architect implements or prototypes each vertical slice to confirm if it actually works. You should do this because architectures cannot be created on paper. It does not mean that you cannot use horizontal layering or any other pattern in your architecture. In the case of layering the horizontal layers are smaller.

Say you use layering in your architecture design because your stakeholders expect that the components that you develop for this system will be used in other systems as well. During the first iteration you design a small part of the User Interface (UI), a small part of the Business Layer (BL) and a small part of the Data Layer (DL). You make sure that this works as expected by proving it with a prototype or by actually implementing it. In the second iteration you add new functionality and expand each layer horizontally with the needed functionality. The existing UI, BL and DL are combined with the new UI, BL and DL to form the new layers.
	
The difficulty with slicing is how to slice the cake so that the next slice will properly align with the previous.

#5. Prototype
When creating a software architecture design make sure that you prototype your design. Validate your assumptions, do that performance test and make sure that the security architecture is valid. A prototype will give you the opportunity to fail fast which is a good thing.

#6. Quantify

![Measure](../../../img/Measure1.jpg)

This principle extends the first principle “Based on Requirements”. To be able to create a proper software architecture design you need to quantify your Non-functional requirements. It should be “fast” cannot be a requirement neither is maintainable or testable. How will you know if you have met these requirements?

[ISO 9126](http://en.wikipedia.org/wiki/ISO/IEC_9126) and [QUINT](http://www.serc.nl/quint-book/) both describe ways to quantify the non-functional requirements. For example testability specifies an indicator “number of test cases per unit volume”. QUINT also specifies how you can actually measure an indicator for example the indicator “Ratio Reused Parts” from the quality attribute [Reusability](http://www.serc.nl/reusability.htm) which you can measure using the following protocol:

1. Measure the size of each reused part;
2. Measure the size of the entire software product;
3. Calculate the ratio of reused parts, which is the sum of reused parts divided by (2).

#7. Get it working, Get it right, Get it optimized

In many projects I have seen architects and developers design software architectures that focus on creating general purpose libraries, services or infrastructure. These are created without a direct reference to a concrete application. Instead they are designing for tomorrow. This for me is like walking backwards, it is very difficult to design generality up-front. Today’s businesses change too fast to design for generality up-front.

You should always start with a concrete implementation for a specific problem. At the time you start working on the next application and find similarities, that’s the time to think about generalizing. This makes the first solution simpler, which should be your design goal.

#8. Focus on the Boundaries and Interfaces

When creating your software architecture design you should focus on the boundaries of your system and components. When starting blank you should think about separation of concerns. What component or system has which responsibility? Between the components or system design explicit interfaces. Don’t separate a system of component when a lot of communication is necessary between these components or systems.

#9. The Perfect is the enemy of the Good

![Perfection](../../../img/Perfection1.jpg)

The phrase “The perfect is the enemy of the good” from [Voltaire](http://en.wikiquote.org/wiki/Voltaire) is also valid for software architecture design. How many times have you started a new project and thought I want this project to be perfect? And how many times have you actually found out that the project wasn’t perfect. Well, I think a project will never be perfect. There will always be problems or forgotten requirements.

Perfection is never possible. However you are able to create a good software architecture design. Do not try to analyze everything during the start of the project it will slow you down. Watch out for [Analysis Paralysis](http://en.wikipedia.org/wiki/Analysis_paralysis) 

#10. Align with your stakeholders

![](../../../img/Stakeholders.jpg)

Before you can create any type of system you need to identify your **stakeholders**. Each stakeholder has different needs of your software architecture and may require a different view. Software developers may need descriptions using **Unified Modeling Language (UML)** while business sponsors need a description in natural language. Operations and support staff for example may need other view such as context diagrams. There is a tension between creating all these views for stakeholders and principle **3. Don’t Repeat Yourself**. Each view essentially describe the same system and adds repetition. Therefore you should only add those descriptions that adds value for a specific stakeholder.

Well there you have it, my 10 tips to improve your **Software Architecture Design**. If you have another tip that you use to improve your architecture design, let me know!