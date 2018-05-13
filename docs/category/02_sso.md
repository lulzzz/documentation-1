---
layout: category
title: 02 Single Sign On
---

Identity Service provides single-sign-on (SSO) for ASP.NET Core applications using Open ID connect.

## Sample Client

This article uses a Sample Web Client to demonstrate the use of Identity Service. It is recommended that you install the Sample Client and complete the tasks outlined in this article. You can install the Sample Client in an AKS Linux cluster using a Helm chart as demonstrated in this link :

[Installing the Sample MVC Client](https://github.com/rcladmin/helm-charts/tree/master/identitysvc3-sample-web-client)

The source code for the sample ASP.NET Core MVC client used for this article can be found at :

[GitHub : ASP.NET Core Sample MVC Client](https://github.com/rcl-identityserver/sample-web-client)

## Client Setup in Identity Service

A client is a web application that you want to add SSO. In this case, we will add sign on to the Sample Web Client. In Identity Service, use the dashboard to add a client. The Client Id to be used for the sample application is 'sample-client'

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/add.PNG "Add Client")

### Add a Client Secret

In the Identity Service dashboard, add a client secret. The secret will be hashed, so please write down the actual value to be used for your web application. The secret used for the sample application is '1234'

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/secrets.PNG "Add Client")

### Add a Sign In Uri

The Sign In Uri is the url of your Client Web application with '/signin-oidc' appended at the end.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/signin-url.PNG "Add Client")

### Add a Sign Out Uri

The Sign Out Uri is the url of your Client Web application with '/signout-callback-oidc' appended at the end.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/signout-url.PNG "Add Client")

## Test the Sample Client Web Application

Open the Sample Client Web application in a browser and test the single sign on provided by Identity Service.

### Register for a new account

* Click on the register link at the top of the Client Web application

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/register.PNG "Register Link")

* The user registers on Identity Service

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/register_form.PNG "Register")

* The user confirms their email on Identity Service

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/email_confirm.PNG "Register")

### Login

* Click the Login link in the Client Web application.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/login_link.PNG "Register")

* The user logs in with Identity Service

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/login-form.PNG "Login")

* The user is authenticated and is returned to the Client Web application

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/login_return.PNG "Login Return")

## Explaining the code in the Sample Client Web application

We will explain the code for the Sample Client Web application. You can use this code in your own ASP.NET Core applications to add single sign on with Identity Service.

### Authorization Configuration

The  Startup.CS file in the sample Client Web application is as follows :

```csharp
using System.Collections.Generic;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;
using System.IdentityModel.Tokens.Jwt;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.HttpOverrides;

namespace SampleClient
{
    public class Startup
    {
        public static string _Authority = string.Empty;
        public static string _ClientId = string.Empty;
        public static string _ClientSecret = string.Empty;
        public static string _ApiUrl = string.Empty;
        public static string _ApiScope = string.Empty;

        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {
            _Authority = Configuration["Authority"];
            _ClientId = Configuration["ClientId"];
            _ClientSecret = Configuration["ClientSecret"];
            _ApiUrl = Configuration["ApiUrl"];
            _ApiScope = Configuration["ApiScope"];
           
            services.AddAuthentication(options =>
            {
                options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
                options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
            })
           .AddCookie()
           .AddOpenIdConnect(o =>
           {
               o.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
               o.ResponseType = OpenIdConnectResponseType.CodeIdToken;
               o.Authority = _Authority;
               o.ClientId =  _ClientId;
               o.ClientSecret =  _ClientSecret;
               o.SaveTokens = true;
               o.SecurityTokenValidator = new JwtSecurityTokenHandler
               {
                   InboundClaimTypeMap = new Dictionary<string, string>()
               };
               o.TokenValidationParameters.NameClaimType = "email";
               o.GetClaimsFromUserInfoEndpoint = true;
               o.ClaimActions.MapJsonKey("Contributor", "Contributor");
           });

            services.AddMvc();

            services.AddAuthorization(options =>
            {
                options.AddPolicy("Contributor", policy => policy.RequireClaim("Contributor", "True"));
            });

            services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();
            services.AddTransient<INavigationService, NavigationService>();
        }

        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            var forwardingOptions = new ForwardedHeadersOptions()
            {
                ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
            };
            forwardingOptions.KnownNetworks.Clear();
            forwardingOptions.KnownProxies.Clear();
            app.UseForwardedHeaders(forwardingOptions);
            app.Use((context, next) =>
            {
                context.Request.Scheme = "https";
                return next();
            });
            app.UsePathBase("/sampleclient");
            if (env.IsDevelopment())
            {
                app.UseBrowserLink();
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
            }

            app.UseStaticFiles();

            app.UseAuthentication();

            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });
        }
    }
}
```

The authorization service is configured in the following section :

```csharp
services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(o =>
{
    o.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    o.ResponseType = OpenIdConnectResponseType.CodeIdToken;
    o.Authority = _Authority;
    o.ClientId =  _ClientId;
    o.ClientSecret =  _ClientSecret;
    o.SaveTokens = true;
    o.SecurityTokenValidator = new JwtSecurityTokenHandler
    {
        InboundClaimTypeMap = new Dictionary<string, string>()
    };
    o.TokenValidationParameters.NameClaimType = "email";
    o.GetClaimsFromUserInfoEndpoint = true;
    o.ClaimActions.MapJsonKey("Contributor", "Contributor");
});

services.AddMvc();

services.AddAuthorization(options =>
{
    options.AddPolicy("Contributor", policy => policy.RequireClaim("Contributor", "True"));
});
```

The OpenIdConnect authorization is configured. The **Client ID** and **Client Secret** are stored in configuration as environment variables. The values are the same as what we set earlier in the Identity Service dashboard. The **Authority** is the url of the Identity Service application. The following is an example of the configuration values from the User Secrets. At this point, we can ignore the ApiUrl and ApiScope variables.

```json
{
  "Authority": "https://rclappdev.eastus.cloudapp.azure.com/identitysvc",
  "ClientId": "sample-client",
  "ClientSecret": "1234",
  "ApiUrl": "https://rclappdev.eastus.cloudapp.azure.com/sampleapi",
  "ApiScope": "petsapi-full"
}
```

The following line in the 'Configure' method enables Authentication in your application.

```csharp
 app.UseAuthentication();
```

### The Account Controller

 The **Account controller** in the Controllers folder, is used to establish the login and logout actions in your application.

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.AspNetCore.Authorization;

namespace SellingService.Controllers
{
    public class AccountController : Controller
    {
        public async Task Logout()
        {
            await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            await HttpContext.SignOutAsync(OpenIdConnectDefaults.AuthenticationScheme);
        }

        [Authorize]      
        public IActionResult Login(string ReturnUrl)
        {
            if(!string.IsNullOrEmpty(ReturnUrl))
            {
                return Redirect(ReturnUrl);
            }
            return RedirectToAction("Index", "Home", null);
        }

        public IActionResult AccessDenied(string ReturnUrl = null)
        {
            return View();
        }
    }
}
```

### Login / Log out links

The **_LoginPartialView** in the Views > Shared folder provides the login and logout links.

```html
@model String
@{

    string registerUrl = $"{Startup._Authority}/Account/Register?returnUrl={Model}";
}
@if (User.Identity.IsAuthenticated == true)
{
    <form asp-area="" asp-controller="Account" asp-action="Logout" method="post" id="logoutForm" class="navbar-right">
        <ul class="nav navbar-nav navbar-right">
            <li>
                <a href="#">@User.Identity.Name</a>
            </li>
            <li>
                <button type="submit" class="btn btn-link navbar-btn navbar-link">Logout</button>
            </li>
        </ul>
    </form>
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li><a href="@registerUrl">Register</a></li>
        <li><a asp-area="" asp-controller="Account" asp-action="Login" asp-route-ReturnUrl="@Model">Log In</a></li>
    </ul>
}
```

The _LoginPartialView is added to the **_Layout** page to display the login links in the top menu.

```html
@inject INavigationService NavService

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Sample Client</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <nav class="navbar navbar-default navbar-fixed-top">
        <div class="container">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="sr-only">Toggle navigation</span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                <a asp-area="" asp-controller="Home" asp-action="Index" class="navbar-brand">Sample Client</a>
            </div>
            <div class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li><a asp-area="" asp-controller="Home" asp-action="Index">Home</a></li>
                </ul>
                @await Html.PartialAsync("_LoginPartial", NavService.GetAbsoluteUrl());
            </div>
        </div>
    </nav>
    <div class="container body-content">
        @RenderBody()
        <hr />
        <footer>
            <p>&copy; 2018 - Sample Client</p>
        </footer>
    </div>
        <script src="~/lib/jquery/dist/jquery.js"></script>
        <script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>
    @RenderSection("Scripts", required: false)
</body>
</html>
```

This line of code is responsible for displaying the login / logout links.

```csharp
@await Html.PartialAsync("_LoginPartial", NavService.GetAbsoluteUrl());
```

You will notice that we injected a 'NavigationService' in the _Layout view, the reason for this is to return the user to your application's current absolute url after they login (or register a new account) in Identity Service. The code files for the NavigationService are shown below.

### INavigationService Interface in the Services folder

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace SampleClient
{
    public interface INavigationService
    {
        string GetAbsoluteUrl();
    }
}
```
### NavigationService Class in the Services folder

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Http;

namespace SampleClient
{
    public class NavigationService : INavigationService
    {
        private readonly IHttpContextAccessor _request;

        public NavigationService(IHttpContextAccessor request)
        {
            _request = request;
        }

        public string GetAbsoluteUrl()
        {
            return _request.HttpContext.Request.GetAbsoluteUrl();
        }
    }
}
```

### HttpRequestExtension in the Extensions folder

This is used to extend 'HttpContext.Request' in the 'NavigationService' class to get the current absolute url.

```csharp
using Microsoft.AspNetCore.Http;

namespace SampleClient
{
    public static class HttpRequestExtensions
    {
        public static string GetAbsoluteUrl(this HttpRequest request)
        {
            var httpContext = request.HttpContext;
            string pathBase = httpContext.Request?.PathBase ?? string.Empty;
            string path = httpContext.Request?.Path ?? string.Empty;
            string qrystr = httpContext.Request?.QueryString.Value ?? string.Empty;
            return $"{httpContext.Request.Scheme}://{httpContext.Request.Host}{pathBase}{path}{qrystr}";
        }
    }
}
```

### NavigationService is injected into the application

```csharp
services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();
services.AddTransient<INavigationService, NavigationService>();
```

## Page Authorization

You can use role claims to authorize a user to access certain protected pages in a Client application, such as an 'Admin" page.

### Create a role claim in Identity Service

Add a role using the Identity Service dashboard. The sample Client Web application uses a "Contributors" role.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/role_add.PNG "Register")

### Add a role claim

The role claims will be used to secure a web page. In the sample Client Web application, only a user with a 'Contributor' claim type with a claim value of 'True' will be able to access a protected 'Contributors' page.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/role_claim.PNG "Register")

### Add a user to a role

Users in a role will inherit the claims specified for that role. For instance, 'anil.ripla@gmail.com' will inherit the 'Contributor' role claim.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/role_user.PNG "Register")

### Test the Sample Client Web Application

Open the Sample Client Web application in a browser and test the following operations. Access a role protected page in the application by clicking the link on the front page. A user in the 'Contributors' role will be able to access the page as shown below.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/role-protected-page.PNG "Register")

Unauthorized users will be directed to an Access Denied page in the application.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/client/access_denied.PNG "Register")

## Explaining the code in the Sample Client Web application

### Role authorizations

In the Startup.CS file in the sample Client Web application, there is a security policy that requires the claim value of 'Contributor' with a value of 'True'. We previously set up this role claim in the Identity Service dashboard.

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy("Contributor", policy => policy.RequireClaim("Contributor", "True"));
});
```

### Securing a page

In the Home controller in the sample Client Web application, the action 'ProtectedRolePage' is decorated with the Authorize attribute with the 'Contributor' policy we set in the Startup.CS file. In this way, only users with the 'Contributor' claim type will be able to access this ProtectedRolePage (see this page code in the Views > Home folder).


```csharp
[Authorize(Policy = "Contributor")]
 public IActionResult ProtectedRolePage() 
 {
    return View();
 }
```

In the Account controller, an AccessDenied controller is added to redirect unauthorized users to the Access Denied page. The Access Denied page is in the Views > Account folder.

```csharp
public IActionResult AccessDenied(string ReturnUrl = null)
{
    return View();
}
```



















