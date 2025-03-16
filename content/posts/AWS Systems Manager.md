+++
date = '2025-03-15T11:30:50-04:00'
draft = false
title = 'Using AWS Systems Manager in a docker image'
tags = ['programming', 'aws']
categories = ['dot-net', 'aws']
+++

I had issues using AWS Systems Manager in a docker container; I wanted to pass my login credentials via environment variables. Systems Manager does not use environment variables like AWS_ACCESS_KEY_ID,  AWS_SECRET_ACCESS_KEY, etc. and I didn't see any examples of how people use Systems Manager within a docker image.  

This is the way I solved it and yes it works!
```cs
using Amazon.Extensions.NETCore.Setup;
using Amazon.Runtime;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace CloudConsole.Services;

public static class ServiceBuilder
{
    private static IConfiguration? Configuration;

    internal static IServiceCollection ConfigureServices(string applicationName)
    {
        IServiceCollection services = new ServiceCollection();
        Configuration = BuildConfiguration(applicationName);
        services.AddSingleton(Configuration);
        return services;
    }

    private static IConfiguration BuildConfiguration(string applicationName)
    {
        var allEnvVariables = Environment.GetEnvironmentVariables();
        //var region = allEnvVariables["AWS_DEFAULT_REGION"]?.ToString() ?? string.Empty;
        var accessKey = allEnvVariables["AWS_ACCESS_KEY_ID"]?.ToString() ?? string.Empty;
        var secretKey = allEnvVariables["AWS_SECRET_ACCESS_KEY"]?.ToString() ?? string.Empty;

        var awsCredentials = new BasicAWSCredentials(accessKey, secretKey);
        AWSOptions aWSOptions = new()
        {
            Region = Amazon.RegionEndpoint.USEast1,
            Credentials = awsCredentials
        };
        IConfigurationBuilder builder = new ConfigurationBuilder();
        builder.SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
            .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Production"}.json"
                , optional: true, reloadOnChange: true)
            .AddEnvironmentVariables()
            .AddSystemsManager(@"/" + applicationName + @"/", aWSOptions, TimeSpan.FromMinutes(5))
            .AddSystemsManager("""/PortfolioManager/""", aWSOptions);
        Configuration = builder.Build();
        return Configuration;
    }
}
```

Create an AWSOptions instance and use this instance when you call AddSystemsManager.

And when I run my image I issue the following command:
```sh
docker run --name test_container1 -e AWS_ACCESS_KEY_ID=AccessKey -e AWS_SECRET_ACCESS_KEY=SecretKey test_image3
```
