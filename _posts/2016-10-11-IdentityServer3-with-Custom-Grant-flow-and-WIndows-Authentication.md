---
layout: post
title: IdentityServer3 with Custom Grant flow and Windows Authentication
---

#### What we are going to do?

1. Create and configure `IdentityServer3`
2. Create client for `IdntSrv` with `Flows.Other` and `AllowedCustomGrantTypes: "windows"`
3. Create and configure `WindowsAuthenticationServic`e with enabled WindowsAuthentication to convert windows identity to `jwt`
4. Create custom grant validator to validate tokens converted from `WindowsAuthenticationService`

#### What we will archive?

Our client (in my case desktop application) will call `WindowsAuthenticationService` for converting `windows identity` to `jwt token`
that is trusted by our `IdentityServer`. Then `Client` will call to `/token` endpoint (with custom grant: windows )
with newly getted `jwt token` and get authenticated in IdentityServer. After that user will have access to `id_token access_token`, scopes
and whatever `IdentityServer3` supports.

### 1. Create and configure `IdentityServer3`

Create a new empty asp.net web project. We are going to use OWIN-based web app hosted in IIS. 
For this we need to install `Owin.Host.SystemWeb` package.
Open Nuget-Package Manager Console and write following command:

`Install-Package Microsoft.Owin.Host.SystemWeb`

After this we need to install IdentityServer3 with this command:

`Install-Package IdentityServer3`

Afther that we need our `Startup.cs` to configure our web app and `IdentityServer3`.
Right Click On Project > Add > OWIN Startup Class.

I will not go deeper for configuring IdentityServer3 basics. You can read about that in documentation pagexx
Also you can get full source code from this sample ( i will post github link in end of the post )

We should define our Client with `Flows.Custom` and `AllowedCustomGrantTypes: "windows"`. 
![Custom Flow and AllowedCustomGrantTypes "windows"](http://i.imgur.com/EbpLjxy.png)


Also In my case all my users should come from windows authentication for that you should disable local login.
![Disable local login](http://i.imgur.com/mSirFpM.png)

Af




