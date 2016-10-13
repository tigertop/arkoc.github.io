---
layout: post
title: IdentityServer3 with custom grant flow and windows authentication
---

#### What we are going to do?

1. Host and configure `IdentityServer3`
2. Create a client for `IdentityServer3` with `Flows.Other` and `AllowedCustomGrantTypes: "windows"`
3. Host and configure `WindowsAuthenticationService`
4. Create custom grant validator in `IdentityServer3` for `windows` grant type
5. Create console client and get token from `IdentityServer3` by current `windows principal`.

#### How this all communicate?

![Communication](http://i.imgur.com/X86m1fK.png)

Our client (in my case desktop application) will call `WindowsAuthenticationService` for converting current `windows principal` to `jwt token` trusted by `IdentityServer3`.
Then `Client` will call to `/token` endpoint (with the custom grant: windows ) of `IdentityServer3` with providing `jwt token` got from `WindowsAuthenticationService`. `IdentityServer3` will issue new token with requested scopes, custom claims and so on.

#### What we will archive?

This will allow us to authenticate users in `IdentityServer3` with windows authentication. Also, we can add roles, claims to these users through [IdentityManager](https://github.com/IdentityManager/IdentityManager).

<!--more-->

### 1. Host and configure `IdentityServer3`

Create a new empty asp.net web project. We are going to use OWIN-based web app hosted in IIS.

Install following packages:

`Install-Package Microsoft.Owin.Host.SystemWeb`
`Install-Package IdentityServer3`

After that we need our `Startup.cs` to configure our web app and `IdentityServer3`.
Add `Startup.cs` file (Right Click On Project > Add > OWIN Startup Class) to configure our app to use `IdentityServer3`.

I will not go deeper for basics configuration of IdentityServer3 . Read more about basics of configuring IdentityServer3 in
[official documentation](https://identityserver.github.io/Documentation/docsv2/configuration/overview.html)

Don't forget to Enable SSL in Properties Menu (F4) and update ProjectURL in Project Properties: Web tab.


### 2. Create client for `IdentityServer3` with `Flows.Other` and `AllowedCustomGrantTypes: "windows"`

We should define our Client with `Flows.Custom` and `AllowedCustomGrantTypes: "windows"`. Read more about configuring Clients
in [official documentation](https://identityserver.github.io/Documentation/docsv2/configuration/clients.html)

![Custom Flow and AllowedCustomGrantTypes "windows"](http://i.imgur.com/EbpLjxy.png)

Also in my case all my users should come from windows authentication for achieving it I disabled local login.

![Disable local login](http://i.imgur.com/mSirFpM.png)


### 3. Host and configure `WindowsAuthenticationService`

Again Create a new empty asp.net web project, with `Startup.cs` and `Owin.Host.SystemWeb` nuget installed on it.

Then we need `WindowsAuthentication` package for issuing `jwt token`'s based on current `windows principal`.

`Install-Package IdentityServer.WindowsAuthentication`

Now we need to configure our application to use Windows Authentication. 

For IIS Express (F4 properties menu)

![IIS Express](http://i.imgur.com/FjcfTOr.png)


For IIS (uncomment this line in web.config)

![IIS](http://i.imgur.com/L2QV1CJ.png)


Now we should configure `IdentityServer.WindowsAuthentication`. Here it is:

![WinAuth configuration](http://i.imgur.com/aB7HJm6.png)


Note to highlighting lines. You should Load the same Certificate as in IdentityServer, and you should set `EnableOAuth2Endpoint` true, to get `jwt token` by requesting custom grant from `WindowsAuthenticationService`'s `/token` endpoint.

Again don't forget to Enable SSL in Properties Menu (F4) and update ProjectURL in Project Properties: Web tab.


### 4. Create custom grant validator in `IdentityServere3` for `windows` grant type

Now the most important part is to teach our `IdentityServer3` to understand `windows` grant type.

For that, we need to implement `ICustomGrantValidator` interface. Read more about Custom Grant Validators
in [official documentation](https://identityserver.github.io/Documentation/docsv2/advanced/customGrantTypes.html)

We should define what we need in the custom grant request to be passed when we are granting access to `windows` grant type. We need `jwt token` that is issued by `WindowsAuthenticationService`. We will set/get it as `win_token` param.

![Getting win_token from request](http://i.imgur.com/wpNTRMb.png)


After that, we need to validate that `win_token` is issued from our `WindowsAuthenticationService`. 

![Token Validation](http://i.imgur.com/VcTNGU4.png)


Now we have validated (trusted) token. Next step is to get unique identifier of user. It is stored in `NameIdentifier` claim. 

![Getting NameIdentifier token](http://i.imgur.com/mgVeOoT.png)


Everything ready to authenticate the user in `IdentityServer3` and issue token to the client.

![User Authentication](http://i.imgur.com/AVO9rLM.png)


### 5. Create console client and get token from `IdentityServer3` by current `windows principal`.

Everything setup, now we should test our services. Create new `ConsoleApplication` and install `Thinktecture.IdentityModel.Client` package.

`Install-Package Thinktecture.IdentityModel.Client`

Now We need to connect to `WindowsAuthenticationService` with `UseDefaultCrediantals` (this will include our windows principal in request) and get converted `jwt token` back.

![Getting win token from request](http://i.imgur.com/PFsSQRB.png)

After that we should get access token from result (`resultToken.AccessToken`) and send it via `win_token` param by requesting custom `windows` grant from `IdnetityServer3`.

![Requesting custom windows grant](http://i.imgur.com/P6p37Ff.png)

We Done!

A whole working sample can be found here: [IdSrv3.WindowsCustomGrant.Sample]().


#### Links

[Documentation](https://identityserver.github.io/Documentation/)

[IdentityServer3](https://github.com/IdentityServer/IdentityServer3)

[WindowsAuthenticationService](https://github.com/IdentityServer/WindowsAuthentication)

[IdentityManager](https://github.com/IdentityManager/IdentityManager)

