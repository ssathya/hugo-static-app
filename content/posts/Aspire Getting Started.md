+++
date = '2025-08-30T17:31:31-04:00'
draft = false
title = 'Aspire Getting Started'
tags = ['Programming']
categories = ['Aspire', 'Blazor']
+++


Microsoft has [excellent documentation](https://learn.microsoft.com/en-us/dotnet/aspire/get-started/build-your-first-aspire-app?tabs=windows&pivots=visual-studio) for getting started with Aspire, but I am preparing this document for my personal reference and easy access.

## Prerequisites

- .NET 9
  - Though .NET 8 will be suitable for Aspire, we'll go with version 9 for Aspire 9.x support.
- An OCI-compliant container runtime, such as Podman.
- Visual Studio 2022

## Create your first Aspire Project.

Start Visual Studio and create a new project. In the Create New Project template, select to create a new "Aspire Starter App". ![New project template](https://learn.microsoft.com/en-us/dotnet/aspire/docs/media/aspire-templates.png)

In the following window, select 'AspireSample' as the project name and click Next.

In the next dialog box, ensure .NET 9.0 is selected, ensure Redis for caching is checked, and select **Create**.

Visual Studio creates a new solution that is structured to use .NET Aspire.

## Testing the application

The sample application includes a frontend Blazor app that communicates with a Minimal API project. The API project is used to provide weather data to the frontend. The frontend app is configured to use service discovery to connect to the API project. The API project is configured to use output caching with Redis. The sample app is now ready for testing. When we deploy the application, we can verify the following conditions:

- Weather data is retrieved from the API project using service discovery and displayed on the weather page.
- Subsequent requests are handled via the output caching configured by the .NET Aspire Redis integration.

In Visual Studio, set the *AspireSample.AppHost* project as the startup project if it isn't set as startup project. When you start debugging/running the application:

1. The application displays the .NET Aspire dashboard in the browser. 
   
   ![](https://learn.microsoft.com/en-us/dotnet/aspire/docs/get-started/media/aspire-dashboard-webfrontend.png)

2. Launch the Blazor frontend by clicking on the *launch* button in Aspire dashboard.
   
   - Navigate from the home page to the weather page. Make a mental note of some of the values represented in the forecast table.

3. Continue occasionally refreshing the page. If it is refreshed within 10 seconds, the cached data is returned. Eventually, the cache data will expire, and a new set of values will be displayed.

Congratulations! You have created and run your first .NET Aspire solution! Below, we will outline the steps to migrate your existing solution to Aspire.

# Moving an existing .NET application to Aspire

If you have an existing microservice and .NET web app, you can add .NET Aspire to it and get all the included features and benefits.

## Get started

We will work on an existing application created by Microsoft for this tutorial. We'll use the following command to clone an existing repository:

```bash
git clone https://github.com/MicrosoftDocs/mslearn-dotnet-cloudnative-devops.git eShopLite
```

## Explore the sample application

- **Data Entities:** This project serves as an example class library. It defines the *Product* class used in the Web App and Web API.

- **Products:** This example Web API returns a list of products in the catalog, along with their properties.

- **Store:** This example Blazor Web App shows the product catalog to visitors on the website.

## Run the sample application

Load the solution in **<mark>Visual Studio</mark>** and in the *Solution Explorer* right-click the *eShopLite* solution, and then select *Configure Startup Projects.*

- Select *Multiple startup projects*.

- In the Action column, Select Start for both the Products and Store projects.

- Select *Ok*.

Start debugging the solution by pressing the {{<kbd>}}F5{{</kbd>}} key.

Two pages open in the browser:

- A page displays products in JSON format from a call to the Products Web API.

- A page displays the website's homepage. In the menu on the left, select **"Products"** to view the catalog retrieved from the Web API.

Regardless of the tool employed, initiating multiple projects manually or establishing the necessary connections between them can be a laborious and time-consuming task. Furthermore, the Store project mandates explicit configuration of endpoints for the Products API, which can be both cumbersome and susceptible to errors. In such scenarios, .NET Aspire offers an effective solution by simplifying and streamlining these processes.

## Ensure .NET Aspire templates are installed

To develop .NET Aspire projects, we need to ensure that Aspire templates are installed on our computer. We can install and list the Aspire templates using the following commands:

```bash
dotnet new install Aspire.ProjectTemplates
```

This will install Aspire, and we can check the installed packages and version using the following command:

```bash
dotnet new list aspire
```

## Add .NET Aspire to the Store web app

Now, let's enroll the Store project, which implements the web user interface, in .NET Aspire orchestration: 

1. In Visual Studio, in the Solution Explorer, right-click the Store project, select Add, and then select <mark>.NET Aspire Orchestrator Support</mark>.

2. In the Add .NET Aspire Orchestrator Support dialog, select Ok. ![](https://learn.microsoft.com/en-us/dotnet/aspire/docs/get-started/media/add-aspire-orchestrator-support.png) You should now have two new projects, both of which have been added to the solution.
   
   - **eShopLite.AppHost** is an orchestrator project designed to connect and configure the different projects and services of your application. The orchestrator is set as the *Startup project*, and it depends on the *eShopLite*.
   
   - **ServiceDefaults:** This shared project is used to manage configurations that are reused across an orchestrator project designed to connect and configure the various projects and services within *eShopLite.Store* project.
   
   - **eshopLite.ServiceDefaults:** This shared project is used to manage configurations that are reused across the projects in your solution related to resilience, service discovery, and telemetry.
   
   If you would examine the *AppHost.cs* file, you'll find the following line of code, which registers the Store project in the .NET Aspire orchestration:
   
   ```csharp
   builder.AddProject<Projects.Store>("store");
   ```

To add the *Products* project to the .NET Aspire:

1. In Visual Studio, in the Solution Explorer, right-click the Products project, select Add, and then select .NET Aspire Orchestrator Support.

2. A dialog indicating that the .NET Aspire Orchestrator project already exists, select OK. ![](https://learn.microsoft.com/en-us/dotnet/aspire/docs/get-started/media/orchestrator-already-added.png) In the eShopLite.AppHost project, open *AppHost.cs* file. Notice this line of code, which registers the Products project in the .NET Aspire orchestration:
   
   ```csharp
   builder.AddProject<Projects.Products>("products");
   ```

Also notice that the *eShopLite.AppHost* project, now depends on both the Store and Products projects.

### Service Discovery

At this point, both projects are part of .NET Aspire orchestration, but the Store project needs to rely on the Products backend address through .NET Aspire's service discovery. To enable service discovery, open the *AppHost.cs* file in the *eShopLite.AppHost* project and update the code so that the `builder `adds a reference to the *Products* project:

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var products = builder.AddProject<Projects.Products>("products");

builder.AddProject<Projects.Store>("store")
       .WithExternalHttpEndpoints()
       .WithReference(products)
       .WaitFor(products);

builder.Build().Run();
```

The preceding code expresses that the *Store* project depends on the *Products* project. This reference is used to discover the address of the *Products* project at run time. Additionally, the *Store* project is configured to use external HTTP endpoints. If you later choose to deploy this application, you'd need the call to `WithExternalHttpEndpoints `to ensure that it's public to the outside world. Finally, the `WaitFor` API ensures that the *Store* app waits for the *Products* app to be ready to serve requests.

Next, update the *appsettings.json* in the *Store* project with the following JSON:

```json
{
  "DetailedErrors": true,
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ProductEndpoint": "http://products",
  "ProductEndpointHttps": "https://products"
}
```

Change to note is `ProductEndpoint `and `ProductEndpointHttps`. The destination addresses were hardcoded, but now use the "products" name that was added to the orchestrator in the *AppHost*. These names are used to discover the address of the *Products* project.

<div class="alert is-info">
<p class="alert-title"><span class="docon docon-status-error-outline" aria-hidden="true"></span> Note</p>
<p>Notice that the <strong>eShopLite.AppHost</strong> project is the new startup project.</p>
</div>

## Explore the enrolled application

Let's start by examining the new behavior that .NET Aspire provides.

**Note**

*Notice that the **eShopLite.AppHost** project is the new statup project.*

1. In Visual Studio, to start debugging, press `F5` to build and launch the project.

2. If the Start Docker Desktop dialog appears, select Yes. Visual Studio starts the Docker engine and creates the necessary containers. When the deployment is complete, the .NET Aspire dashboard is displayed.

3. In the dashboard, select the endpoint for the *Products*  project.

4. In the dashboard, select the endpoint for the products project. A new browser tab appears and displays the product catalog in JSON format.

5. In the menu on the left, select Products. The product catalog is displayed.

6. To stop debugging, close the browser. 

Congratulations, you added .NET Aspire orchestration to your preexisting web app. You can now add .NET Aspire integrations and use the .NET Aspire tooling to streamline your cloud native web app development.
