---
title: "Connecting to Office365 by API"
datePublished: Mon May 14 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iri6k00xszfs13y2sgxa9
slug: connecting-to-office365-by-api-the-code
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380790230/YIy0RdQV-.jpeg
tags: csharp, authentication, authorization, dotnetcore

---


At work, we made the switch from a local mail server that was accessed over POP3 and IMAP to Office365 Outlook which we access through their RESTful API. To learn more about how this works, I tried to duplicate this process so I can access my personal Outlook emails via a console application.

To get access to my emails, there are two facets: the code to retrieve the emails and the setup to allow the code to access the information. Let's start with the easiest part (and my favourite): the code to access the emails.

In this example, I will only be [fetching emails](https://msdn.microsoft.com/en-us/office/office365/api/mail-rest-operations?f=255&MSPPError=-2147217396#get-messages) to keep this example simple. For more information on sending too, I refer to the [full MSDN article](https://msdn.microsoft.com/en-us/office/office365/api/mail-rest-operations?f=255&MSPPError=-2147217396#). I want to draw attention to the fact that the v1.0 API is being deprecated starting 1st of November 2018 and will be taken offline the 1st of November 2019. The biggest change is to how access can be granted to users. Don't worry, I'll highlight that as best I can for services.

On to the code. The first thing I did was use a [JSON to C# service](https://www.google.be/search?q=json+to+c%23&oq=json+to+c%23) to map the sample response to [a C# class](https://github.com/KenBonny/KenBonny.Office365Experiment/blob/master/KenBonny.Office365Access.Console/Email.cs). This allows me to easily deserialise the response. The request for mails is then very simple:

```
private static IReadOnlyCollection GetEmails(string clientId, string clientSecret)
{
  var officeClient = new RestClient("https://graph.microsoft.com/v1.0/me")
  {
    Authenticator = new Office365Authenticator(clientId, clientSecret)
  };
  var request = new RestRequest("messages", Method.GET);
  var response = officeClient.Execute(request);
  return response.IsSuccessful ? response.Data.Emails : new Email[0];
}
```
Here I use [RestSharp](http://restsharp.org/) as an http client to make a `GET` request to the https://graph.microsoft.com api to get all information about emails. You may notice that this is not the https://outlook.office.com API as is used in the MSDN article I linked to earlier. The [Microsoft Graph API](https://developer.microsoft.com/en-us/graph/docs/concepts/overview) is their newest API to get to information. The interface of this API is similar to the outlook.office.com API. I used the two documentations to get the code to work. The API's are very similar and the documentation of one extends mostly to the other.

[Microsoft suggests](https://docs.microsoft.com/en-us/outlook/rest/compare-graph-outlook) moving away from the outlook.office.com API and on to the graph.microsoft.com API. The API looks very similar to the outlook.office.com API, but the graph API will be their preferred API going forward.

The last thing that you might have noticed is that I used a custom `Authenticator`. According to the Microsoft Graph API documentation on how to [get an authentication token](https://developer.microsoft.com/en-us/graph/docs/concepts/auth_overview) I need to request one from the https://login.microsoftonline.com/ endpoint with the following parameters:

```
private static IRestRequest CreateAuthenticationRequest(string clientId, string clientSecret)
{
  var request = new RestRequest("/24b1c222-db0f-4bef-b980-e08ac1762707/oauth2/v2.0/token", Method.POST);
  request.AddParameter("grant_type", "client_credentials");
  request.AddParameter("client_id", clientId);
  request.AddParameter("client_secret", clientSecret);
  request.AddParameter("scope", "https://graph.microsoft.com/.default");
  request.AddHeader("Content-Type", "application/x-www-form-urlencoded");
  return request;
}
```

The response of this request will contain the token that will need to be passed along to every request to the graph API. That is why I created a custom object to contain the response:

```
internal class Office365AuthenticationToken
{
  private DateTime _createdDate;
  public Office365AuthenticationToken()
  {
    _createdDate = DateTime.Now;
  }
  [JsonProperty("access_token")] public string AccessToken { get; set; }
  [JsonProperty("token_type")] public string TokenType { get; set; }
  [JsonProperty("expires_in")] public int ExpiresIn { get; set; }
  [JsonProperty("expires_on")] public long ExpiresOn { get; set; }
  [JsonProperty("resource")] public string Resource { get; set; }
  public string Token => $"{TokenType} {AccessToken}";
  public bool IsExpired => DateTime.Now > ExpirationDate;
  private DateTime ExpirationDate => CreateExpirationDate();
  private DateTime CreateExpirationDate()
  {
    var expireDate = DateTime.MinValue;
    if (ExpiresOn > 0)
    {
      expireDate = new DateTime(ExpiresOn);
    }
    else if (ExpiresIn > 0)
    {
      expireDate = _createDate.AddSeconds(ExpiresIn);
    }
    return expireDate;
  }
}
```

The JSON properties are the properties that are returned from the login request. The other properties are things I added to easily retrieve the token and whether or not it's expired. I do not know if there is any existing authenticator that does the same as my custom [`Office365Authenticator`](https://github.com/KenBonny/KenBonny.Office365Experiment/blob/master/KenBonny.Office365Access.Console/Office365Authenticator.cs). All information about these request can be found on the [Azure documentation page on client credentials](https://docs.microsoft.com/en-gb/azure/active-directory/develop/active-directory-protocols-oauth-service-to-service) and [the MSDN page on obtaining a token](https://msdn.microsoft.com/en-us/library/hh454950.aspx?f=255&MSPPError=-2147217396).

For the curious, you won't find the `clientSecret` anywhere in code or configuration. Also, the application/client id that I use in the `RestRequest` is by now also inactive and cannot be used anymore. Sorry hackers, you'll have to try harder than this.

Unfortunately, this brings me to the end of the code part. What needs to be done now is to grant the application access to the emails so that when I make a request, the graph API returns data instead of a 401 - Unauthorized. I'll write about that part next week.

The start of this puzzle began over at [get access page](https://developer.microsoft.com/en-us/graph/docs/concepts/auth_v2_service) on the graph documentation site. There are several different authorisation types that can be used. According to the [Get Started with the Outlook REST APIs page](https://docs.microsoft.com/en-us/outlook/rest/get-started) there are three:

1. [Implicit flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-implicit) mostly used for SPA's
2. [Client credentials flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-client-creds) mostly used for web or mobile applications
3. [Authorisation code flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-code) for applications that do not have end users

The implicit and client credentials flow are used for applications that have users who want to access their own mails. For example an app on my phone that fetches my mails and on another phone fetches that owners mails. The authorisation code flow is for back end services that needs to read and send emails. For example, I could set up an email address pay@mycompany.com where clients could send invoices to and they will be automatically added to the accountancy program.

The first thing I did was register the application in the [Microsoft App Registration Portal](https://apps.dev.microsoft.com). After I create an application, give it a name, tick the _Let us help you get started_ checkbox and select the _Service and Daemon App (app with no user interaction)_, I get redirected to the [OAuth client credentials flow documentation](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-client-creds). Use the link in the top right corner to skip this wizard, which for a service app is only useful to get to the OAuth client credentials flow documentation.

\[gallery ids="8644,8645,8646" type="rectangular"\]

After the wizard, I'm in the application management screen. Here I need to create a client secret to serve as the password to log in. You do that by clicking the _Generate New Password_ button. Make sure to save that password somewhere secure as it cannot be retrieved after it is shown once. There is no reveal password button anywhere. I can always create a new password, but experience has taught me that updating the password is troublesome to say the least.

A platform should only be set if you are making calls for other users, since I won't be doing that, I don't need that.

The final step here, is to set the permissions this application has. The _Delegated Permissions_ are for applications posing as a user (such as an email app on my phone). The _Application Permissions_ are for service apps like the one I am building now. These permissions can be edited later. Remember that removing permissions can lead to errors, always program defensively.

![04-app-permissions.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380787141/uSkvIhtMm.jpeg)

Now that I have a login and password... I mean, client id and secret and the permissions are set, I still need to do one more step. When I install an app on my phone, I need to grant it rights to access to all the permissions it's asking for. The same needs to be done with service applications. Instead of first login, I need to log into a special url to provide [admin consent](https://developer.microsoft.com/en-us/graph/docs/concepts/auth_v2_service).

The url looks like this:

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
?client_id=
&response_type=code
&response_mode=query
&redirect_uri=
&scope=
```
The base is always the same: [https://login.microsoftonline.com/common/oauth2/v2.0/authorize](https://login.microsoftonline.com/common/oauth2/v2.0/authorize). The `client_id` is the application id which you can find on the Microsoft App Registration Portal. The `response_type` is a hard coded value `code` as is the `response_mode` value `query`. The `redirect_uri` is not important in this scenario as it is needed for implicit flows or client credentials flows. The url in that case needs to be the same as the platform that is registered in the Microsoft App Registration Portal. The `scope` parameter needs to contain all the permissions that the application requests, separated by a comma or semi-colon. I'm not sure on that last part. [The Microsoft Graph permissions reference page](https://developer.microsoft.com/en-us/graph/docs/concepts/permissions_reference) has a nice overview of all the permissions that can be granted.

When I create the url for my test application, it looks like this:

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
?client_id=7755da74-961a-455e-9777-34e773a2c3dc
&response_type=code
&response_mode=query
&redirect_uri=http://localhost/myapp/
&scope=mail.read
```

This brings me by the only problem that I cannot seem to overcome. When I go to the admin consent url and log in with the correct personal Outlook account, I get an error.

![admin-consent-error](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380788663/f96mSam4M.jpeg)

Yet when I did this at work with an Office365 Outlook account, it succeeded without a hitch. I logged on with an administrator account, got an overview of the permissions and had a nice little button to accept the permission or decline. I have reached out to Microsoft via [Twitter](https://twitter.com/bonny_ken/status/937374905238261760), [Stack Overflow](https://stackoverflow.com/questions/47728476/cannot-authorise-api-to-access-consumer-outlook-account) and two of Microsofts [own](https://social.msdn.microsoft.com/Forums/office/en-US/61570f47-1ff1-4427-b4f2-b03a75e1dc28/cannot-authorise-api-to-access-consumer-outlook-account?forum=Office365forDevelopers) [forums](https://social.msdn.microsoft.com/Forums/en-US/a72f4798-6561-487f-a2dc-1800571e77eb/cannot-authorise-api-to-access-consumer-outlook-account?forum=appsforoffice), but have not gotten a solution to this. If anybody has a solution, let me know and I'll give you credit for finding it.

After pouring over all documentation, combining approaches described in both the outlook.office.com and graph.microsoft.com documentation and trying different combinations, me and a colleague found a way to access the emails. Hopefully somebody else will find this useful.

All the code from this article can be found in [a repository](https://github.com/KenBonny/KenBonny.Office365Experiment) on [my GitHub profile](https://github.com/KenBonny).
