# Blazor Docker

This repo contains working examples for containerizing Blazor applications.  There are three types of Blazor projects.

**Hosted** - A Blazor WebAssembly client app hosted on nginx and ASP.NET Core server app hosted in ASP.NET Core. 

    - Created with `dotnet new blazorwasm --hosted -o . -n BlazorHosted`

**Server** - A Blazor Server app hosted by ASP.NET Core.

    - Created with `dotnet new blazorserver -o . -n BlazorServer`

**Standalone** - A Blazor WebAssembly client app hosted on nginx.

    - Created with `dotnet new blazorwasm -o . -n BlazorStandalone`


## How to Run

Run `docker-compose up --build` from Hosted, Server, or Standalone folders to run each version.

## Hosted

Blazor WebAssembly app hosted on ASP.NET Core.  It is broken up into three projects "Client", "Server", and "Shared".

- docker-compose.yml - builds and runs both the Client and the Server projects.  


### How to run

Run `docker-compose up --build` from Hosted folder.  Open browser and go to http://localhost:5080/

### Client - uses nginx on alpine base image.
 - /Pages/FetchData.razor - includes the following code to show you what endpoint is being hit.
    ```csharp
    @using Microsoft.Extensions.Configuration
    @inject IConfiguration Configuration

    <h1>Weather forecast</h1>
    <p>SERVER_HOST: @Configuration["SERVER_HOST"]</p>
    ```
 - /wwwroot/appsettings.json - includes SERVER_HOST setting for production or when running in a container.  You'll want to change this to match your production endpoint.
 - /wwwroot/appsettings.Development.json - includes SERVER_HOST setting for development machine.
 - /Dockerfile - builds, publishes, and uses nginx to host it.
 - /nginx.conf - the nginx.conf file needed to serve the site.
 - /Program.cs - includes the following code to read the SERVER_HOST setting from appsettings.json

    ```csharp
    var serverHost = string.IsNullOrEmpty(builder.Configuration["SERVER_HOST"]) ?
        builder.HostEnvironment.BaseAddress :
        builder.Configuration["SERVER_HOST"];

        builder.Services.AddTransient(sp => new HttpClient { BaseAddress = new Uri(serverHost) });
    ```

### Server - uses ASP.NET runtime image

- /Dockerfile - builds, and starts ASP.NET app
- /Startup.cs - includes code to enable Cors, so the client app can call from a different host.

    ```csharp
    readonly string CorsOrigins = "CorsOrigins";

    services.AddCors(options =>
           {
               options.AddPolicy(CorsOrigins,
                   builder => builder.AllowAnyOrigin()
                   .AllowAnyMethod()
                   .AllowAnyHeader());
           });

    app.UseCors(CorsOrigins);
    ```
## Server

### How to run

Run `docker-compose up --build` from Server folder.  Open browser and go to http://localhost:6080/

### Server - Uses ASP.NET Core runtime host

- /Dockerfile - builds, publishes and starts the ASP.NET site.

## Standalone

### How to run

Run `docker-compose up --build` from Standalone folder.  Open browser and go to http://localhost:7080/

### Standalone - uses nginx on alpine base image.

 - /Dockerfile - builds, publishes, and uses nginx to host it.
 - /nginx.conf - the nginx.conf file needed to serve the site.