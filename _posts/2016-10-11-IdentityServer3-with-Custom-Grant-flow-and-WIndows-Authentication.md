---
layout: post
title: IdentityServer3 with Custom Grant flow and Windows Authentication
---

#### What we are going to do?

1. Host and configure `IdentityServer3`
2. Create client for `IdntSrv` with `Flows.Other` and `AllowedCustomGrantTypes: "windows"`
3. Create and configure `WindowsAuthenticationService` with enabled WindowsAuthentication to convert windows identity to `jwt` token
4. Create custom grant validator to validate tokens converted from `WindowsAuthenticationService`

#### What we will archive?

Our client (in my case desktop application) will call `WindowsAuthenticationService` for converting `windows identity` to `jwt token`
that is trusted by our `IdentityServer`. Then `Client` will call to `/token` endpoint (with custom grant: windows )
with newly getted `jwt token` and get authenticated in IdentityServer. After that user will have access to `id_token access_token`, scopes and whatever `IdentityServer3` supports.

<!--more-->

### 1. Create and configure `IdentityServer3`

Create a new empty asp.net web project. We are going to use OWIN-based web app hosted in IIS. 
For this we need to install `Owin.Host.SystemWeb` package.
Open Nuget-Package Manager Console and write following command:

`Install-Package Microsoft.Owin.Host.SystemWeb`

After this we need to install `IdentityServer3` with this command:

`Install-Package IdentityServer3`

Also we need `System.IdentityModel.Tokens.Jwt` to validate tokens issued from `WindowsAuthenticationService` in IdentityServer side.

`Install-Package System.IdentityModel.Tokens.Jwt`

Afther that we need our `Startup.cs` to configure our web app and `IdentityServer3`.
Right Click On Project > Add > OWIN Startup Class.

Don't forget to Enable SSL in Properties Menu (F4) and configure ProjectURL in Project Properties Web Tab.


### 2. Create client for `IdntSrv` with `Flows.Other` and `AllowedCustomGrantTypes: "windows"`

I will not go deeper for basics configuration of IdentityServer3 . Read more about basics of configuring IdentityServer3 in
[official documentation](https://identityserver.github.io/Documentation/docsv2/configuration/overview.html)

We should define our Client with `Flows.Custom` and `AllowedCustomGrantTypes: "windows"`. Read more about configuring Clients
in [official documentation](https://identityserver.github.io/Documentation/docsv2/configuration/clients.html)

![Custom Flow and AllowedCustomGrantTypes "windows"](http://i.imgur.com/EbpLjxy.png)


Also In my case all my users should come from windows authentication for that you should disable local login.

![Disable local login](http://i.imgur.com/mSirFpM.png)


### 3. Create and configure `WindowsAuthenticationService` with enabled WindowsAuthentication to convert windows identity to `jwt token`

Again Create a new empty asp.net web project, with `Startup.cs` and `Owin.Host.SystemWeb` nuget installed on it.

Then we need `WindowsAuthentication` Nuget for converting `win_identity` into `jwt token` to install this run this command:

`Install-Package IdentityServer.WindowsAuthentication`

Now we need to configure our application to use Windows Autehntication. 

For IIS Express ( F4 properties menu )

![IIS Express](http://i.imgur.com/FjcfTOr.png)


For IIS ( uncomment this line in web.config )

![IIS](http://i.imgur.com/L2QV1CJ.png)


Now we should configure `IdentityServer.WindowsAuthentication`. Here it is:

![WinAuth configuration](http://i.imgur.com/aB7HJm6.png)


Note to higlighthing lines. You should Load same Certificate as in IdentityServer, and you should enable OAuth Endpoint to 
be able to get converted token by Requesting custom grant from `WinAuthService`.

Don't forget to Enable SSL fom Properties Menu (F4) and configure ProjectURL in Project Properties Web Tab.


### 4. Create custom grant validator to validate tokens converted from `WindowsAuthenticationService`

Now the most important part is to teach our IdentityServer to understand "windows" grant type (:

For that we need to implement `ICustomGrantValidator` interface. Read more about Custom Grant Validators
in [official documentation](https://identityserver.github.io/Documentation/docsv2/advanced/customGrantTypes.html)

We should define what we need when we are going to grant access with "windows" grant type. We need our `win_token` that
our `WindowsAuthenticationService` issued. This service can be used with `IdentityServer3` for WS-Federation, but we aren't going to
use it now.

![Getting win_token from request](http://i.imgur.com/wpNTRMb.png)


After that we need to validate that token in `win_token` field is issued with our `WindowsAuthenticationService`. 

![Valdiation Token](http://i.imgur.com/VcTNGU4.png)


Now we have validated(trusted) token, now we should get `NameIdentifier` claim aka 'sub'. It includes Unique Identifier of user in Windows ( Active Directory ). It will be our `ProviderId` in `IdentityServer`.

![Getting NameIdentifier token](http://i.imgur.com/mgVeOoT.png)


Now we have everything to authenticate user in IdentityServer Side and return result to client.

![User Authentication](http://i.imgur.com/AVO9rLM.png)


You can find whole source code used in this post here.
It is working sample about what I was talking :)




