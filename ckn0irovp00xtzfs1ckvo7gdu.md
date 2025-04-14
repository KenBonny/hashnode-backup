---
title: "Connecting to Office365 by API"
datePublished: Mon May 21 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0irovp00xtzfs1ckvo7gdu
slug: connecting-to-office365-by-api-the-setup
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380798930/8PvT_LCjv.jpeg
tags: csharp, authentication, authorization, dotnetcore

---


Last week, I wrote about the program that will [read the emails from my Outlook account](http://kenbonny.net/2018/05/14/connecting-to-office365-by-api-the-code/). This week I'll grant the application read rights so that it can actually read the emails.

The start of this puzzle began over at [get access page](https://developer.microsoft.com/en-us/graph/docs/concepts/auth_v2_service) on the graph documentation site. There are several different authorisation types that can be used. According to the [Get Started with the Outlook REST APIs page](https://docs.microsoft.com/en-us/outlook/rest/get-started) there are three:

1. [Implicit flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-implicit) mostly used for [SPA](https://en.wikipedia.org/wiki/Single-page_application)'s
2. [Client credentials flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-client-creds) mostly used for web or mobile applications
3. [Authorisation code flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-code) for applications that do not have end users

The implicit and client credentials flow are used for applications that have users who want to access their own mails. For example an app on my phone that fetches my mails and on another phone fetches that owners mails. The authorisation code flow is for back end services that needs to read and send emails. That is the authorisation flow I'm interested in as I won't have end users. Just a single account the program should be able to access.

The first thing I did was register the application in the [Microsoft App Registration Portal](https://apps.dev.microsoft.com). After I create an application, give it a name, tick the _Let us help you get started_ checkbox and select the _Service and Daemon App (app with no user interaction)_, I get redirected to the [OAuth client credentials flow documentation](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-client-creds). Use the link in the top right corner to skip this wizard, which for a service app is only useful to get to the OAuth client credentials flow documentation.

\[gallery ids="8644,8645,8646" type="rectangular"\]

After the wizard, I'm in the application management screen. Here I need to create a client secret to serve as the password to log in. You do that by clicking the _Generate New Password_ button. Make sure to save that password somewhere secure as it cannot be retrieved after it is shown once. There is no reveal password button anywhere. I can always create a new password, but experience has taught me that updating the password is troublesome to say the least.

> May I suggest saving passwords in a password manager. I highly recommend [1Password](https://1password.com/). I've been using this myself for years now and I am very happy with their service.

A platform should only be set if you are making calls for other users, since I won't be doing that, I don't need that.

The final step here, is to set the permissions this application needs. The _Delegated Permissions_ are for applications posing as a user. The _Application Permissions_ are for service apps like the one I am building now. These permissions can be edited later. Remember that removing permissions can lead to errors, always program defensively.

![04-app-permissions.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380794701/Oc9L9dISI.jpeg)

Now that I have a login and password... I mean, application id, secret and the permissions are set, there is still one more step. When I install an app on my phone, I need to grant the app rights so it can access everything it needs to function properly. The same needs to be done with service applications. Instead of first login, I need to log into a special URL to provide [admin consent](https://developer.microsoft.com/en-us/graph/docs/concepts/auth_v2_service).

The URL looks like this:

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
?client_id=
&response_type=code
&response_mode=query
&redirect_uri=
&scope=
```

The base is always the same: [https://login.microsoftonline.com/common/oauth2/v2.0/authorize](https://login.microsoftonline.com/common/oauth2/v2.0/authorize). The `client_id` is the application id which I can find on the Microsoft App Registration Portal. The `response_type` is a hard coded value `code` as is the `response_mode` value `query`. The `redirect_uri` is not important in this scenario as it is needed for implicit or client credentials flows. The URL in that case needs to be the same as the platform that is registered in the Microsoft App Registration Portal. The `scope` parameter needs to contain all the permissions that the application requests, separated by a comma or semi-colon. I'm not sure on that last part, the documentation is quite vague on this. [The Microsoft Graph permissions reference page](https://developer.microsoft.com/en-us/graph/docs/concepts/permissions_reference) has a nice overview of all the permissions that can be granted.

When I create the URL for my test application, it looks like this:

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
?client_id=7755da74-961a-455e-9777-34e773a2c3dc
&response_type=code
&response_mode=query
&redirect_uri=http://localhost/myapp/
&scope=mail.read
```

This brings me to the only problem that I cannot seem to overcome. When I go to the admin consent URL and log in with the correct personal Outlook account, I get an error.

![admin-consent-error](https://cdn.hashnode.com/res/hashnode/image/upload/v1617380796238/6Qz8r96yb.jpeg)

Yet when I did this at work with an Office365 Outlook account, it succeeded without a hitch. I logged on with an administrator account, got an overview of the permissions and had a nice little button to accept or decline the permissions. I have reached out to Microsoft via [Twitter](https://twitter.com/bonny_ken/status/937374905238261760), [Stack Overflow](https://stackoverflow.com/questions/47728476/cannot-authorise-api-to-access-consumer-outlook-account) and two of Microsofts [own](https://social.msdn.microsoft.com/Forums/office/en-US/61570f47-1ff1-4427-b4f2-b03a75e1dc28/cannot-authorise-api-to-access-consumer-outlook-account?forum=Office365forDevelopers) [forums](https://social.msdn.microsoft.com/Forums/en-US/a72f4798-6561-487f-a2dc-1800571e77eb/cannot-authorise-api-to-access-consumer-outlook-account?forum=appsforoffice), but have not gotten a solution to this. If anybody has a solution, [let me know](http://kenbonny.net/contact/).

After pouring over all documentation, combining approaches described in both the outlook.office.com and graph.microsoft.com documentation and trying different approaches, me and a colleague found a way to access the emails. Hopefully somebody else will find this useful.
