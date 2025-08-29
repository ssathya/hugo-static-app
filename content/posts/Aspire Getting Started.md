+++ 
date = '2025-08-28T16:53:30-04:00' 
draft = true 
title = 'Aspire Getting Started' 
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

1. The application displays the .NET Aspire dashboard in the browser. ![Aspire frontend](https://learn.microsoft.com/en-us/dotnet/aspire/docs/get-started/media/aspire-dashboard-webfrontend.png\#lightbox)

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

## Ensure.NET Aspire templates are installed

To develop .NET Aspire projects, we need to ensure that Aspire templates are installed on our computer. We can install and list the Aspire templates using the following commands:

```bash
dotnet new install Aspire.ProjectTemplates
```

This will install Aspire and we can check the installed packages and version using the following command:

```bash
dotnet new list aspire
```
