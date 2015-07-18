---
layout: post
title: Dynamically Generated WCF Proxy
tags:
- WCF
---

#Introduction

Most programmers that start with Windows Communication Foundation (WCF) use the provided samples and tutorials from Microsoft. All these samples are based on static generated service references or proxies. When you want to connect from a client to a server, you create a reference from the client to the server project using the Visual Studio **Add Service Reference** function. Visual Studio then connects to the server, extracts the meta data from the server, and generates a proxy. The client uses this proxy to actually connect and call methods on the server.

Most programmers don't know that there is another way for WCF to generate this proxy. That is dynamically. This dynamic generation of a proxy has a lot of advantages.

#Background

As we all know, sometimes during development, an interface of a service changes. The same is possible for the configuration of a service; for example, the port number on which a service listens changes. Now, say that a developer changes the interface or the configuration of a service and somehow he or she forgets to change the client (this has happened to me more than once). With a statically generated proxy, you would notice the error late in the development cycle, probably during the first test.

This scenario has happened to me many times, and led me to thinking that there had to be another way. Wouldn't it be much better to tackle this problem during compilation on your local workstation or during a build on your build server? With dynamically generated proxy, this is possible.

#Structuring Your Assemblies

To be able to use the dynamic proxy and its advantages, you have to structure your assemblies or classes somewhat different. Both the client and the server reference and use a separate assembly that contains the interface and the configuration of the WCF service. The picture below shows the idea:

![Structure](../../../img/Divide_Assemblies.png)

The assembly called "Interface + Configuration" contains the WCF configuration of the server, including the interface itself. By structuring your assemblies this way, it is not possible to forget changing the client when the interface changes. It simply won't compile!

#Using the Code

The source code contains three assemblies, the server, the client, and the interface / configuration assembly. These are contained within a single Visual Studio 2008 solution. When you run the application, both the server and the client will start. The client sends a couple of messages to the server.

Generating the dynamic proxy using WCF is not difficult, the <code>ChannelFactory</code> from WCF makes it possible. The source code to generate a dynamic proxy is shown below.

#The Client

{% highlight csharp %}
EndpointAddress endpointAddress = 
	new EndpointAddress(Configuration.CreateServerUriString());
ChannelFactory<imessagesender> channelFactory =
  new ChannelFactory<imessagesender>(Configuration.CreateBinding(), endpointAddress);
IMessageSender messageSender = channelFactory.CreateChannel(endpointAddress);
messageSender.ShowMessage("Test message");
((IChannel)messageSender).Close();
{% endhighlight %}

<code>ChannelFactory</code> uses the interface <code>IMessageSender</code> to create a channel through the method <code>CreateChannel</code>. The interface <code>IMessageSender</code> and <code>Configuration</code> are both from the shared assembly.

The server side also uses this information from the <code> Configuration</code> class. It creates a new <code>ServiceHost</code> and listens on the <code>Uri</code> from the <code>Configuration</code>.

#The Server

{% highlight csharp %}
Server singleServerInstance = new Server();
Uri baseUri = new Uri(Configuration.CreateServerUriString());
ServiceHost serviceHost = new ServiceHost(singleServerInstance, baseUri);
serviceHost.AddServiceEndpoint(typeof(IMessageSender),
            Configuration.CreateBinding(), baseUri);
serviceHost.Open();
Console.ReadLine();
serviceHost.Close();
{% endhighlight %}

The method <code>Configuration.CreateBinding()</code> creates the binding. This method returns an instance of a specific binding which is configured in the <code>Configuration</code> class. If you change the binding of the server, for example, from HTTP to TCP, both the client and the server will automatically use it.

#Points of Interest

This solution is not exactly rocket science, but it has helped me and my company save time in the past. Which is the reason that I want to share this information with you. When someone forgets to change the client after an interface change, the solution won't compile. This way, it fails fast, which is a good thing. Another advantage is that you could stop publishing meta data of your service, this will increase the security of your service.

#Central Binding Configuration

Together with the configuration, the binding is also configured centrally in the configuration assembly. This enables you to configure the binding for the client as well as for the server. You could, for example, set the <code>MaxReceivedMessageSize</code> and the <code>ReceiveTimeout</code>, which should be set for the client as for the server. By centralizing the configuration, this can now be done in a single place instead of two or more configuration files. The code below shows an example of how to do this.

{% highlight csharp %}
public static class Configuration
{
  public static Binding CreateBinding()
  {
     NetTcpBinding binding = new NetTcpBinding();
     binding.MaxReceivedMessageSize = 25 * 1024 * 1024;
     binding.ReceiveTimeout = new TimeSpan(0, 2, 0);
     return new NetTcpBinding();
  }
}
{% endhighlight %}

#Further Extension of the Configuration

By standardizing the configuration of your services, it is possible to extend this even further. For example, you could create a standard base class or interface for all your configurations. This standardization makes it possible to extract information from your configuration, for example, documentation. This would enable automatic documentation creation from your configuration classes. The result, complete up to date documentation of all your services. A good way to start is to define an interface with methods for all the attributes to document. See the code below for an example.

{% highlight csharp %}
public interface IWcfConfiguration
{
   string RetrieveUriString();
   int RetrievePortNumber();
   TimeSpan RetrieveTimeOut();
}
{% endhighlight %}

#Using Callbacks

Callbacks are possible using this solution. When using callbacks, the <code>ChannelFactory</code> used in the client should be changed to a <code>DuplexChannelFactory</code>. The constructor of a <code>DuplexChannelFactory </code>expects an instance where the callback method(s) can be called by the server. Also the client should call the subscribe and unsubscribe methods to add the callback method. See the code below for an example. The provided source code in the download includes an example using callbacks. </p>

{% highlight csharp %}
DuplexChannelFactory<imessagesenderwithcallback> duplexChannelFactory = 
	new DuplexChannelFactory<imessagesenderwithcallback>(this, binding, endpointAddress);
IMessageSenderWithCallBack duplexMessageSender = 
	duplexChannelFactory.CreateChannel(endpointAddress);
duplexMessageSender.Subscribe();
duplexMessageSender.ShowMessage("message");
duplexMessageSender.Unsubscribe();
{% endhighlight %}

#Disadvantages

This solution has its disadvantages. It works only if you control both sides of the wire. If you publish a service for usage outside your company, it's a different scenario. You could publish the interface and configuration assembly, but I think that enabling meta-data on your service would be easier for such a situation.</p>