---
layout: category
title: 03 Protecting Web APIs
---

Identity Service protects ASP.NET Core Web APIs. This makes it ideal as a security gateway service for APIs in a micro-services architecture.

## Sample Web API 

This article uses a Sample Web API to demonstrate the use of Identity Service. It is recommended that you install the Sample Web API and complete the tasks outlined in this article. You can install the Sample Web API in an AKS Linux cluster using a Helm chart as demonstrated in this link :

[Installing the Sample Web API](https://github.com/rcladmin/helm-charts/tree/master/identitysvc3-sample-web-api)

The source code for the sample ASP.NET Core Web API used for this article can be found at :

[GitHub : ASP.NET Core Sample Web API](https://github.com/rcl-identityserver/sample-web-api)

You will also need to install the [Sample MVC Client](https://github.com/rcladmin/helm-charts/tree/master/identitysvc3-sample-web-client) to complete the tasks in this article.

## Setting up APIs in Identity Service

Use the Identity Service dashboard to add an API. The sample API application uses the name **'petsapi'**.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/api/add.PNG "Register")

### API Scope

An API must have a scope for a web client to access it. In the sample API application, the scope is called **petsapi-full**.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/api/api-scope.PNG "Register")

### Adding the API Scope to the list of allowed Client Scopes

The API scope must now be added to the list of allowed Client Scopes for the client application to access the api. In the previous section : [Single Sign On](/documentation/category/02_sso.html), we added the sample client to Identity Service. We must now add the API Scope to the client using the dashboard. The image below illustrates how the API Scope was added to the existing list of Client Scopes.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/api/client_scope.PNG "Register")

### Accessing the API

Open the sample MVC Client application in a browser and click on the link in the front page to send a 'GET' request to the sample API. Ensure that the sample web API is running and is able to accept requests. 

The image below shows the response of the sample API application accessed from the Sample MVC Client.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/api/api_response.PNG "Register") 

## Explaining the Code for the Sample Web API

### Protecting an ASP.NET Core Web API

The Startup.CS class for the sample web API is shown below.

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.HttpOverrides;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace SampleApi
{
    public class Startup
    {
        public static string _Authority = string.Empty;
        public static string _Audience = string.Empty;

        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {
            _Authority = Configuration["Authority"];
            _Audience = Configuration["Audience"];

            services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(o =>
            {
                o.Authority = _Authority;
                o.Audience = _Audience;
            });

            services.AddMvc();
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
                context.Request.PathBase = new PathString("/sampleapi");
                return next();
            });

            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseAuthentication();

            app.UseMvc();
        }
    }
}
```

### Configuring JWT Bearer Authentication

The JWT Bearer authentication is configured in the following section of ConfigureServices.

```csharp
_Authority = Configuration["Authority"];
_Audience = Configuration["Audience"];

services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
.AddJwtBearer(o =>
{
    o.Authority = _Authority;
    o.Audience = _Audience;
});
```
The 'Authority' is the url of the Identity Service application. The 'Audience' is the name of the API that we set in the Identity Service dashboard. These values are stored in configuration as environment variables. The following is an example of the configuration values in the User Secrets.

```json
{
  "Authority": "https://rclappdev.eastus.cloudapp.azure.com/identitysvc",
  "Audience": "petsapi",
}
```

The following line in the 'Configure' method enables Authentication in your application.

```csharp
 app.UseAuthentication();
```

### Protecting the API Controller

In the sample web API application, the PetsController is decorated with the Authorize attribute to secure it. 

```csharp
using System.Collections.Generic;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using SampleApi.Models;

namespace SampleApi.Controllers
{
    [Authorize]
    [Route("api/[controller]")]
    public class PetsController : Controller
    {
        // GET: api/Pets
        [HttpGet]
        public List<Pet> GetAll()
        {
            List<Pet> lst = new List<Pet>
            {
                new Pet{Id = 1, name = "Rex", type="Dog"},
                new Pet{Id = 2, name = "Fluff", type="Cat"},
                new Pet{Id = 3, name = "Polly", type="Parrot"}
            };
            return lst;
        }
    }
}
```

## Explaining the code for the Sample MVC Web Client

### Accessing a protected API in a web client.

In the Home Controller in the sample web client application, the ProtectedApiPage action demonstrates how to make an authorized API request.

```csharp
public async Task<IActionResult> ProtectedApiPage()
{
    // Use IdentityModel to get the endpoints from Identity Service
    var disco = await DiscoveryClient.GetAsync(Startup._Authority);
    string m = string.Empty;
    if (disco.IsError)
    {
        ViewBag.ErrorMessage = disco.Error;
        return View("Error");
    }

    // Get the token from the Identity Service endpoint to access the protected API
    var tokenClient = new TokenClient(disco.TokenEndpoint, Startup._ClientId, Startup._ClientSecret);
    var tokenResponse = await tokenClient.RequestClientCredentialsAsync(Startup._ApiScope);
    if (tokenResponse.IsError)
    {
        ViewBag.ErrorMessage = tokenResponse.Error;
        return View("Error");
    }

    // Set the bearer token in the header of the API request
    var client = new HttpClient();
    client.SetBearerToken(tokenResponse.AccessToken);

   // Get the response for the API and store the content as a string
    var response = await client.GetAsync($"{Startup._ApiUrl}/api/pets");
    if (!response.IsSuccessStatusCode)
    {
        ViewBag.ErrorMessage = $"The server responded with {response.StatusCode}";
        return View("Error");
    }
    else
    {
        m = await response.Content.ReadAsStringAsync();
    }

    // Return the string API content as a viewbag to the view
    ViewBag.Pets = m;
    return View();
}
```

The IdentityModel NuGet package includes a client library that you can use to find the discovery endpoints of Identity Service.

![placeholder](https://raw.githubusercontent.com/rcl-identityserver/documentation/master/images/api/identity_model.png "Register") 

To access the API using the Client Credentials flow, a 'Client Id' and 'Client Secret' must be provided by the client in the request to the Identity Service token endpoint to acquire a token.

These 'Client Id' and 'Client Secret' were set in the Identity Service dashboard for the Client in the previous section : [Single Sign On](/documentation/category/02_sso.html). 

The 'API Url' is the endpoint for API requests. The 'API Scope' must be provided by the client and is checked by Identity Service to ensure that the client is authorized to access the API. In the section above, the API Scope was set in the Identity Service dashboard and added to the list of allowed Client Scopes.

The following is an example of the configuration values in the Sample Client application in the User Secrets. 

```json
{
  "Authority": "https://rclappdev.eastus.cloudapp.azure.com/identitysvc",
  "ClientId": "sample-client",
  "ClientSecret": "1234",
  "ApiUrl": "https://rclappdev.eastus.cloudapp.azure.com/sampleapi",
  "ApiScope": "petsapi-full"
}
```

A token is then provided by Identity Service from its endpoint to the client to access the API. The bearer token must be set in the header  by the client when it makes a request to the API. Once authorized, the API will provide a success response and send the JSON payload. If not authorized, the API will respond with a '401 Unauthorized'. 





