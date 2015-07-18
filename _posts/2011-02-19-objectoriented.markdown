---
layout: post
title: 19 1/2 Things to Make You a Better Object Oriented Programmer
tags:
- Object Oriented Design
---

#Introduction

Last year, OreDev organized a developer Conference in Malmö Sweden. The theme of the conference was called "get real", it was about how to stay in balance, between today’s realities and tomorrow’s possibilities while the universe is in constant motion. One of the more interesting session for me was a session by Grey Young called [19 1/2 Things to Make You a Better Object Oriented Programmer](http://vimeo.com/17151526). In this talk he discusses several things to improve your object-oriented programming skills and more importantly give you explanations on the thought processes behind the ideas.

Greg Young for people who do not know him is an independent consultant who lives in two suitcases (literally). When not travelling around working for clients throughout the world you can often find him on the domain driven design list, blogging at [codebetter.com](http://codebetter.com/), or floating upside down in a kayak through rapids.
	
During the talks he mentions the followings things which will make you a better regarding object-oriented programmer
	
#1. Class != Object

Explicitly making the distinction between classes and objects is important. A class is nothing more than a template where objects are instances of these templates. These instances do not necessarily look like the class. In modern object-oriented programming books, not much attention is given about the difference between classes and objects.

#2. Method Call = Message

Calling a method on an object is the same as sending a message to an object. Many of the earlier object programming books use sending a message and calling a method interchangeable. When you look at it as sending a message interesting possibilities are possible. For example filtering the messages when they go through the pipeline. This for example would enable setting a trap to catch specific messages when you are testing an object. This will remove the need for mock frameworks.
	
#3. Objects are not "data"
	
Objects are data and behavior put together. Many domain models consists only of objects with getters and setters without any behavior in any of them. On top of these objects sit a layer of services that interact with these objects. This describes procedural code and the result is called an [Anemonic Domain Model](http://martinfowler.com/bliki/AnemicDomainModel.html). In his talk Greg discusses if this is a pattern or an anti-pattern. The conclusion was that if you intent to create an object oriented domain model and the result is an anemonic domain model it is an anti-pattern. If on the other end you choose for this anemonic domain model because the development team has little experience with object-oriented programming it is a pattern.
	
#4. Is a?

An “Is a” relationship between two classes is the strongest possible coupling you can create in C#. When you create an inheritance tree with multiple levels, maintenance becomes a nightmare especially if you introduce a new relationship. For example, having an inheritance tree with in it a plant and an animal and adding a virus to it which is both a plant and an animal. A better alternative for a “Is a“ relationship is an interface implementation. With an interface you only couple the behavior instead of the data and the behavior.

#5. Dependencies should point in

The difference between procedural code and object-oriented code is that with procedural code dependencies points outwards and with object-oriented code the dependencies point in. For example the following procedural code needs a reference to System.Console.

{% highlight csharp %}
void WriteMessage(string message)
{
  Console.WriteLine(message);
}
{% endhighlight %}
 
If instead the constructor of a class would receive an ITextWriter interface which has a WriteMessage method like below there would not be a dependency to System.Console. The user of the MessageWriter class should create an implementation of the ITextWriter interface and therefore depends of the ITextwriter interface hence pointing the dependency in.
 
{% highlight csharp %}
public class MessageWriter
{
  private readonly ITextWriter textWriter;
 
  public MessageWriter(ITextWriter textWriter)
  {
    this.textWriter = textWriter;
  }
	
  void WriteMessage(string message)
  {
    Console.WriteLine(message);
  }
}
 
internal interface ITextWriter
{
  void WriteMessage(string message);
}
{% endhighlight %}

According to Greg this is the core concept of OO programming. This inversion of dependencies could also be done with messages for example use events to invert the dependencies. In system integration this is one of the reasons for choosing a push model instead of a pull model. You invert the dependencies between the systems.

#6. Constructors are special

Constructors are special methods that create our objects. The dependencies of an object that are set through the constructor should always be read-only. See the example below in which the textReader and textWriter dependencies of the MessageWriter class are both read-only and therefore cannot be changed once the instance of the class is created.

{% highlight csharp %}
public class MessageWriter
{
  private readonly ITextReader textReader;
  private readonly ITextWriter textWriter;
  
  public MessageWriter(ITextReader textReader, ITextWriter textWriter)
  {
    this.textReader = textReader;
	this.textWriter = textWriter;
  }
  
  .....
}

{% endhighlight %}

If you need to change the dependency of an object during the lifetime of that object you probably need a new instance of that object with a different dependency. If you really need to change the dependency during the life time of an object you would be better off injecting that dependency as a parameter of a method that needs that dependency. This according to Greg will solve many of the complexities during the development. See below for an example.

{% highlight csharp %}
void WriteMessage(ITextWriter textWriter, string message)
{
  textWriter.WriteLine(message);
}
{% endhighlight %}
	  
#7. Single Responsibility
	
The [Single Responsibility Principle](http://www.objectmentor.com/resources/articles/srp.pdf) is one of the SOLID principles and describes that there should be a single reason for a class to change. Greg discusses what exactly is a single reason or that one thing? If you go to far with single responsibility you end up with procedural code. All classes would contain only a single method. Therefore he describes that single responsibility should be a trade-off with cohesion. Cohesion is the amount that fields or the data of an object is used by the members of an object. 

For example a class that has perfect cohesion uses all its fields in all the methods.
	
#8. SOLID are heuristics to Ca, Ce and cohesion

The [SOLID Design Principles](http://www.semanticarchitecture.net/blog/solid/solid-software-architecture) are simply heuristics to [Afferent Coupling](http://en.wikipedia.org/wiki/Software_package_metrics) (Ca), [Efferent Coupling](http://en.wikipedia.org/wiki/Software_package_metrics) (Ce) and cohesion. The SOLID design principles such as SRP (Single Responsibility Principle) are simple teaching tools that are easier to understand than the underlying principles. If you want to become a better programmer you should strive to understand these underlying principles and concepts as they are more important. For example afferent couple describes the number of types from an external assembly that are using the types inside your assembly. Efferent coupling describes how many type you are using from an external assembly.

Stability is calculated by determining on how many concrete types you depend against the dependencies on abstract types. Depending on abstract types is better and increases stability. The table below shows several tools that can measure Ca, Ce, Cohesion and stability of your assembly. The advise is to use these tool to learn about the underlying principles and concepts of the SOLID design principles such as Ca, Ce, cohesion and stability.

|**Tool**|**Platform**|**Open Source**|
|[NDepends](http://www.ndepend.com/)|Microsoft.Net|No|
|[XDepend](http://www.xdepend.com/)|Java|No|
|[Sonar](http://www.sonarsource.org/downloads/)|Java|Yes|
|[Sonar C# Plugin](http://docs.codehaus.org/display/SONAR/.Net+plugin)|Microsoft.Net|Yes|
|[Lattix](http://www.lattix.com/)|Microsoft.Net / Java / C++ |No|

#9. Liskov is Special
	
The [Liskov substitution principle](http://www.objectmentor.com/resources/articles/lsp.pdf) is one of the SOLID design principles. Liskov is special as this is one of the most important concepts that you need to master. Liskov is important because the human brain is only capable of handling so many things at once. For example it is impossible to get an understanding of a system in which 50 or more objects interact with each other. Liskov enables you to break apart your object graph. You define separate subsystem that coupled using interface. Because the systems that are coupled with an interface are interchangeable they become better to understand.
	
#10. MVC is not a UI pattern

Although controversial, [MVC](http://nl.wikipedia.org/wiki/Model-view-controller-model) is not a UI pattern, it is an architectural pattern that has been proven to be useful in a number of situations, including with a user interface.

#11. Value objects are important
	
[Value objects](http://domaindrivendesign.org/node/135) are important because they make our code clearer and more easily to understand. For example, Value objects such as money and a time span make code easier to read. Another thing that is important is that objects should always have a valid state. When objects always have a valid state there is no need to check all over in your code for the validity of your object. This will reduce the complexity of the source code.

#12. DRY our psychology pattern
	
Most programmers know [DRY](http://en.wikipedia.org/wiki/Don\'t_repeat_yourself) or Don\'t Repeat Yourself. This principle states that you should always strive to remove all duplication of code. Greg discusses that DRY has a trade-off with coupling. Say two classes A and B both have more or less the same method. By extracting this method in a separate class C and using this from both classes you create a coupling between classes A and B. This coupling did not exists with the duplication of the code. By using DRY we have increased the complexity of our code.

#13. Model "useful abstractions" of the reality
	
With this Greg tends to warn about trying to model the world in objects. Many developers are modeling objects because they exists in the real world not because the software needs them. Only model object that are useful for your software.

#14. Testing builds better objects, as does contracts
	
Testing or [TDD](http://en.wikipedia.org/wiki/Test-driven_development) forces us to build better objects. For example, objects with a lot of getters and setters are hard to test. The asserts of a test should be based on the behavior of an object not on its state. If you want to read a good book on object-oriented programming, read this one from [Bertrand Meijer](http://www.amazon.com/Object-Oriented-Software-Construction-Book-CD-ROM/dp/0136291554/ref=ntt_at_ep_dpi_1). Also using the new Code Contracts library in [version 4](http://msdn.microsoft.com/en-us/library/dd264808.aspx) of the Microsoft .Net framework will make you a better object-oriented programmer.

#15. Commands and Queries

All commands have a void return type but are allowed to change the state. Queries can have any return type but are not allowed to mutate state. By separating this concern the complexity of our applications will decrease.

#16. Objects play roles

Look at objects as a set of things that each play a separate role and work together to carry out a certain task instead of a single object that performs all the functionality of a task.
	
#17. Models the boundaries of transactions explicitly
	
You should explicitly model which objects of your domain model are part of which transactions. Define exactly where the boundaries of each transaction are.

#18. Pattern languages are about communication

A pattern language is a tool for easier communication. One should not force yourself to use a certain pattern or any other. They offer value in communication.

#19. Not all code is or should be OO, boundaries are often not

Not all the code should be OO, the boundaries of your domain model will often be programmed using procedural code. For example services will exchange data objects, objects that only holds data and no behavior.
	
#19,5 Don\'t take yourself too seriously

Well this one speaks for itself. We all make mistakes and through these mistakes we learn.

These are the things Greg Young talked in his [session](http://vimeo.com/17151526). All other sessions from [OreDev](http://oredev.org/2010) are also [availale](http://vimeo.com/user2649908).