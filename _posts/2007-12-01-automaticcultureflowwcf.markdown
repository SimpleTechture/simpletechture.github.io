---
layout: post
title: Automatically Culture Flow using WCF
tags:
- WCF
- Culture
---

#Introduction

On a project I am working on, we use WCF to communicate from an internet client to a Windows service. WCF automatically lets the windows identity flow from the client to the server. This is great but it is not the case for the culture of the client. If the culture would also flow from the client to the server reading localised data from, for example, the resource file could be completely transparent for the application. This example demonstrates how you could implement such a solution by implementing a custom WCF <code>MessageInspector</code>.

![Screenshot](../../../img/CultureServer.png)

#Background

The given extension should only be used if you are controlling both ends of the wire (client and server). If you are only creating the server and care about interop you are better off using a more standard method like the one described in <a href="WSI18N.asp">Globalization Patterns in WCF (WS-I18N implementation)</a> by [Pablo Cibraro](http://www.codeproject.com/script/Articles/list_articles).

WCF has several extension points; in this example a custom <code>MessageInspector </code>is used. A custom <code>MessageInspector </code>can be used to inspect or modify messages as they pass through a WCF client or server object. To inspect or modify messages as they pass through a client, you should implement the <code>IClientMessageInspector </code>interface. To inspect or modify messages as they pass through a server, you should implement the <code>IDispatchMessageInspector </code>interface. The custom <code>MessageInspector </code>in this example implements both interfaces.

The custom <code>MessageInspector </code>adds a custom Message Header to the request at the client which contains the culture info of the thread running the client. At the server side, the custom <code>MessageInspector </code>retrieves the culture info from the custom Message Header and sets the culture info on the running thread.

To use the custom message inspector, a custom behaviour also needs to be created which implements <code>IEndpointBehavior</code>. This behaviour is then added to the behaviours collection of the WCF endpoint.

#Using the Code

The solution is divided into four projects: the server, the client, the extension and the service interface. The interesting part is the extension project. In the example, the client sets the culture info of the current thread and calls a simple "hello world" method on the server. The server responds with a <code>Console.Writeline </code>with the culture of the running thread.

#MessageInspector

This is the custom Message Inspector where the <code>BeforeSendRequest </code>at the client adds the message header and the <code>AfterReceiveRequest </code>at the server extracts the custom header and sets the culture of the currently executing thread.

{% highlight csharp %}
public class CultureMessageInspector : 
    IClientMessageInspector, IDispatchMessageInspector
{
  private const string HeaderKey = "culture";

  public object BeforeSendRequest
        (ref System.ServiceModel.Channels.Message request,
        System.ServiceModel.IClientChannel channel)
  {
    request.Headers.Add(MessageHeader.CreateHeader
        (HeaderKey, string.Empty, Thread.CurrentThread.CurrentUICulture.Name));
    return null;
  }

  public object AfterReceiveRequest
        (ref System.ServiceModel.Channels.Message request, 
        System.ServiceModel.IClientChannel channel,
        System.ServiceModel.InstanceContext instanceContext)
  {
    int headerIndex = request.Headers.FindHeader(HeaderKey, string.Empty);
    if (headerIndex != -1)
    {
      Thread.CurrentThread.CurrentUICulture = 
            new CultureInfo(request.Headers.GetHeader<string>(headerIndex));
    }
    return null;
  }
  ...
{% endhighlight %}

#Custom Behaviour

The custom behaviour is created like this:

{% highlight csharp %}
public class CultureBehaviour : IEndpointBehavior
{
  public void ApplyClientBehavior(ServiceEndpoint endpoint, 
        System.ServiceModel.Dispatcher.ClientRuntime clientRuntime)
  {
    clientRuntime.MessageInspectors.Add(new CultureMessageInspector());
  }

  public void ApplyDispatchBehavior(ServiceEndpoint endpoint, 
        System.ServiceModel.Dispatcher.EndpointDispatcher endpointDispatcher)
  {
    endpointDispatcher.DispatchRuntime.MessageInspectors.Add
                    (new CultureMessageInspector());
  }
  ...
{% endhighlight %}

#Creating the Service Host

When creating the <code>ServiceHost</code>, the custom behaviour is added to the behaviours collection of the <code>ServiceEndPoint</code>. This could also be configured in the configuration file.

{% highlight csharp %}
Server server = new Server();
ServiceHost host = 
    new ServiceHost(server, new Uri("net.tcp://localhost:8080"));
NetTcpBinding binding = new NetTcpBinding();
ServiceEndpoint endPoint = 
    host.AddServiceEndpoint(typeof(IHelloWorld), binding, "Server");
endPoint.Behaviors.Add(new CultureBehaviour());
{% endhighlight %}

#Creating the Client Channel

When creating the client channel, the custom behaviour is also added to the behaviours collection of the <code>ServiceEndPoint</code>. This could also be configured in the configuration file.

{% highlight csharp %}
ServiceEndpoint tcpEndPoint = new ServiceEndpoint(
    ContractDescription.GetContract(typeof(IHelloWorld)), 
        new NetTcpBinding(), new EndpointAddress
        ("net.tcp://localhost:8080/server"));
ChannelFactory<ihelloworld /> factory = new ChannelFactory<ihelloworld />(tcpEndPoint);
factory.Endpoint.Behaviors.Add(new CultureBehaviour());
return factory.CreateChannel();
{% endhighlight %}

#Points of Interest

I did not know for sure that the thread running the <code>AfterReceiveRequest </code>would be the same as the thread running the actual server code. In this case it was. If you need the data from the message header in another part of the WCF pipeline, you should add the data to the properties collection of the request.

After I found the right extension point, the actual implementation was simple. There are many extension points, finding the right one is the difficult part.