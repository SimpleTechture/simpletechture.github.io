

<!DOCTYPE HTML>
<html>
<head>
<title>Article Source</title>
<link rel="stylesheet" type="text/css" 
    href="//codeproject.cachefly.net/App_Themes/CodeProject/Css/Main.css?dt=2.8.150707.1" />
<base href="http://www.codeproject.com/KB/WCF/" />
</head>
<body>
<!--
HTML for article "Digest Authentication on a WCF REST Service" by Patrick Kalkman
URL: http://www.codeproject.com/KB/WCF/DigestAuthWCFRest.aspx
Copyright 2011 by Patrick Kalkman
All formatting, additions and alterations Copyright © CodeProject, 1999-2015
-->



<p><b>Please choose 'View Source' in your browser to view the HTML, or File | Save to save this 
file to your hard drive for editing.</b></p>

<hr class="Divider subdue" />
<div>




<!-- Start Article -->
<span id="ArticleContent">



<ul class="download">
<li><a href="DigestAuthWCFRest/DigestAuthenticationOnWCF_VS2010_Src.zip">Download source - 258.29 KB</a></li>
</ul>

<h2>Contents</h2>

<ul>
<li><a href="#Introduction0">Introduction</a> </li>

<li><a href="#OverviewofDigestCommunication1">Overview of Digest Communication</a> </li>

<li><a href="#DigestAuthentication2">Digest Authentication</a> 
<ul>
<li><a href="#TheNonce3">The Nonce</a> </li>

<li><a href="#GeneratingtheNonce4">Generating the Nonce</a> </li>

<li><a href="#ValidatingtheNonce5">Validating the Nonce</a> </li>

<li><a href="#QualityOfProtection6">Quality Of Protection</a> </li>

<li><a href="#NoneordefaultQOP7">None or default QOP</a> </li>

<li><a href="#ExtendingWCFREST8">Extending WCF REST</a> </li>

<li><a href="#RetrievingandStoringusercredentials9">Retrieving and Storing user credentials</a> </li>

<li><a href="#Usingthesourcecode10">Using the Source Code</a> </li>
</ul>
</li>

<li><a href="#PointsofInterest11">Points of Interest</a> </li>

<li><a href="#History12">History</a> </li>
</ul>

<h2><a name="Introduction0">Introduction</a></h2>

<p>This is a follow-up on <a title="Basic Authentication on a WCF REST service" href="BasicAuthWCFRest.aspx" target="_blank">my earlier article</a> that described how to use BASIC Authentication with a WCF REST Service. The disadvantage of <a title="Basic Authentication on a WCF REST Service" href="BasicAuthWCFRest.aspx" target="_blank">that solution</a> was that you need a https tunnel to really secure the username password verification process. Although possible, this is not always a feasible situation, for example, if you don't want to invest in certificates or loose performance when using a https tunnel. </p>

<p>This article explains a different authentication mechanism called Digest Authentication which provides an alternative. This security mechanism is more secure than Basic Authentication and does not have the drawbacks from using a https tunnel.</p>

<p>Digest Authentication is available on multiple web servers and supported by multiple internet browsers. The drawback when using Digest Authentication with Internet Information server is that it automatically authenticates credentials against active directory. This article describes an implementation which enables you to secure a WCF REST service with Digest Authentication and authenticate against any back-end.</p>

<p>Digest Authentication was first described in <a title="An Extension to HTTP : Digest Access Authentication" href="http://www.ietf.org/rfc/rfc2069.txt" target="_blank">RFC 2069</a> as an extension to HTTP Basic Authentication. Later, the verification algorithm and security was improved by <a title="HTTP Authentication: Basic and Digest Access Authentication" href="http://www.ietf.org/rfc/rfc2617.txt" target="_blank">RFC 2617</a>. This is the current stable specification. The implementation in this article is based on that <a title="HTTP Authentication: Basic and Digest Access Authentication" href="http://www.ietf.org/rfc/rfc2617.txt" target="_blank">RFC 2617</a> specification. Digest Authentication is more secure because it uses <a title="MD5 cryptographic hashing" href="http://en.wikipedia.org/wiki/MD5" target="_blank">MD5 cryptographic hashing</a> and the use of a nonce to discourage cryptanalysis.</p>

<h2><a name="OverviewofDigestCommunication1">Overview of Digest Communication</a></h2>

<p>Digest communication starts with a client that requests a resource from a web server. If the resource is secured with Digest Authentication, the server will respond with the http status code 401, which means Unauthorized.</p>

<p><img title="Digest Authentication Communication" style="WIDTH: 500px; HEIGHT: 300px" height="300" alt="Digest Authentication Communication" hspace="0" src="DigestAuthWCFRest/DigestAuthenticationUsingWCFRest_1.png" width="500" border="0" /></p>

<p>In the same response, the server indicates in the HTTP header with which mechanism the resource is secured. The HTTP header contains the following <strong>&quot;WWW-Authenticate: Digest realm=&quot;realm&quot;, nonce=&quot;IVjZjc3Yg==&quot;, qop=&quot;auth&quot;</strong>. The first thing you should notice is the string <strong>Digest</strong> in the response, here the server indicates that the resource that was requested by the client is secured using Digest Authentication. Secondly, the server indicates the type of Digest Authentication algorithm to use by the client with <strong>Quality Of Protection (QOP)</strong> and the string called nonce which I will explain later in this article. </p>

<p>An internet browser responds to this by presenting the user a dialog, in this dialog the user is able to enter his username and password. Note, that this dialog does not show the warning about transmitting the credentials in clear text as with a Basic Authentication secured site. </p>

<p><img title="Digest Authentication Credentials Screen" style="WIDTH: 450px; HEIGHT: 265px" height="265" alt="Digest Authentication Credentials Screen" hspace="0" src="DigestAuthWCFRest/DigestAuthenticationUsingWCFRest_3.png" width="450" border="0" /></p>

<p>When the user enters the credentials in this dialog, the browser requests the resource from the server again. This time, the client adds additional information to the HTTP header regarding Digest Authentication.</p>

<p><img title="Digest Authentication Second" style="WIDTH: 500px; HEIGHT: 407px" height="407" alt="Digest Authentication Second" hspace="0" src="DigestAuthWCFRest/DigestAuthenticationUsingWCFRest_2.png" width="500" border="0" /></p>

<p>The server validates the information and returns the requested resource to the client. The details of the response from the server and the additional request of the client will be described in the following part of this article.</p>

<h2><a name="DigestAuthentication2">Digest Authentication</a></h2>

<p>When the server responds to an unauthenticated client request, the server adds a nonce and a qop key to the header of the HTTP response. Both are typical for Digest Authentication. First, the nonce will be described and second the QOP quality of protection.</p>

<h3><a name="TheNonce3">The Nonce</a></h3>

<p>The Nonce stands for &quot;Number used Once&quot;, this is a pseudo random number that ensures that old communications between a client and a server cannot be reused in replay attacks. A replay attack is a network attack in which previous valid data transmission is repeated. This is done by an adversary who intercepts the data and retransmit it. According to the RFC 2716 specification, the Nonce is a server specified data string which should be uniquely generated each time a 401 response is returned by the server. The 401 response that is sent back to the client includes the Nonce generated by the server. According to RFC 2716, the client should add this nonce to the header of next requests.</p>

<h3><a name="GeneratingtheNonce4">Generating the Nonce</a></h3>

<p>The format of the nonce depends on the implementation. Each RFC 2617 digest authentication implementation may define their own nonce format. However, one should carefully design the format of the nonce as it is a part of the quality of the security. For my implementation, I choose to include a date time stamp and the IP address of the client into the nonce. The implementation generates the nonce as follows.</p>
<center>
<p>
<table style="BORDER-RIGHT: rgb(0,0,0) 1px solid; BORDER-TOP: rgb(0,0,0) 1px solid; BORDER-LEFT: rgb(0,0,0) 1px solid; BORDER-BOTTOM: rgb(0,0,0) 1px solid" cellspacing="0" cellpadding="2" bgcolor="#ffbf40" border="0">
<tbody>
<tr>
<td valign="top">
<p>Nonce = Base64( TimeStamp : PrivateHash)</p>
</td>
</tr>
</tbody>
</table>
</p>
</center>
<p>The nonce is generated by <a title="Base64 encoding" href="http://en.wikipedia.org/wiki/Base64" target="_blank">base64</a> encoding the <code>string </code>that is constructed by concatenating the time stamp, a colon and a generated private hash. In the source code, this is handled by the <code>NonceGenerator </code>class which has a <code>Generate </code>method that generates the <code>Nonce string</code>.</p>

<pre lang="cs">public string Generate(string ipAddress)
{
   double dateTimeInMilliSeconds =
      (DateTime.UtcNow - DateTime.MinValue).TotalMilliseconds;
   string dateTimeInMilliSecondsString =
      dateTimeInMilliSeconds.ToString();
   string privateHash = privateHashEncoder.Encode(
      dateTimeInMilliSecondsString,
      ipAddress);
   string stringToBase64Encode =
      string.Format(&quot;{0}:{1}&quot;, dateTimeInMilliSecondsString, privateHash);
   return base64Converter.Encode(stringToBase64Encode);
}</pre>

<p>MD5 is used to generate the private hash of the <code>string </code>that is constructed by concatenating the time stamp, a colon, the IP address of the client, a colon and a private key that is only known to the server. As MD5 is used, the generation is one-way. It is not possible to reconstruct this information from the private hash.</p>
<center>
<p>
<table style="BORDER-RIGHT: rgb(0,0,0) 1px solid; BORDER-TOP: rgb(0,0,0) 1px solid; BORDER-LEFT: rgb(0,0,0) 1px solid; BORDER-BOTTOM: rgb(0,0,0) 1px solid" cellspacing="0" cellpadding="2" bgcolor="#ffbf40" border="0">
<tbody>
<tr>
<td valign="top">
<p>PrivateHash = MD5Hash( TimeStamp : IP Address : Private key)</p>
</td>
</tr>
</tbody>
</table>
</p>
</center>
<p>In the source code, generating the private hash is handled by the method <code>Encode </code>in the <code>PrivateHashEncoder </code>class. It uses the <code>MD5Encoder </code>class to actually generate the MD5 hash.</p>

<pre lang="cs">public string Encode(string dateTimeInMilliSecondsString,
    string ipAddress)
{
  string stringToEncode = string.Format(
     &quot;{0}:{1}:{2}&quot;,
     dateTimeInMilliSecondsString,
     ipAddress,
     privateKey);
  return md5Encoder.Encode(stringToEncode);
}</pre>

<h3><a name="ValidatingtheNonce5">Validating the Nonce</a></h3>

<p>Every time the client sends the nonce to the server, the server validates if this is the nonce that the server sends to the client. The server validates the nonce in two steps:</p>

<p>The first thing that this implementation on the server does is validate if this <code>PrivateHash </code>was generated by this server and returned to this client. The server does this by generating the <code>PrivateHash </code>with the time stamp that is available in the Nonce and the IP address of the client. If this does not deliver the same <code>PrivateHash </code>as in the nonce from the client, the nonce is incorrect and the server responds with a 401. The <code>NonceValidator </code>is responsible in the source code for validating this nonce.</p>

<pre lang="cs">public virtual bool Validate(string nonce,
   string ipAddress)
{
   string[] decodedParts = GetDecodedParts(nonce);
   string md5EncodedString = privateHashEncoder.Encode(
      decodedParts[0],
      ipAddress);
   return string.CompareOrdinal(
      decodedParts[1],
      md5EncodedString) == 0;
}</pre>

<p>Secondly, the server checks if the Time Stamp is too old. The server holds a certain time-out for a nonce. For example, the time-out is 300 seconds. The server validates if the time stamp in the nonce is not older than 300 seconds. If the nonce is older than 300 seconds, the server returns an indication in the HTTP header that the received nonce is too old together with a new nonce. RFC 2617 uses a special key called <strong>Stale</strong> in the header that is sets to <code>true </code>when the Nonce is too old. The <code>NonceValidator </code>is also responsible for checking if the time stamp is too old.</p>

<pre lang="cs">public virtual bool IsStale(string nonce)
{
   string[] decodedParts = GetDecodedParts(nonce);
   DateTime dateTimeFromNonce =
      nonceTimeStampParser.Parse(decodedParts[0]);
   return dateTimeFromNonce.AddSeconds(
      staleTimeOutInSeconds) &lt; DateTime.UtcNow;
}</pre>

<p>By using a time stamp and the IP address in the nonce, we make sure that the request is recent and comes from the client that requested the resource.</p>

<h3><a name="QualityOfProtection6">Quality Of Protection</a></h3>

<p>Digest Authentication allows the server to ask which algorithm the client should use to encrypt the credentials of the user. Digest Authentication allows the following Quality Of Protection.</p>

<ul>
<li><strong>none</strong> = Default protection compatible with <a title="An Extension to HTTP : Digest Access Authentication" href="http://www.ietf.org/rfc/rfc2069.txt" target="_blank">RFC 2069</a> </li>

<li><strong>auth</strong> = Increased protection that includes a client nonce and a client nonce counter </li>

<li><strong>auth-int</strong> = Increased protection and integrity that included all of auth and a hash of the contents of the body </li>
</ul>

<p>Note that it is a request from the server, the client itself is allowed to choose a lesser secure qop algorithm. If the server requests for <code>auth</code>, it is ok for the client to start communicating with the default or none qop.</p>

<p>The implementation with this article supports both <strong>default/none</strong> and auth. The class <code>DefaultDigestEncoder </code>and the class <code>AuthDigestEncoder </code>implement the default and the auth type of quality of protection. Both classes derive from <code>DigestEncodeBase </code>which holds common functionality.</p>

<p><img title="DigestEncoder" style="WIDTH: 427px; HEIGHT: 444px" height="444" alt="DigestEncoder" hspace="0" src="DigestAuthWCFRest/DigestAuthenticationUsingWCFRest_4.png" width="427" border="0" /></p>

<p>At runtime, the server instantiates both types of encoders and stores them in a dictionary with the qop algorithm as the key. This enables the server to easily switch between different types of encoders at runtime.</p>

<pre lang="cs">internal class DigestEncoders :
   Dictionary
{
 public DigestEncoders(MD5Encoder md5Encoder)
 {
  Add(DigestQop.None, new DefaultDigestEncoder(md5Encoder));
  Add(DigestQop.Auth, new AuthDigestEncoder(md5Encoder));
 }

 public virtual DigestEncoderBase GetEncoder(DigestQop digestQop)
 {
  return this[digestQop];
 }
}</pre>

<h3><a name="NoneordefaultQOP7">None or Default QOP</a></h3>

<p>When an internet browser receives 401 HTTP status code with Digest in the authentication header, it will show a dialog for entering the username and password. When the client uses the <strong>default qop</strong> which is compatible with <a title="An Extension to HTTP : Digest Access Authentication" href="http://www.ietf.org/rfc/rfc2069.txt" target="_blank">RFC 2069</a>, the client encrypts the user name and password as follows.</p>
<center>
<p>
<table style="BORDER-RIGHT: rgb(0,0,0) 1px solid; BORDER-TOP: rgb(0,0,0) 1px solid; BORDER-LEFT: rgb(0,0,0) 1px solid; BORDER-BOTTOM: rgb(0,0,0) 1px solid" cellspacing="0" cellpadding="2" bgcolor="#ffbf40" border="0">
<tbody>
<tr>
<td valign="top">
<p>HA1 = MD5( username : realm : password)</p>
</td>
</tr>
</tbody>
</table>
</p>

<table style="BORDER-RIGHT: rgb(0,0,0) 1px solid; BORDER-TOP: rgb(0,0,0) 1px solid; BORDER-LEFT: rgb(0,0,0) 1px solid; BORDER-BOTTOM: rgb(0,0,0) 1px solid" cellspacing="0" cellpadding="2" bgcolor="#ffbf40" border="0">
<tbody>
<tr>
<td valign="top">
<p>HA2 = MD5( method : digestURI)</p>
</td>
</tr>
</tbody>
</table>

<p />

<table style="BORDER-RIGHT: rgb(0,0,0) 1px solid; BORDER-TOP: rgb(0,0,0) 1px solid; BORDER-LEFT: rgb(0,0,0) 1px solid; BORDER-BOTTOM: rgb(0,0,0) 1px solid" cellspacing="0" cellpadding="2" bgcolor="#ffbf40" border="0">
<tbody>
<tr>
<td valign="top">
<p>response = MD5( HA1 : nonce : HA2)</p>
</td>
</tr>
</tbody>
</table>

<p />
</center>
<p>An MD5 hash is created from the user name, realm and password, a separate MD5 hash is created from the HTTP method and the URI of the resource that the client requests. The response is created through a MD5 hash that combines the previous two MD5 hashes and the server generated nonce. The <code>DigestEncoderBase </code>class holds the functionality to generate both the HA1&nbsp;and HA2 hashes.</p>

<pre lang="cs">private string CreateHa1(DigestHeader digestHeader,
   string password)
{
  return md5Encoder.Encode(
    string.Format(
    &quot;{0}:{1}:{2}&quot;,
    digestHeader.UserName,
    digestHeader.Realm,
    password));
}

private string CreateHa2(DigestHeader digestHeader)
{
  return md5Encoder.Encode(
    string.Format(
    &quot;{0}:{1}&quot;,
    digestHeader.Method,
    digestHeader.Uri));
}</pre>

<p>The base classes <code>AuthDigestEncoder </code>and <code>DefaultDigestEncoder </code>are responsible for generating the response. This last step, generating the response, is what differs in the two derived classes. The response of the <strong>Auth</strong> algorithm should be generated differently. The Auth algorithm includes a <code>nonceCount </code>and a client generated Nonce in the response. Also, the actual qop string is concatenated before the hash is calculated.</p>
<center>
<p>
<table style="BORDER-RIGHT: rgb(0,0,0) 1px solid; BORDER-TOP: rgb(0,0,0) 1px solid; BORDER-LEFT: rgb(0,0,0) 1px solid; BORDER-BOTTOM: rgb(0,0,0) 1px solid" cellspacing="0" cellpadding="2" bgcolor="#ffbf40" border="0">
<tbody>
<tr>
<td valign="top">
<p>response = MD5( HA1 : nonce : nonceCount : clientNonce : qop : HA2)</p>
</td>
</tr>
</tbody>
</table>
</p>
</center>
<p>This is why the Auth algorithm is more secure than Default, the server performs an additional check to see if the <code>nonceCount </code>is incremented by the client with every request. The <code>CreateResponse </code>method of the <code>AuthDigestEncoder </code>generates the Auth response.</p>

<pre lang="cs">public override string CreateResponse(
   DigestHeader digestHeader,
   string ha1,
   string ha2)
{
  return
   md5Encoder.Encode(
     string.Format(
     &quot;{0}:{1}:{2}:{3}:{4}:{5}&quot;,
     ha1,
     digestHeader.Nonce,
     digestHeader.NounceCounter,
     digestHeader.Cnonce,
     digestHeader.Qop.ToString(),
     ha2));
}</pre>

<h3><a name="ExtendingWCFREST8">Extending WCF REST</a></h3>

<p>To be able to integrate Digest Authentication with WCF REST, the WCF REST framework has to be extended. This is done by creating a custom <code>RequestInterceptor</code>. For more information, take a look at my <a title="Extending WCF REST through RequestInterceptor" href="BasicAuthWCFRest.aspx" target="_blank">previous article</a> on CodeProject which explains this extension in more detail.</p>

<h3><a name="RetrievingandStoringusercredentials9">Retrieving and Storing User Credentials</a></h3>

<p>The password of the user is transmitted as part of the response generated by the client to the server. It is not possible for the server to extract the password from the response. The server generates a response and checks if the response is equal to the response that was given by the client. This means that there are two options for storing and retrieval of user credentials using Digest Authentication.</p>

<ul>
<li>The first and most secure option is for every user to store the HA1 key in the credentials data storage and validate using the stored HA1 key. This has a disadvantage because you have to change the HA1 key in the data storage if the username, password or realm changes. </li>

<li>The second option is to store the password of the use in the credentials data storage in such a way that it is possible to retrieve the original password. This is obviously less secure than the first option. </li>
</ul>

<h3><a name="Usingthesourcecode10">Using the Source Code</a></h3>

<p>If you want to secure your own WCF REST service with Basic Authentication using the provided source code, you need to execute the following steps:</p>

<ul>
<li>Add a reference to the <code>DigestAuthenticationUsingWCF </code>assembly </li>

<li>Create a custom membership provider derived from <code>MembershipProvider</code> </li>

<li>Implement the <code>ValidateUser</code> method against your back-end security storage </li>

<li>Create a custom membership user derived from Membership user </li>

<li>Implement the <code>GetUser</code> method against your back-end security storage </li>

<li>Create a custom <code>DigestAuthenticationHostFactory</code>, see the example in the provided source code </li>

<li>Add the new <code>DigestAuthenticationHostFactory </code>to the markup of the <em>.svc</em> file </li>
</ul>

<h2><a name="PointsofInterest11">Points of Interest</a></h2>

<p>The provided source code is developed using TDD and uses the <a href="http://nunit.org/">NUnit framework</a> for creating and executing tests. <a href="http://www.ayende.com/projects/rhino-mocks.aspx">Rhino mocks</a> is used as a mocking framework inside the unit tests. </p>

<h2><a name="History12">History</a></h2>

<ul>
<li>28<sup>th</sup> Feb, 2011 
<ul>
<li>Initial post </li>
</ul>
</li>
</ul>




</span>
<!-- End Article -->




</div> 
</body>
</html>
