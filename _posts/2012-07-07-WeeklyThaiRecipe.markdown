---
layout: post
title: Weekly Thai Recipe
tags:
- Windows Phone
- Apps
---
#Introduction

|![Splash Screen](../../../img/WeekylThaiRecipeSplash.jpg)|![ScreenShot](../../../img/WeeklyThaiScreenshot.jpg)|

This article describes the implementation and process of developing Weekly Thai Recipe. Weekly Thai Recipe is the third application that I developed for the Window Phone. This article is also the third article about developing for windows phone. More information about why I am developing Windows phone applications can be found [here]() and [here]().

I got a lot of positive reactions from my [first]() and [second]()article, which made me decide to write a new article and also open up the source of this application. I hope that this article will inspire other people to start developing new windows phone applications. I wrote this article during the implementation of the application and finished it just after Weekly Thai Recipe got certified. Weekly Thai Recipe is currently available in the [Marketplace](http://windowsphone.com/s?appid=45b5b83c-ae6f-419d-bbf0-7ff21884e03d).

#Weekly Thai Recipe

The application Weekly Thai Recipe is an application that provides original recipes from Thailand. The Thai cuisine consist mostly of lightly prepared dishes with strong aroma. It is know for its spiciness and always tries to balance 3 to 4 fundamental tastes in each dish being sour, sweet, bitter and salty.

When the application is started it shows three recipes. Every week that passes another original Thai recipe is added to the recipe list in the application. This new recipe is received from a webservice and stored locally on the phone. A live tile and toast notifications are used to notify the user that a new recipe is available.

#Overall Architecture
The architecture of Weekly Thai Recipe consists of three main parts. First, the Windows Phone application that shows the recipes, secondly a web service and website which manage recipes and last the Microsoft Push Notification Service which is needed to be able to send notifications and tile updates to the Windows phone.
![Architecture](../../../img/WeeklyThaiRecipeArchitecture.jpg)

To register an application with MPNS and send a push notification your application has to follow the following proces.

1. Register with MPNS
The application registers itself with the MPNS and retrieves an uri that uniquely identifies the channel on which **this single** Windows Phone device can be notified.
2. Register with webservice
The application sends this uri and a unique identifier to a custom webservice (my implementation uses ASP.NET Web API) which stores this unique identifier and uri in a SQL database.
3. Send push notification
Via the website it is possible to send push notifications to all the registered phones. The web application retrieves all registered uri's and sends a specially formatted XML message to the uri to notify each phone.
4. Send the push notification to the phone
MPNS acts as a broker which receives the XML message and then sends it to the phone

By acting as a middle man, MPNS is able to provide control over who and how many messages are send to  phones. It will prevent for example applications flooding an windows phone with too many push notifications.

#Weekly Thai Recipe Phone Architecture

The application on the windows phone that receives and shows the recipes consists of three parts.

![Windows Phone Architecture](../../../img/WindowsPhoneArchitecture.jpg)

First, there is the view which consist of the Windows Phone panorama control. The application uses the MVVM pattern to separate views from the logic. To assist in using the MVVM pattern the <a href="http://mvvmlight.codeplex.com/">MVVM Light</a> framework is used.&nbsp;</p>
<p>Secondly there are a set of service that allow the viewmodel or model to retrieve data from the local storage or from an external source. To assist in retrieving data from external sources such as the recipe web service the <a href="http://restsharp.org/">RestSharp</a> library for Window Phone is used.</p>
<p>And last there is the local recipe storage layer which is responsible for retrieving and storing local recipe data. The recipe's are stored in the isolated storage of the phone in XML format.</p>
<h3><a name="Twodifferentscenarios5">Two different scenarios</a></h3>
<p>These layers assist in two different types of scenario (connected and disconnected) that are supported by the application. </p>
<p>1. Disconnected scenario</p>
<p>When the phone has (temporary) no network connection, the application will not receive any recipes from the external recipe webservice. Therefore, the application has a set of 3 recipes which are distributed together with the application. When a user does not have a network connection, the application is still able to show the list of the three fixed recipe's.</p>
<p>The application checks to see if there is a network connection available and if there is none it receives the recipes from the local XML file. Whether the application has a network connection or not is abstracted by the services layer. The view just calls GetRecipes and receives them. </p>
<p>2. Connected scenario&nbsp;</p>
<p>If the phone has a network connection the viewmodel instructs the service to retrieve the recipes. The service layer detects that there is a network connection and send a request to the recipe webservice to check if the latest recipe is already retrieved or not. If not it retrieves the latest recipe, stores it onto the isolated storage of the phone and returns the three fixes recipes and the dynamic recipes to the view.</p>
<p>By designing the application this way, it doesn't have to retrieve the recipes everytime from the webservice. Instead, if it has the latest recipe, it just returns the recipes from the local storage. After the development of the application I also notices that there are other frameworks that provide these kind of (caching) services. For example, <a href="https://github.com/shawnburke/AgFx">AgFx</a> that provides this functionality. I will be looking at integrating this framework in the next version of the application.</p>
<h3><a name="View(Panoramacontrol)6">View (Panorama control)</a></h3>
<p>The panorama control is a control that provide parallax scolling which can be used to provide a digital magazine look and feel. Normally the control has a solid background. But for my app I wanted to do something different. I found <a href="http://www.jeff.wilcox.name/2010/11/wp7-panorama-smooth-background-changing/">an article by Jeff Wilcox</a> that describes a way to smoothly fade the background of the panorama control. I decided to use this method and use a timer that rotates the background image of the panorama control.&nbsp;</p><p><a href="http://www.youtube.com/embed/fe_r7UyB1aw" target="_blank" title="Background fading">This video</a>&nbsp;shows the effect in the application, every 10 second a different background is shown using a fade effect. I used three different images of recipes.</p>

<p>The <code>BackgroundImageRotator</code> class is responsible for providing new images to the Background image brush.</p>
<pre lang="csharp">public ImageBrush Rotate()
{
  if (CurrentTheme == AppTheme.Dark)
  {
    string backgroundImageLocation = string.Format(&quot;/Images/Panorama{0}.jpg&quot;, this.currentBackgroundIndex + 1);
    var backgroundImageBrush = new ImageBrush{ ImageSource = new BitmapImage(new Uri(backgroundImageLocation, UriKind.Relative)) };

    this.currentBackgroundIndex++;
    if (this.currentBackgroundIndex &gt;= NumberOfBackgroundImages)
    {
        this.currentBackgroundIndex = 0;
    }

    return backgroundImageBrush;
  }
  return null;
}
</pre>
<p>The <code>Rotate</code> method iterates over the available images and every time it gets called the next image bruch is returned.</p>
<h3><a name="Security7">Security</a></h3>
<p>The webservice uses forms authentication to validate if the request comes from a trusted client. This mild form of security prevents the anonymous use of the recipe webservice. The <a href="http://restsharp.org/">RestSharp</a> library provides a nice clean method to use forms authentication from the client.</p>
<p>A <code>RestRequest</code> is used to perform the request to the webservice. ASP.NET MVC4 is used to implement the recipe webservice and website. Out of the box ASP.NET MVC4 provides a JsonLogin action on the account controller to perform the authentication. 
</p><pre lang="csharp">private static RestRequest CreateAuthenticationRequest()
{
    var request = new RestRequest(&quot;account/JsonLogin&quot;, Method.POST);
    request.AddParameter(&quot;UserName&quot;, Constants.Settings.UserName);
    request.AddParameter(&quot;Password&quot;, Constants.Settings.Password);
    return request;
}</pre>
<p>If successfully authenticated ASP.NET forms authentication provides you with an authorization cookie that should be added to each subsequent request. The <a href="http://restsharp.org/">RestSharp</a> library provides an easy way to handle this by using a <code>cookiecontainer</code>. When providing an instance of the cookie container to the <code>RestClient</code>, it handles adding the authorization cookie to each subsequent request.</p>
<pre lang="cs">var client = new RestClient(Constants.Settings.Recipe_Service_Api_Url);
client.CookieContainer = new CookieContainer();</pre>
<h3><a name="Advertising8">Advertising</a></h3>
<p>It is really easy to integrate ads into your windows phone application. There are different advertisement providers that you can use. Below is a list of possible advertisements providers that you can use with Windows Phone.</p>
<ul>
<li><a href="http://pubcenter.microsoft.com">Microsoft PubCenter</a></li>
<li><a href="http://www.adduplex.com/">AdDuplex</a></li>
<li><a href="www.admob.com/">Google AdMob</a></li>
<li><a href="http://inner-active.com/">Inner-Active</a></li>
<li><a href="http://www.mobfox.com/">MobFox</a></li>
<li><a href="http://www.smaato.com/">Smaato</a></li>
</ul>
<p>This is not a complete list. Advertisement on windows phone is still developing, new providers come and existing provides go.</p>
<p> I used Microsoft Pubcenter for adding advertisement to Weekly Thai Recipe. Mainly I choose this provider because the integration was very easy todo. I show the advertisement in the recipe detail screen, see the bottom of the screenshot below. </p>
<p><img width="240" height="400" src="409470/Advertisement.png" /></p>
<p>To use Microsoft Pubcenter, you have to sign up at the <a href="http://pubcenter.microsoft.com">website</a> and place the adcontrol on to a view of your application. You have to enter your application Id and ad unit Id in the control. That's basically it, you can change the size of the advertisement or provide different size of controls on different views. </p>
<pre>&lt;Grid x:Name=&quot;AdvertisementRow&quot; Grid.Row=&quot;2&quot;&gt;
  &lt;my:AdControl 
    AdUnitId=&quot;your ad unit id&quot; 
    ApplicationId=&quot;your application id&quot; 
    Height=&quot;80&quot; 
    HorizontalAlignment=&quot;Left&quot; Margin=&quot;0,-6,0,0&quot; 
    Name=&quot;adControl1&quot; 
    VerticalAlignment=&quot;Top&quot; 
    Width=&quot;480&quot; /&gt;
  &lt;/Grid&gt;</pre>
<h2><a name="WeeklyThaiRecipeManagementArchitecture9">Weekly Thai Recipe Management Architecture</a></h2>
<p>As mentioned in the previous section the recipe and notification management part of the application is implemented using ASP.NET MVC4 (Beta). Both ASP.NET MVC4 and WebApi are used to provide services to the windows phone application.</p>
<p>There are two areas of functionality provides by the central application, the service to store and retrieve recipes and the ability to notify users of the windows phone application that a new recipe is available.</p>
<center><img src="409470/CentralArchitecture.jpg" /></center>
<p>Razor views are used to provide functionality to store new recipes and they also provide the functionality to send notifications to the connected phones. <a href="http://code.google.com/p/dapper-dot-net/">Dapper</a> is used to retrieve and store data from the database. Dapper is a simple and fast object mapper for .Net. Dapper is open source framework and is used by the famous <a href="http://stackoverflow.com/">StackOverflow</a> web site.</p>
<h3><a name="Pushnotifications10">Push notifications</a></h3>
<p>Push notification such as toast notification and tile updates are created by sending an XML message to the url that is received from the Windows Phone application. The central application stores all these url in a SQL database, so that they can be used later to send notifications. </p>
<h4><a name="Toastnotifications11">Toast notifications</a></h4>
<p>The class <code>ToastSender</code> is responsible for sending toast notifications to all the connection phones. When the <code>Send</code> method is called, the toast notification XML message is send to all the connected phones.</p>
<pre>public void Send(
   string title, 
   string message, 
   PhoneUriCollection phoneUriCollection)
{
  var toastMessage = &quot;&quot; +
    &quot;&lt;wp:Notification xmlns:wp=\&quot;WPNotification\&quot;&gt;&quot; +
       &quot;&lt;wp:Toast&gt;&quot; +
          &quot;&lt;wp:Text1&gt;{0}&lt;/wp:Text1&gt;&quot; +
          &quot;&lt;wp:Text2&gt;{1}&lt;/wp:Text2&gt;&quot; +
       &quot;&lt;/wp:Toast&gt;&quot; +
    &quot;&lt;/wp:Notification&gt;&quot;;

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
</pre> 
<p>The toast notification XML message has  two text strings, one that indicates the title and another one for the actual text of the notification. This functionality is provided using the razor view from below.</p>
<p><img width="425" height="336" src="409470/CentralScreenShot1.jpg" /></p>
<h4><a name="Tilenotifications12">Tile notifications</a></h4>
<p>The class <code>TileSender</code> is responsible for sending Tile notifications to the phones. It works the same as the NotificationSender but a different XML message is send to the phones.</p>
<pre>public void Send(
  string frontTitle, 
  int count, 
  string frontImageLocation, 
  string backTitle, 
  string backImageLocation, 
  string backContent, 
  PhoneUriCollection phoneUriCollection)
{
  var tileMessage = &quot;&quot; +
    &quot;&lt;wp:Notification xmlns:wp=\&quot;WPNotification\&quot;&gt;&quot; +
      &quot;&lt;wp:Tile&gt;&quot; +
        &quot;&lt;wp:BackgroundImage&gt;{2}&lt;/wp:BackgroundImage&gt;&quot; +
        &quot;&lt;wp:Count&gt;{1}&lt;/wp:Count&gt;&quot; +
        &quot;&lt;wp:Title&gt;{0}&lt;/wp:Title&gt;&quot; +
        &quot;&lt;wp:BackBackgroundImage&gt;{4}&lt;/wp:BackBackgroundImage&gt;&quot; +
        &quot;&lt;wp:BackContent&gt;{5}&lt;/wp:BackContent&gt;&quot; +
        &quot;&lt;wp:BackTitle&gt;{3}&lt;/wp:BackTitle&gt;&quot; +
      &quot;&lt;/wp:Tile&gt; &quot; +
  &quot;&lt;/wp:Notification&gt;&quot;;

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

</pre>
<h4><a name="SendingthepushnotificationXMLmessage13">Sending the push notification XML message</a></h4>
<p>The MessageSender class is responsible for sending the bytes of the XML message to the server. The &quot;X-WindowsPhone-Target&quot; and the &quot;X-NotificationClass&quot; headers are added to Http request to indicates the type of notification and when the notification should be send to the phone.</p>
<pre>public void Send(Uri uri, byte[] message, NotificationType notificationType)
{
  var request = (HttpWebRequest)WebRequest.Create(uri);
  request.Method = WebRequestMethods.Http.Post;
  request.ContentType = &quot;text/xml&quot;;
  request.ContentLength = message.Length;

  request.Headers.Add(
    &quot;X-MessageID&quot;, 
    Guid.NewGuid().ToString());

  switch (notificationType)
  {
    case NotificationType.Toast:
      request.Headers[&quot;X-WindowsPhone-Target&quot;] = &quot;toast&quot;;
      request.Headers.Add(
        &quot;X-NotificationClass&quot;,  
        ((int)BatchingInterval.ToastImmediately)
           .ToString(CultureInfo.InvariantCulture));
      break;
    case NotificationType.Tile:
      request.Headers[&quot;X-WindowsPhone-Target&quot;] = &quot;token&quot;;
      request.Headers.Add(
        &quot;X-NotificationClass&quot;, 
        (int)BatchingInterval.TileImmediately).
           ToString(CultureInfo.InvariantCulture));
      break;
    default:
      request.Headers.Add(
        &quot;X-NotificationClass&quot;,
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
    Debug.WriteLine(string.Format(&quot;ERROR: {0}&quot;, ex.Message));
    throw ex;
  }
}
</pre>
<p>Weekly Thai Recipe uses two types of push notifications, toast and tile. The third type, raw notifications is not used. Raw notifications can be used to send custom data to your windows phone application. Note, that if your application is not running the raw notification is discarded.</p>
<h3><a name="Communicationfromtheviewtotheservice14">Communication from the view to the service</a></h3>
<p>Communication from the view to the service is implemented using jQuery. The service is implemented using an WEB API controller.</p>
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
<p>The ToastController is responsible for receiving the Toast send request. It contains a title and a message that should be send to all the registered phones. </p>
<p>A javascript class is used to actually send the message to the controller. The inialize methods binds a method to the blick event of the sendToastButton. The method retrieves the value from the title and message text field and sends them to the controller using the $.ajax jQuery method. When the method succeeds it shows an alert that the toast has been send.</p>
<pre lang="javascript">var toastInitializer = function () {

  var initialize = function (sendToastUrl) {

    $('#sendToastButton').click(function () {

      var toastTitle = $('#toastTitle').val();
      var toastMessage = $('#toastMessage').val();

      if (toastTitle.length &gt; 0 &amp;&amp; toastMessage.length &gt; 0) {
        $.ajax({
          url: sendToastUrl,
          success: function (data) {
            alert('Toast is send');
          },
          data: {
            toastTitle: toastTitle,
            toastMessage: toastMessage
          },
          dataType: &quot;json&quot;,
          type: &quot;POST&quot;
       });
     }
   });
  };

  return {
    initialize: initialize
  };
};
</pre>
<p>Error handling on the client is  centralized by overriding the jQuery $.ajax error method. </p>
<pre>var generalErrorHandler = function() {

  var initialize = function() {
    $.ajaxSetup({
      &quot;error&quot;: function (xhr) {
        var errorMessage = $('#errorMessage');
        if (errorMessage.length &gt; 0) {
          errorMessage.html(xhr.responseText);
        }

        var errorDialog = $('#errorDialog');
        if (errorDialog.length &gt; 0) {
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
<p>An error div is added to the shared ASP.NET MVC _Layout.cshtml
  that shows the actual error </p>
<pre>&lt;body&gt;
  &lt;div id=&quot;errorDialog&quot; class=&quot;errorDialog&quot; style=&quot;display: none&quot;&gt;
    &lt;div id=&quot;errorIcon&quot; class=&quot;errorIcon&quot; style=&quot;cursor: pointer;&quot; &gt;&lt;!-- --&gt;&lt;/div&gt;
    &lt;div class=&quot;messageText&quot;&gt;
      &lt;span id=&quot;errorMessage&quot;&gt;&lt;/span&gt;
    &lt;/div&gt;
  &lt;/div&gt;
  @this.RenderBody()
&lt;/body&gt;
</pre>
<p>The general error handler and all the necessary javascript classes are initialized in the  $(document).ready method of the main view.</p>
<pre>$(document).ready(function () {

  var errorHandler = new generalErrorHandler();
  errorHandler.initialize();

  var toast = new toastInitializer();
  toast.initialize('@Url.Action(&quot;Send&quot;, &quot;api/Toast&quot;)');

  var tile = new tileInitializer();
  tile.initialize('@Url.Action(&quot;Send&quot;, &quot;api/Tile&quot;)');

  var recipe = new recipeSaver();
  recipe.initalize('@Url.Action(&quot;Save&quot;, &quot;api/Recipe&quot;)');

});
</pre>
<h3><a name="DapperDataAccess15">Dapper Data Access</a></h3>
<p>I used Dapper for data access because using a full ORM such as Entity Framework or NHibernate is just too much for managing those two or three database tables. Dapper is what is called a micro orm. It provides a subset of the services provided by a full ORM. This enables you to have more power and control over how records are stored and retrieved from the database.</p>
<p>Dapper provides   functionality to parameterize queries and materialize the results from those queries. For example, the following source code is used to retrieve all the registered phones (subscriptions) from the database.</p>
<pre>IEnumerable&lt;subscription&gt; subscriptions = connection.Query&lt;subscription&gt;(@&quot;select PhoneId, Uri from subscription&quot;);
&lt;/subscription&gt;&lt;/subscription&gt;</pre>
<p>Dapper takes care of performing the query and creating a list of subscription instances filled with the data from the database. The mapping between the table columns and the property of the class is based on the name of the column and the name of the property. These two column name and property name should be the same for Dapper to recognize this.</p>
<h2><a name="Toolsandframeworks16">Tools and frameworks</a></h2>
<p>Below the list of tools and frameworks I used for developing Weekly Thai Recipe.</p>
<p>Windows Phone Application</p>


<ul>
<li><a href="http://mvvmlight.codeplex.com/">MVVM Light</a>
</li><li><a href="http://silverlight.codeplex.com/">Silverlight Toolkit</a>
</li><li><a href="http://www.mtiks.com/">Mtiks</a>
</li><li><a href="http://mvvmlight.codeplex.com/">SimpleIoc</a></li>
<li><a href="http://restsharp.org/">RestSharp</a></li>
</ul>
<p>Central Management Application</p>
<ul>
  <li><a href="http://www.asp.net/mvc/mvc4">ASP.NET MVC4 (Beta)</a></li>
  <li><a href="http://docs.structuremap.net/">StructureMap</a></li>
  <li><a href="http://code.google.com/p/elmah/">Elmah</a></li>
  <li><a href="http://code.google.com/p/dapper-dot-net/">Dapper</a></li>
</ul>
<h2><a name="Usingthesource17">Using the source</a></h2>
<p>There a two solutions inside the source package. For opening the Windows Phone solution you need Visual Studio 2010 and the <a href="http://www.microsoft.com/en-us/download/details.aspx?id=27570">Windows Phone SDK</a> installed. For opening the central recipe management solution you need to install <a href="http://www.asp.net/mvc/mvc4">ASP.NET MVC4 (beta)</a>.</p><p>There is a detached SQL server database in the source package which you can attach to your local SQL server to create a fully working application. If you use this database can login with username Codeproject and password Codeproject.&nbsp;</p>
<h2><a name="Conclusion18">Conclusion</a></h2>
<p>The application is available in the <a href="http://windowsphone.com/s?appid=9fffb384-8d52-46cc-82a4-e05d931920c7">marketplace</a> and the full source code can be downloaded from the top of the article. If you like the article, a vote or comment is appreciated. Thanks.</p>
<h2><a name="What'snext19">What's next?</a></h2>
<p>My next Window Phone project will be a game using the XNA framework. I am still figuring out the details, but it will appear here on CodeProject in the form of a  article.</p>
<h2><a name="History20">History</a></h2>
<ul>
  <li>v1.0 First version</li><li>v1.1 Update of the source of the central application to use the RC of ASP.NET MVC4 (Thanks to Jeff Albrecht for the headsup!)&nbsp;</li><li>v1.2 Added a detached database to the source zip to create a complete package.&nbsp;&nbsp;&nbsp;</li>
</ul>



</span>
<!-- End Article -->




</div> 
</body>
</html>
