+++
author = "kiwiyumi"
date = "2024-03-01"
description = "Load Balancer everwhere!"
title = "NGINX - How to Create a Load Balancer"
tags = [
    "Web-server",
    "Infra"
]
+++

<!--more-->

Hello everyone, today we're going to talk about a subject related to server infrastructure, more specifically **Load Balancers**. Have you ever wondered how servers can handle so many requests per second, tens of thousands of users and still keep the server running? In this article we'll learn a little about load balancers and perform a sample configuration.

# What is Load Balancer?
According to the official **NGINX** documentation "Load balancing refers to the efficient distribution of incoming network traffic across a group of backend servers, also known as a server farm or server pool."

In simpler terms, the Load Balancer works from layer 7 of the OSI (Open Systems Interconnection) model. Once a new request arrives at the Load Balancer, it has the task of validating the rules defined and seeing which server can receive the connection, so as to avoid high load and instability problems.

# Algorithms for Load Balancer
Each Load Balancer can have its own characteristics and provide different benefits, so it is important to know which scenario best suits the use of the Load Balancer and which algorithm can provide the best performance. Below is a list of some existing algorithms:

- **Round Robin:** Requests are distributed sequentially to each server;
- **Least Connections:** The request is sent to the server with the lowest load at the time;
- **Least Time:** Sends requests to selected servers based on a calculation, to design a faster response;
- **Hash:** Distributes requests based on a predefined key, such as an IP address or URL;
- **IP Hash:** The client's IP address is used to determine which server should receive the request;
- **Random with Two Choices:** The Load Balancer server will select two servers and send the request to the first one selected (based on the algorithm with the least time).

# What is NGINX?
NGINX is an open source software for web servers created in October 2004, originally designed to serve HTTP browsers, but nowadays it can be used as a reverse proxy, HTTP load balancer, proxy for emails in IMAP, POP3 and SMTP protocols.

# Designed Scenario
The idea of this article is to enable you to understand how to create a simple load balancer with NGINX and to provide guidance for your future studies. Below is an image of the environment we will be building:

![](/assets/images/nginx-load-balancer/2d92160b-07d2-4329-b71b-a27a3aab0e08.png)
 
Our goal is to allow any request made by a user (Client) to be redirected to our Load Balancer and then, depending on availability, to choose the best path (App-01 or App-02). 

The settings for each server are as follows:
- **Load Balancer:** Ubuntu server 22.04 with NGINX 1.18.0;
- **App-01:** Ubuntu server 20.04 with NGINX 1.18.0;
- **App-02:** Ubuntu server 20.04 with NGINX 1.18.0.

I won't be covering the installation of Ubuntu server in this article, but just download the iso and install it in a virtual machine (or docker if you prefer) to follow along with this article, follow the download link:

[https://ubuntu.com/download/server](https://ubuntu.com/download/server)

# General Installation
In this topic we are going to install everything we need for the three servers in question, so let's first update the packages:

```bash
sudo apt update && sudo apt upgrade -y
```

With the updated packages we can install NGINX with the following command:

```bash
sudo apt install nginx
```

After you have finished downloading and installing NGINX, you need to initialize the process:

```bash
sudo systemctl start nginx
```

This is completely optional, but if you want to activate NGINX every time you turn on the machine, just enable it with the following command:

```bash
sudo systemctl enable nginx
```

To finalize and validate the installation, simply access your server and see the standard NGINX welcome message:

![](/assets/images/nginx-load-balancer/084d9276-9fd5-4559-9aa8-f446c5553941.png)

# Configuring Load Balancer
Now let's start configuring the Load Balancer. To do this, go to the available sites configuration file, the following command will take you to the directory in question:

```bash
cd /etc/nginx/sites-available
```

By default you will only find a file called **default**, let's open it with a text editor (such as nano, vim and the like), delete the existing content and apply the new settings.

Initially in the file, we have to define the name of our "system", the algorithm to be used and which servers we are going to point to, the following code exemplifies the syntax:

```text
upstream app {
    # Here is the algorithm to be used, if one is not chosen
    # Round Robin is used by default

    # Here are the servers (IP or DOMAIN) to which requests should be forwarded
    server nginx-app01;
    server nginx-app02;
}
```

Next, we need to configure the server options, which port we are listening on, the server name and so on. The following code illustrates a configuration:

```text
server {
    # Configuring the port
    listen 80 default_server;

    # Setting the server name
    server_name nginxlb;

    location / {
        # Enter the name of the upstream previously defined, in my case app
        proxy_pass http://app;

        # Disable redirection in other directives
        proxy_redirect off;

        # Set the HTTP version, by default 1.0
        proxy_http_version 1.1;
    }
}

```

With each part explained, our configuration file is as follows:

```text
upstream app {
    server nginx-app01;
    server nginx-app02;
}

server {
    listen 80 default_server;
    server_name nginxlb;

    location / {
        proxy_pass http://app;
        proxy_redirect off;
        proxy_http_version 1.1;
    }
}
```

Now just save the file and then perform syntax validation with NGINX, using:

```bash
sudo nginx -t
```

If all goes well, a success message should appear, otherwise look at the error message to solve the problem. Finally, restart the NGINX server:

```bash
sudo systemctl restart nginx
```

Load Balancer configuration complete!

# Configuring App-01
Accessing the App-01 server, go to the directory containing the web page files, usually located in **/var/www/html**, delete the existing content and create an index.html file with simple HTML, for example:

```html
<h1>This is App-01!</h1>
```

# Configuring App-02
Accessing the App-02 server, go to the directory containing the WEB page files, usually located in **/var/www/html**, delete the existing content and create an index.html file with simple HTML, for example:

```html
<h1>This is App-02!</h1>
```

# Testing!

Now that everything is set up, by accessing the Load Balancer IP/Domain, I can see that we are redirected between the App-01 and App-02 servers at each request.

The following images show how the servers change with each request:

![Loading the App-01](/assets/images/nginx-load-balancer/95e8666f-5a7c-4ab4-a3fa-53161e9bb8e4.png)

![Loading the App-02](/assets/images/nginx-load-balancer/57c582d5-38f1-414c-bc63-f2d6f87af2cd.png)


# Security Tip
Whenever a Load Balancer is used in an application, it is important that it always has the same configuration on all servers, otherwise some endpoints that should not be allowed to be accessed (403 Forbidden) may be accessed due to a lack of configuration.

# Conclusion
In this article it was possible to learn how a Load Balancer works, how to use NGINX for this purpose and how to create an environment from scratch in the simplest way possible. I strongly recommend reading the official NGINX documentation for more information on use and configuration.

Thanks for reading!

## References

- [http://nginx.org/en/docs/http/load_balancing.html](http://nginx.org/en/docs/http/load_balancing.html)
- [https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)