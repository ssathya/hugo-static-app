+++
date = '2025-03-15T11:28:39-04:00'
draft = false
title = 'Blazor on RP'
tags = ['hosting']
categories = ['dot-net', 'hosting']
+++

## Introduction
I have always been interested in creating a blog using Blazor, a web framework that allows me to write C# code for both the client and the server side. I also wanted to host it on a Raspberry Pi, a low-cost and portable device that can run Linux and .NET applications. In this post, I will share how I configured my Blazing Blog, the name I gave to my project, on my Raspberry Pi.

## How are the executable(s) built?
I attempted to use DuckDNS as a dynamic DNS service, but it required me to configure my router settings, which I was not comfortable with. Therefore, I decided to look for another solution for hosting my Blazing Blog website. I wanted to deploy my blog as a docker container, but I faced a challenge in creating a Dockerfile that would work on both Intel and Raspberry Pi architectures. As a result, I opted to install .Net SDK on Raspberry Pi and compile my application using the following commands:

```
dotnet build release
```


## Enviromnet variables
One of the common problems that developers face when deploying their applications on Raspberry Pi devices is the configuration of environment variables. Environment variables are key-value pairs that store information such as paths, credentials, settings, etc. that are specific to the operating system or the application. When we develop our applications in Visual Studio, we often rely on its built-in features to manage the environment variables for us, without paying much attention to how they are set or where they are stored. However, when we try to run our applications on a new Raspberry Pi device, we may encounter errors or unexpected behaviors because the device does not have the same environment variables that we used in Visual Studio. Therefore, it is important to understand how to properly configure the environment variables for our applications on Raspberry Pi devices, and how to troubleshoot any issues that may arise from them.
Here are a few challenges and how I resolved it:

### Kestrel 
 Kestrel is a lightweight web server that can run on its own without a reverse proxy like Nginx. I chose to use Kestrel for my Blazing Blog project, which is hosted on a Raspberry Pi and has a very low traffic volume (no more than a handful concurrent users). To make Kestrel accessible from other machines, I had set enviroment 
variable ASPNETCORE_URLS as follows:

```
export ASPNETCORE_URLS="http://0.0.0.0:5001;https://0.0.0.0:5002"
```
One way to configure the ASP.NET Core application to listen on all network interfaces is to set the ASPNETCORE_URLS environment variable to the value of 0.0.0.0. This means that the application will accept requests from any IP address. 

### AWS credentials 
Blazing Blog is a web application that uses AWS S3 and System Manager's parameter store to manage its data. It stores the database connection string in the parameter store and uses it to connect to the database. It also uploads and downloads images from S3 buckets. To ensure security and limit the scope of access, a new IAM user BlazingBlogUser was created with permissions to only access S3 and System Manager. The access keys of this user were saved in a secure location and used by the application to authenticate with AWS.
### Self signed SSL
As Blazing Blog is a intranet application we are going to use a Self signed SSL certificate to make browsers happy! Self signed SSL certificates are a type of digital certificates that are created and signed by the owner of the website or server, rather than by a trusted third-party authority. They are often used for testing or development purposes, or for internal networks where trust is not an issue. However, self signed SSL certificates are not recommended for public-facing websites or services, as they can trigger security warnings in browsers and clients, and expose users to potential man-in-the-middle attacks.

The SSL was created using the following commands:
```
dotnet dev-certs https -ep $env:USERPROFILE\.aspnet\https\aspnetapp.pfx -p crypticpassword
dotnet dev-certs https --trust
```
This creates a file $HOME/.aspnet/https/aspnetapp.pfx and we set the following two envirornment variables to use this certificate.
```
export ASPNETCORE_Kestrel__Certificates__Default__Password=MySecretPassword
export ASPNETCORE_Kestrel__Certificates__Default__Path=/home/sridh/.aspnet/https/aspnetapp.pfx
```
### Putting all together
```
export ASPNETCORE_URLS="http://0.0.0.0:5001;https://0.0.0.0:5002"
export ASPNETCORE_Kestrel__Certificates__Default__Password=MySecretPassword
export ASPNETCORE_Kestrel__Certificates__Default__Path=/home/sridh/.aspnet/https/aspnetapp.pfx
cd ~/dotnetApp/bin/Release/net8.0/publish
 ./BlazingBlog &
 ```
