+++
date = '2025-03-15T11:31:36-04:00'
draft = false
title = 'Nginx Dot Net Core'
tags = ['nginx', 'hosting', 'gcp']
categories = ['dot-net', 'hosting']
+++

## How was Stock filter installed?

# Introduction

Stockfilter is a Dot Net application that uses the inbuilt webserver Kestrel to host the application. Kestrel is a light weight web server for hosting ASP.NET Core applications on really any platform. Kestrel lacks a lot of the things that an ASP.NET web developer might have come to expect from a web server like IIS. For instance you cannot do SSL termination with Kestrel or URL rewrites or GZip compression.

Nginix is a full-fledged web server. You can use it as a reverse proxy; in this configuration it takes load off your actual web server by preserving a cache of data which it serves before calling back to your web server. As a proxy it can also sit in front of multiple end points on your server and make them appear to be a single end point. This is useful for hiding a number of microservices behind a single end point. It can do SSL termination which makes it easy to add SSL to your site without having to modify a single line of code.

This document was prepared using the procedures mentioned in [Microsoft  docs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-6.0) and following a [YouTube](https://www.youtube.com/watch?v=mkBYgwfOzdc) by Ayden’s Developer Channel.

# Register your domain

For this application I used [Google domain](https://domains.google/) to register the domain name [Stock Filter](https://stockfilter.de/).

## DNS Resolve

Click on the name of the new domain you created in Google Domain and on the left menu click on DNS. Select *Default name servers* and *Manage custom records.* Here we are going to create an *A Record* and a *CNAME Record*. (A record is domain name to IP address mapping and CNAME points to another name *usually the value on A record*).

## Install required software

This is simple; we are going to issue the following commands.
```sh
sudo apt install nginx
sudo apt install certbot
sudo apt install python3-certbot-nginx
```

Lets assume you worked on the CNAME and A Record(s); if not take a walk. This might be good time to ensure your installation is working fine until now. You should be greeted with the default Nginx web page and if you get see any errors then **Stackoverflow**!

## Build your application

By now you should have your application fully tested and ready for deployment. Build your application with the command
```sh
dotnet publish -c Release 
```
Once successfully built the executables will be in 
```sh
........./bin/Release/netX.X/
```
folder. We want this application to start on startup as a service and the best way to start the application and restart it if needed will be to run it as a Daemon application. To do that we will create a file in 
```sh 
/etc/systemd/system/
```
folder. Though you can create the file with any name I like the naming convention
```
ApplicationName.service
```
The content of this file will be as follows (adjust it as per your needs)
```sh
[Unit]
Description= StockSelector
[Service]
User=srvean
Group=srvean
WorkingDirectory=/home/srvean/DotNet/Stock-Selector-UI/StockSelector/Server/
ExecStart=/usr/bin/dotnet run --configuration Release
Restart=always
# Restart service after 60 seconds if the dotnet service crashes:
RestartSec=60
SyslogIdentifier=StockSelector
Environment=ASPNETCORE_ENVIRONMENT=Production

[Install]
WantedBy=multi-user.target                    
```
Most of the values above should be self-explanatory. The working directory could have been the *bin/Release/netx/* folder but didn't try **dotnet run xxxx.dll** command. Going forward we'll assume that the application **listens on port 5000 for HTTP** requests.  Now let us enable the server and run it:
```sh
sudo systemctl enable ApplicationName.service
sudo systemctl restart ApplicationName.service
sudo systemctl status ApplicationName.service
```
Well, you know what we are doing above.
## Configure Nginx
To configure Nginx as a reverse proxy to forward HTTP requests to your ASP.NET Core app, modify `/etc/nginx/sites-available/default`. Open it in a text editor, **and replace the contents with the following snippet**:
```
server {
    listen        80;
    server_name   stockfilter.de;
    location / {
        proxy_pass         http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```
You can use the above code as is; only change the value for *server_name*. After updating the file you can run the command:
```sh
sudo nginx -t
```
to test if the configuration is free of manual errors. Now let us restart the server with command
```sh
sudo systemctl restart nginx
```
Now if you navigate to your site (http not ~~https~~) your Dot Net application should be running. If you do not get the expected results => **Stackoverflow**!
## Securing our site.
To automate the process of obtaining, installing, and updating TLS/SSL certificates on a web server, [Let’s Encrypt](https://letsencrypt.org/) is an extremely useful tool. It is a certificate authority (CA) that comes packaged with a corresponding software client, Certbot, that will automatically install TLS/SSL certificates. This means that we can run encrypted HTTPS on a web server without having to worry about constant maintenance.

We'll relate Certbot and our instance of Nginx using the following command:
```sh
sudo certbot --nginx -d stockfilter.de
```
Obviously, the value for -d in the above command will be for our domain name. Certbot will ask a few straightforward questions which need to be answered.
### Proxy 
Add the _/etc/nginx/proxy.conf_ configuration file:
```
proxy_redirect          off;
proxy_set_header        Host $host;
proxy_set_header        X-Real-IP $remote_addr;
proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header        X-Forwarded-Proto $scheme;
client_max_body_size    10m;
client_body_buffer_size 128k;
proxy_connect_timeout   90;
proxy_send_timeout      90;
proxy_read_timeout      90;
proxy_buffers           32 4k;
```
**Replace** the contents of the _/etc/nginx/nginx.conf_ configuration file with the following file. The example contains both `http` and `server` sections in one configuration file.
```
http {
    include        /etc/nginx/proxy.conf;
    limit_req_zone $binary_remote_addr zone=one:10m rate=5r/s;
    server_tokens  off;

    sendfile on;
    # Adjust keepalive_timeout to the lowest possible value that makes sense 
    # for your use case.
    keepalive_timeout   29;
    client_body_timeout 10; client_header_timeout 10; send_timeout 10;

    upstream helloapp{
        server 127.0.0.1:5000;
    }

    server {
        listen                    443 ssl http2;
        listen                    [::]:443 ssl http2;
        server_name               example.com *.example.com;
        ssl_certificate           /etc/ssl/certs/testCert.crt;
        ssl_certificate_key       /etc/ssl/certs/testCert.key;
        ssl_session_timeout       1d;
        ssl_protocols             TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers off;
        ssl_ciphers               ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
        ssl_session_cache         shared:SSL:10m;
        ssl_session_tickets       off;
        ssl_stapling              off;

        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;

        #Redirects all traffic
        location / {
            proxy_pass http://helloapp;
            limit_req  zone=one burst=10 nodelay;
        }
    }
}
```
In the above file you need to update **ssl_certificate** and **ssl_certificate_key**. We can get its values from */etc/nginx/sites-available/default* file that we updated earlier. After we updated when we ran Certbot it would have added a few lines after our edit and in the inserts that Certbot inserted you would see entries for **ssl_certificate** and **ssl_certificate_key**. Copy those values and replace it in the above code. Also replace *example.com* in the above code to your domain (e.g. stockfilter.de).
## Almost done
Let's make sure our configuration changes are good by issuing the command 
```sh
sudo nginx -t
```
And if nginx states you've done an excellent job then let us restart the server  by issuing the command
```sh
sudo systemctl restart nginx
```
Navigate to your site using the https link (e.g. https://stockfilter.de) and if things are good then its time for your coffee break :coffee:. 

Else **Stackoverflow**! :anguished:
