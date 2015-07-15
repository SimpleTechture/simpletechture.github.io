---
layout: post
title: Weekly Thai Recipe
tags:
- Windows Phone
- Apps
---

|![Splash Screen](../../../img/WeekylThaiRecipeSplash.jpg)|![ScreenShot](../../../img/WeeklyThaiScreenshot.jpg)|
  
#Introduction

This article describes the implementation and process of developing Weekly Thai Recipe. Weekly Thai Recipe is the third application that I developed for the Window Phone. This article is also the third article about developing for windows phone. More information about why I am developing Windows phone applications can be found [here]({% post_url 2012-05-06-bluehour %}) and [here]({% post_url 2012-04-26-regiride %}).

I got a lot of positive reactions from my [first]({% post_url 2012-05-06-bluehour %}) and [second]({% post_url 2012-04-26-regiride %}) article, which made me decide to write a new article and also open up the source of this application. I hope that this article will inspire other people to start developing new windows phone applications. I wrote this article during the implementation of the application and finished it just after Weekly Thai Recipe got certified. Weekly Thai Recipe is currently available in the [Marketplace](http://windowsphone.com/s?appid=45b5b83c-ae6f-419d-bbf0-7ff21884e03d).

#Weekly Thai Recipe

The application Weekly Thai Recipe is an application that provides original recipes from Thailand. The Thai cuisine consist mostly of lightly prepared dishes with strong aroma. It is know for its spiciness and always tries to balance 3 to 4 fundamental tastes in each dish being sour, sweet, bitter and salty.

When the application is started it shows three recipes. Every week that passes another original Thai recipe is added to the recipe list in the application. This new recipe is received from a webservice and stored locally on the phone. A live tile and toast notifications are used to notify the user that a new recipe is available.

#Overall Architecture
The architecture of Weekly Thai Recipe consists of three main parts. First, the Windows Phone application that shows the recipes, secondly a web service and website which manage recipes and last the Microsoft Push Notification Service which is needed to be able to send notifications and tile updates to the Windows phone.
![Architecture](../../../img/WeeklyThaiRecipeArchitecture.jpg)

To register an application with MPNS and send a push notification your application has to follow the following proces.

**1. Register with MPNS:**
  The application registers itself with the MPNS and retrieves an uri that uniquely identifies the channel on which **this single** Windows Phone device can be notified.
    
**2. Register with webservice:**
  The application sends this uri and a unique identifier to a custom webservice (my implementation uses ASP.NET Web API) which stores this unique identifier and uri in a SQL database.
    
**3. Send push notification:**
  Via the website it is possible to send push notifications to all the registered phones. The web application retrieves all registered uri's and sends a specially formatted XML message to the uri to notify each phone.
    
**4. Send the push notification to the phone:**
  MPNS acts as a broker which receives the XML message and then sends it to the phone

By acting as a middle man, MPNS is able to provide control over who and how many messages are send to  phones. It will prevent for example applications flooding an windows phone with too many push notifications.

#Weekly Thai Recipe Phone Architecture

The application on the windows phone that receives and shows the recipes consists of three parts.

![Windows Phone Architecture](../../../img/WindowsPhoneArchitecture.jpg)

First, there is the view which consist of the Windows Phone panorama control. The application uses the MVVM pattern to separate views from the logic. To assist in using the MVVM pattern the [MVVM Light](http://mvvmlight.codeplex.com) framework is used.
Secondly there are a set of services that allow the viewmodel or model to retrieve data from the local storage or from an external source. To assist in retrieving data from external sources such as the recipe web service the [RestSharp](http://restsharp.org/) library for Window Phone is used. And last there is the local recipe storage layer which is responsible for retrieving and storing local recipe data. The recipe's are stored in the isolated storage of the phone in XML format.

##Two different scenarios
These layers assist in two different types of scenario (connected and disconnected) that are supported by the application.

###1. Disconnected scenario
When the phone has (temporary) no network connection, the application will not receive any recipes from the external recipe webservice. Therefore, the application has a set of 3 recipes which are distributed together with the application. When a user does not have a network connection, the application is still able to show the list of the three fixed recipe's. The application checks to see if there is a network connection available and if there is none it receives the recipes from the local XML file. Whether the application has a network connection or not is abstracted by the services layer. The view just calls GetRecipes and receives them. 

###2. Connected scenario
If the phone has a network connection the viewmodel instructs the service to retrieve the recipes. The service layer detects that there is a network connection and send a request to the recipe webservice to check if the latest recipe is already retrieved or not. If not it retrieves the latest recipe, stores it onto the isolated storage of the phone and returns the three fixes recipes and the dynamic recipes to the view.
By designing the application this way, it doesn't have to retrieve the recipes everytime from the webservice. Instead, if it has the latest recipe, it just returns the recipes from the local storage. After the development of the application I also notices that there are other frameworks that provide these kind of (caching) services. For example, [AgFx](https://github.com/shawnburke/AgFx) that provides this functionality. I will be looking at integrating this framework in the next version of the application.

##View (Panorama control)
The panorama control is a control that provide parallax scolling which can be used to provide a digital magazine look and feel. Normally the control has a solid background. But for my app I wanted to do something different. I found [an article by Jeff Wilcox](http://www.jeff.wilcox.name/2010/11/wp7-panorama-smooth-background-changing/) that describes a way to smoothly fade the background of the panorama control. I decided to use this method and use a timer that rotates the background image of the panorama control. 

<iframe width="420" height="315" src="https://www.youtube.com/embed/fe_r7UyB1aw" frameborder="0" allowfullscreen></iframe>

This video shows the effect in the application, every 10 second a different background is shown using a fade effect. I used three different images of recipes.

The <code>BackgroundImageRotator</code> class is responsible for providing new images to the Background image brush.

{% highlight csharp %}
public ImageBrush Rotate()
{
  if (CurrentTheme == AppTheme.Dark)
  {
    string backgroundImageLocation = string.Format("/Images/Panorama{0}.jpg", this.currentBackgroundIndex + 1);
    var backgroundImageBrush = new ImageBrush{ ImageSource = new BitmapImage(new Uri(backgroundImageLocation, UriKind.Relative)) };

    this.currentBackgroundIndex++;
    if (this.currentBackgroundIndex >= NumberOfBackgroundImages)
    {
        this.currentBackgroundIndex = 0;
    }

    return backgroundImageBrush;
  }
  return null;
}
{% endhighlight %}

The <code>Rotate</code> method iterates over the available images and every time it gets called the next image bruch is returned.

#Security
The webservice uses forms authentication to validate if the request comes from a trusted client. This mild form of security prevents the anonymous use of the recipe webservice. The [RestSharp](http://restsharp.org/) library provides a nice clean method to use forms authentication from the client.
A <code>RestRequest</code> is used to perform the request to the webservice. ASP.NET MVC4 is used to implement the recipe webservice and website. Out of the box ASP.NET MVC4 provides a JsonLogin action on the account controller to perform the authentication. 

{% highlight csharp %}
private static RestRequest CreateAuthenticationRequest()
{
    var request = new RestRequest("account/JsonLogin", Method.POST);
    request.AddParameter("UserName", Constants.Settings.UserName);
    request.AddParameter("Password", Constants.Settings.Password);
    return request;
}
{% endhighlight %}

If successfully authenticated, ASP.NET forms authentication provides you with an authorization cookie that should be added to each subsequent request. The [RestSharp](http://restsharp.org/) library provides an easy way to handle this by using a <code>cookiecontainer</code>. When providing an instance of the cookie container to the <code>RestClient</code>, it handles adding the authorization cookie to each subsequent request.

{% highlight csharp %}
var client = new RestClient(Constants.Settings.Recipe_Service_Api_Url);
client.CookieContainer = new CookieContainer();
{% endhighlight %}

#Advertising
It is really easy to integrate ads into your windows phone application. There are different advertisement providers that you can use. Below is a list of possible advertisements providers that you can use with Windows Phone.

- [Microsoft PubCenter](http://pubcenter.microsoft.com)
- [AdDuplex](http://www.adduplex.com/)
- [Google AdMob](www.admob.com/)
- [Inner-Active](http://inner-active.com/)
- [MobFox](http://www.mobfox.com/)
- [Smaato](http://www.smaato.com/)

This is not a complete list. Advertisement on windows phone is still developing, new providers come and existing provides go. I used Microsoft Pubcenter for adding advertisement to Weekly Thai Recipe. Mainly I choose this provider because the integration was very easy todo. I show the advertisement in the recipe detail screen, see the bottom of the screenshot below.

![Advertisement](../../../img/Advertisement.png)

To use Microsoft Pubcenter, you have to sign up at the [web site](http://pubcenter.microsoft.com) and place the adcontrol on to a view of your application. You have to enter your application Id and ad unit Id in the control. That's basically it, you can change the size of the advertisement or provide different size of controls on different views. 

{% highlight xml %}
<pre><Grid x:Name="AdvertisementRow" Grid.Row="2">
  <my:AdControl 
    AdUnitId="your ad unit id" 
    ApplicationId="your application id" 
    Height="80" 
    HorizontalAlignment="Left" Margin="0,-6,0,0" 
    Name="adControl1" 
    VerticalAlignment="Top" 
    Width="480" />
  </Grid>
{% endhighlight %}

#Weekly Thai Recipe Management Architecture
As mentioned in the previous section, the recipe and notification management part of the application is implemented using ASP.NET MVC4 (Beta). Both ASP.NET MVC4 and WebApi are used to provide services to the windows phone application.
There are two areas of functionality provides by the central application, the service to store and retrieve recipes and the ability to notify users of the windows phone application that a new recipe is available.

![Central Architecture](../../../img/CentralArchitecture.jpg)

Razor views are used to provide functionality to store new recipes and they also provide the functionality to send notifications to the connected phones. [Dapper](http://code.google.com/p/dapper-dot-net/) is used to retrieve and store data from the database. Dapper is a simple and fast object mapper for .Net. Dapper is open source framework and is used by the famous [StackOverflow](http://stackoverflow.com/)

#Push notifications

Push notification such as toast notification and tile updates are created by sending an XML message to the url that is received from the Windows Phone application. The central application stores all these url in a SQL database, so that they can be used later to send notifications. 

##Toast notifications

The class <code>ToastSender</code> is responsible for sending toast notifications to all the connection phones. When the <code>Send</code> method is called, the toast notification XML message is send to all the connected phones.

{% highlight csharp %}
public void Send(
   string title, 
   string message, 
   PhoneUriCollection phoneUriCollection)
{
  var toastMessage = "" +
    "<wp:Notification xmlns:wp=\"WPNotification\">" +
       "<wp:Toast>" +
          "<wp:Text1>{0}</wp:Text1>" +
          "<wp:Text2>{1}</wp:Text2>" +
       "</wp:Toast>" +
    "</wp:Notification>";

  toastMessage = string.Format(toastMessage, title, message);

  var messageBytes = System.Text.Encoding.UTF8.
       GetBytes(toastMessa  ge);

  foreach (var uri in phoneUriCollection.Values)
  {
    try
    {
      messageSender.Send(uri, 
        messageBytes, NotificationType.Toast);
    }
    catch (Exception error)
    {
        ErrorSignal.FromCurrentContext().Raise(error);
    }
  }
}
{% endhighlight %}
 
The toast notification XML message has  two text strings, one that indicates the title and another one for the actual text of the notification. This functionality is provided using the razor view from below.

![Central Screenshot](../../../img/CentralScreenShot1.jpg)

#Tile notifications

The class <code>TileSender</code> is responsible for sending Tile notifications to the phones. It works the same as the NotificationSender but a different XML message is send to the phones.

{% highlight csharp %}
public void Send(
  string frontTitle, 
  int count, 
  string frontImageLocation, 
  string backTitle, 
  string backImageLocation, 
  string backContent, 
  PhoneUriCollection phoneUriCollection)
{
  var tileMessage = "" +
    "<wp:Notification xmlns:wp=\"WPNotification\">" +
      "<wp:Tile>" +
        "<wp:BackgroundImage>{2}</wp:BackgroundImage>" +
        "<wp:Count>{1}</wp:Count>" +
        "<wp:Title>{0}</wp:Title>" +
        "<wp:BackBackgroundImage>{4}</wp:BackBackgroundImage>" +
        "<wp:BackContent>{5}</wp:BackContent>" +
        "<wp:BackTitle>{3}</wp:BackTitle>" +
      "</wp:Tile> " +
  "</wp:Notification>";

  tileMessage = string.Format(
    tileMessage, 
    frontTitle, 
    count, 
    frontImageLocation, 
    backTitle, 
    backImageLocation, 
    backContent);

  var messageBytes = System.Text.Encoding.UTF8.GetBytes(tileMessage);

  foreach (var uri in phoneUriCollection.Values)
  {
    try
    {
      messageSender.Send(uri, messageBytes, NotificationType.Tile);
    }
    catch (Exception error)
    {
      ErrorSignal.FromCurrentContext().Raise(error);
    }
  }
}
{% endhighlight %}

##Sending the push notification XML message

The MessageSender class is responsible for sending the bytes of the XML message to the server. The "X-WindowsPhone-Target" and the "X-NotificationClass" headers are added to Http request to indicates the type of notification and when the notification should be send to the phone.
<pre>public void Send(Uri uri, byte[] message, NotificationType notificationType)
{
  var request = (HttpWebRequest)WebRequest.Create(uri);
  request.Method = WebRequestMethods.Http.Post;
  request.ContentType = "text/xml";
  request.ContentLength = message.Length;

  request.Headers.Add(
    "X-MessageID", 
    Guid.NewGuid().ToString());

  switch (notificationType)
  {
    case NotificationType.Toast:
      request.Headers["X-WindowsPhone-Target"] = "toast";
      request.Headers.Add(
        "X-NotificationClass",  
        ((int)BatchingInterval.ToastImmediately)
           .ToString(CultureInfo.InvariantCulture));
      break;
    case NotificationType.Tile:
      request.Headers["X-WindowsPhone-Target"] = "token";
      request.Headers.Add(
        "X-NotificationClass", 
        (int)BatchingInterval.TileImmediately).
           ToString(CultureInfo.InvariantCulture));
      break;
    default:
      request.Headers.Add(
        "X-NotificationClass",
        (int)BatchingInterval.RawImmediately).
           ToString(CultureInfo.InvariantCulture));
      break;
  }

  using (var requestStream = request.GetRequestStream())
  {
    requestStream.Write(message, 0, message.Length);
  }

  try
  {
    var response = (HttpWebResponse)request.GetResponse();
  }
  catch (WebException ex)
  {
    Debug.WriteLine(string.Format("ERROR: {0}", ex.Message));
    throw ex;
  }
}
</pre>
Weekly Thai Recipe uses two types of push notifications, toast and tile. The third type, raw notifications is not used. Raw notifications can be used to send custom data to your windows phone application. Note, that if your application is not running the raw notification is discarded.
<h3><a name="Communicationfromtheviewtotheservice14">Communication from the view to the service</a></h3>
Communication from the view to the service is implemented using jQuery. The service is implemented using an WEB API controller.
<pre>[Authorize]
public class ToastController : ApiController
{
  private readonly ISubscriptionRepository subscriptionRepository;

  private readonly ToastSender toastSender;

  public ToastController(ISubscriptionRepository subscriptionRepository, ToastSender toastSender)
  {
    this.subscriptionRepository = subscriptionRepository;
    this.toastSender = toastSender;
  }

  [HttpPost]
  public void Send(string toastTitle, string toastMessage)
  {
    try
    {
      PhoneUriCollection phonesCollection = subscriptionRepository.GetAll();
      toastSender.Send(toastTitle, toastMessage, phonesCollection);
    }
    catch (Exception error)
    {
      ErrorSignal.FromCurrentContext().Raise(error);
      throw new HttpResponseException(error.Message,
         HttpStatusCode.InternalServerError);
    }
  }
}</pre>
The ToastController is responsible for receiving the Toast send request. It contains a title and a message that should be send to all the registered phones. 
A javascript class is used to actually send the message to the controller. The inialize methods binds a method to the blick event of the sendToastButton. The method retrieves the value from the title and message text field and sends them to the controller using the $.ajax jQuery method. When the method succeeds it shows an alert that the toast has been send.
<pre lang="javascript">var toastInitializer = function () {

  var initialize = function (sendToastUrl) {

    $('#sendToastButton').click(function () {

      var toastTitle = $('#toastTitle').val();
      var toastMessage = $('#toastMessage').val();

      if (toastTitle.length > 0 &amp;&amp; toastMessage.length > 0) {
        $.ajax({
          url: sendToastUrl,
          success: function (data) {
            alert('Toast is send');
          },
          data: {
            toastTitle: toastTitle,
            toastMessage: toastMessage
          },
          dataType: "json",
          type: "POST"
       });
     }
   });
  };

  return {
    initialize: initialize
  };
};
</pre>
Error handling on the client is  centralized by overriding the jQuery $.ajax error method. 
<pre>var generalErrorHandler = function() {

  var initialize = function() {
    $.ajaxSetup({
      "error": function (xhr) {
        var errorMessage = $('#errorMessage');
        if (errorMessage.length > 0) {
          errorMessage.html(xhr.responseText);
        }

        var errorDialog = $('#errorDialog');
        if (errorDialog.length > 0) {
          errorDialog.show();
        }
      } 
    });
  };

  return {
    initialize: initialize
  };
};
</pre> 
An error div is added to the shared ASP.NET MVC _Layout.cshtml
  that shows the actual error 
<pre><body>
  <div id="errorDialog" class="errorDialog" style="display: none">
    <div id="errorIcon" class="errorIcon" style="cursor: pointer;" ><!-- --></div>
    <div class="messageText">
      <span id="errorMessage"></span>
    </div>
  </div>
  @this.RenderBody()
</body>
</pre>
The general error handler and all the necessary javascript classes are initialized in the  $(document).ready method of the main view.
<pre>$(document).ready(function () {

  var errorHandler = new generalErrorHandler();
  errorHandler.initialize();

  var toast = new toastInitializer();
  toast.initialize('@Url.Action("Send", "api/Toast")');

  var tile = new tileInitializer();
  tile.initialize('@Url.Action("Send", "api/Tile")');

  var recipe = new recipeSaver();
  recipe.initalize('@Url.Action("Save", "api/Recipe")');

});
</pre>
<h3><a name="DapperDataAccess15">Dapper Data Access</a></h3>
I used Dapper for data access because using a full ORM such as Entity Framework or NHibernate is just too much for managing those two or three database tables. Dapper is what is called a micro orm. It provides a subset of the services provided by a full ORM. This enables you to have more power and control over how records are stored and retrieved from the database.
Dapper provides   functionality to parameterize queries and materialize the results from those queries. For example, the following source code is used to retrieve all the registered phones (subscriptions) from the database.
<pre>IEnumerable<subscription> subscriptions = connection.Query<subscription>(@"select PhoneId, Uri from subscription");
</subscription></subscription></pre>
Dapper takes care of performing the query and creating a list of subscription instances filled with the data from the database. The mapping between the table columns and the property of the class is based on the name of the column and the name of the property. These two column name and property name should be the same for Dapper to recognize this.
<h2><a name="Toolsandframeworks16">Tools and frameworks</a></h2>
Below the list of tools and frameworks I used for developing Weekly Thai Recipe.
Windows Phone Application


<ul>
<li><a href="http://mvvmlight.codeplex.com/">MVVM Light</a>
</li><li><a href="http://silverlight.codeplex.com/">Silverlight Toolkit</a>
</li><li><a href="http://www.mtiks.com/">Mtiks</a>
</li><li><a href="http://mvvmlight.codeplex.com/">SimpleIoc</a></li>
<li><a href="http://restsharp.org/">RestSharp</a></li>
</ul>
Central Management Application
<ul>
  <li><a href="http://www.asp.net/mvc/mvc4">ASP.NET MVC4 (Beta)</a></li>
  <li><a href="http://docs.structuremap.net/">StructureMap</a></li>
  <li><a href="http://code.google.com/p/elmah/">Elmah</a></li>
  <li><a href="http://code.google.com/p/dapper-dot-net/">Dapper</a></li>
</ul>

#Using the source

There a two solutions inside the source package. For opening the Windows Phone solution you need Visual Studio 2010 and the <a href="http://www.microsoft.com/en-us/download/details.aspx?id=27570">Windows Phone SDK</a> installed. For opening the central recipe management solution you need to install <a href="http://www.asp.net/mvc/mvc4">ASP.NET MVC4 (beta)</a>.There is a detached SQL server database in the source package which you can attach to your local SQL server to create a fully working application. If you use this database can login with username Codeproject and password Codeproject.&nbsp;
<h2><a name="Conclusion18">Conclusion</a></h2>
The application is available in the <a href="http://windowsphone.com/s?appid=9fffb384-8d52-46cc-82a4-e05d931920c7">marketplace</a> and the full source code can be downloaded from the top of the article. If you like the article, a vote or comment is appreciated. Thanks.
<h2><a name="What'snext19">What's next?</a></h2>
My next Window Phone project will be a game using the XNA framework. I am still figuring out the details, but it will appear here on CodeProject in the form of a  article.
<h2><a name="History20">History</a></h2>
<ul>
  <li>v1.0 First version</li><li>v1.1 Update of the source of the central application to use the RC of ASP.NET MVC4 (Thanks to Jeff Albrecht for the headsup!)&nbsp;</li><li>v1.2 Added a detached database to the source zip to create a complete package.&nbsp;&nbsp;&nbsp;</li>
</ul>



</span>
<!-- End Article -->




</div> 
</body>
</html>
