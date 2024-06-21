+++
author = "kiwiyumi"
date = "2024-06-11"
description = "Export source code"
title = "TrakSYS - CVE-2024-6188 - Direct access to export Source Code"
tags = [
    "Web",
    "CVE",
    "CMS"
]
+++

<!--more-->

## What is TrakSYS

TrakSYS is a software platform developed by Parsec Automation Corp. for managing real-time operations and optimizing manufacturing processes. It is widely used in industries to monitor, analyze and improve the efficiency and productivity of production processes.

## Versions affected

During my tests it was observed that versions 11.x.x are vulnerable.

## Affected Endpoint

https://site[.]com/TS/export/pagedefinition?ID={id}


## Impact

An attacker is able to export the source code of the pages without having any credentials to access the application.


## Description

It was not possible to confirm the presence of mechanisms that check whether the user is authorized to carry out certain actions in the system, or whether the user has been authenticated by the application. Because of this, it was possible to export the application's source code.

The following images illustrate successful unauthenticated requests to the affected endpoint:

![](/assets/images/traksys-export-source-code/bc4b5130-8706-48ac-94b2-bb8a5ec868ca.png)

![](/assets/images/traksys-export-source-code/218d3e03-6ea6-4fc7-8586-d5dc6df55275.png)

## Recommendation

It is strongly recommended that changes be made to the application's existing session management and access control, such that access to sensitive functionalities is available only to to autenticated users, and that these users perform only actions by their authorization profile.

## PoC

1. Do the following request replacing the **ID** with a valid id and host with your target:

```http
GET /TS/export/pagedefinition?ID={ID} HTTP/1.1
Host: {HOST}

```

2. Note that it is possible to export and view the source code.

### Font

- [https://cheatsheetseries.owasp.org/](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)