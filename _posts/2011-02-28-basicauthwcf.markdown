---
layout: post
title: Basic Authentication using WCF
tags:
- WCF
- Basic Authentication
---

[Source on GitHub](https://github.com/kalkie/BasicAuthenticationUsingWCFRest/tree/master)

#Introduction

This article explains a method to secure a REST based service using Basic Authentication. The service itself is implemented using Microsoft Windows Communication Foundation. The authentication of the credentials should be possible against any type of backend. For example, authenticate against an Active Directory, a custom file, or a database. Basic Authentication is a standard available in combination with WCF and IIS, but the downside of this is that authentication is only possible against an Active Directory.

Although Basic Authentication is a method to secure a web site or service, the authentication mechanism itself is not secure. The user name and password are sent Base64 encoded over the internet. To make this more secure, the server should offer the service using HTTPS. HTTPS secures the channel so that the Base64 encoded user name and password cannot be decrypted. An alternative to Basic Authentication is Digest Authentication which is also possible with WCF REST. I created a separate article that describes Digest Authentication on a WCF REST Service.

This article and the provided source code can be used in two ways. First, just download the code, go down to the Using the code section, and implement your WCF service using Basic Authentication. Second, you can use the article to learn what exactly is Basic Authentication and how can WCF REST be extended.

#Basic Authentication

Basic Authentication is a standard protocol defined within HTTP 1.0 that defines an authentication scheme. In this scheme, the client must authenticate itself with a user-ID and password. Basic Authentication is described in [RFC2617](http://www.faqs.org/rfcs/rfc2617.html). When a client requests a resource from a site that is protected using Basic Authentication, the server returns a 401 "Not authorized" response. Inside this response, the server has added an indication that the site is protected using Basic Authentication. The server adds <em>WWW-Authenticate: Basic realm="site"</em> to the header of the response; <em>Basic</em> indicates that the authentication scheme is Basic Authentication, and <em>realm</em> is a string that indicates which part of the site is protected. It has no further usage in the actual authentication mechanism itself. An internet browser generates a dialog based on this response message. This dialog shows the realm and allows a user to enter a user name and password.

![Basic Authentication 1](../../../img/BasicAuthenticationUsingWCFRest_2.png)

![Basic Authentication 2](../../../img/BasicAuthenticationUsingWCFRest_1.png)

When a user enters the user name and password, the client resends the request b ut adds **Authorization: Basic SGVsbG8gQmFzZTY0** to the header of the request. The characters after **Basic** are the user name and password, separated by a single colon ":" in a Base64 encoded string. The server decodes the string, extracts the credentials, and validates them against the back-end. When the credentials are correctly validated, the server returns the requested content to the client.

![Basic Authentication 2](../../../img/BasicAuthenticationUsingWCFRest_3.png)

#Extending WCF REST

To be able to integrate Basic Authentication with WCF REST, we have to extend the functionality of the WCF framework. The extension is divided into three steps:

- Find the extension point to apply behavior to all operations of the service
- Create a custom authentication mechanism based on existing standards
- Create a security context based on the given credentials

Each of these steps is described in this article.

##Finding the extension point

WCF REST is part of the [WCF REST Starter Kit](http://aspnet.codeplex.com/releases/view/24644). Unlike standard WCF, the WCF Starter Kit provides a simple way to apply the behaviour to all server operations. Adding this behavior to normal WCF can get fairly complex. WCF REST uses Request Interceptors. Request Interceptors shield you from the more complex extensibility points. Request Interceptors are executed at the WCF channel level, and enable you to interpret the incoming request and generate an appropriate response. 

{% highlight csharp %}
public class MyRequestInterceptor : RequestInterceptor
{
   public override void ProcessRequest(ref RequestContext requestContext)
   {
      //Access request with requextContext.RequestMessage
   }
}
{% endhighlight %}

By deriving a new class from the <code>RequestInterceptor</code> base class, you create a custom request interceptor. The method <code>ProcessRequest</code> is executed for every request that arrives at the service. Inside this method, you can read the header of the request and decode the Base64 encoded string if it is present. If it is not present, a response is generated that includes <em>WWW-Authenticate: Basic realm="site"</em> in the header. If it **is** present, we create a new security context and set the credentials on this context.

##Linking the custom RequestInterceptor to the service

The <code>RequestInterceptor</code> can be linked to the service by creating a custom <code>ServiceHostFactory</code>. The <code>ServiceHostFactory</code> is responsible for providing instances of <code>ServiceHost</code> in managed hosting environments. By managed hosting environments I mean services that are hosted within IIS. The <code>CreateServiceHost</code> method of the <code>ServiceHostFactory</code> creates a new <code>WebServiceHost</code> and adds the new <code>RequestInceptor</code> to the <code>Interceptors</code> collection.</p>

{% highlight csharp %}
public class BasicAuthenticationHostFactory : ServiceHostFactory
{
   protected override ServiceHost CreateServiceHost(Type serviceType, 
                                  Uri[] baseAddresses)
   {
      var serviceHost = new WebServiceHost2(serviceType, true, baseAddresses);
      serviceHost.Interceptors.Add(RequestInterceptorFactory.Create(
                    "DataWebService", new CustomMembershipProvider()));
      return serviceHost;
   }
}
{% endhighlight %}

This new <code>ServiceHostFactory</code> is linked to the service through the markup file (**.svc**) of the service. 

{% highlight csharp %}
Factory="WCFServer.BasicAuthenticationHostFactory"
{% endhighlight %}
Inside the markup, you add the host factory of the service.

##Custom authentication method, creating a custom MembershipProvider

The provided source code uses a <code>MembershipProvider</code> to actually perform the authentication of the credentials of the client. You can write your own membership provider by deriving from <code>MembershipProvider</code>. The only method that needs to be implemented is the <code>ValidateUser</code> method. This is the method the code uses to validate the user. This custom <code>MembershipProvider</code> is linked to the <code>RequestInterceptor</code> in the <code>CreateServiceHost</code> method of the <code>ServiceHostFactory</code>.

{% highlight csharp %}
class MembershipProvider
{
   public override bool ValidateUser(string username, string password)
   {
      //perform validation
      return false;
   }

   .....
}
{% endhighlight %}

Some code is removed to improve readability.

##Creating a security context

After we have authenticated the incoming request, we have to create and set the security context. This enables the usage of the client credentials inside a service method. WCF uses the thread local <code>ServiceSecurityContext</code> for this. After the request has been authenticated, a new <code>ServiceSecurityContext</code> is created and added to the incoming request. See the code below from the provided source code that creates a new <code>ServiceSecurityContext</code>.

{% highlight csharp %}
internal ServiceSecurityContext Create(Credentials credentials)
{
   var authorizationPolicies = new List<IAuthorizationpolicy>();
   authorizationPolicies.Add(authorizationPolicyFactory.Create(credentials));
   return new ServiceSecurityContext(authorizationPolicies.AsReadOnly());
}
{% endhighlight %}

This enables to retrieve the user name of a service method as follows:

{% highlight csharp %}
[OperationContract]
public string DoWork()
{
   string name = ServiceSecurityContext.Current.PrimaryIdentity.Name;
   return "Hello" + name;
}
{% endhighlight %}

The method return a string that contains the name of the identity. 

##Using the code

If you want to secure your own WCF REST service with Basic Authentication using the provided source code, you need to execute the following steps:

- Add a reference to the BasicAuthenticationUsingWCF assembly
- Create a new Membership Provider derived from <code>MembershipProvider</code>
- Implement the <code>ValidateUser</code> method against your back-end security storage
- Create a custom BasicAuthenticationHostFactory, see the example in the provided source code
- Add the new BasciAuthenticationHostFactory to the markup of the <em>.svc</em> file

#Points of interest
Although Basic Authentication is a method to secure a web site or service, the authentication mechanism itself is not secure. The user name and password are sent Base64 encoded over the internet. To make this more secure, the server should offer the service using HTTPS. HTTPS secures the channel so that the Base64 encoded user name and password cannot be decrypted.

Basic Authentication in combination with HTTPS is used frequently when you want to offer your service to third parties and provide easy interoperable service. If however you control both side of the wire, client and server, WCF offers a standard security mechanism that can be added to your service using configuration.

This article and source code is based on the example of [Pablo M. Cibraro](http://weblogs.asp.net/cibrax/archive/2009/03/20/custom-basic-authentication-for-restful-services.aspx) who deserves the credits for the solution. The provided source code is developed using TDD, and uses the [NUnit framework](http://nunit.org/) for creating and executing tests. [Rhino Mocks](http://www.ayende.com/projects/rhino-mocks.aspx) is used as a mocking framework inside the unit tests.
