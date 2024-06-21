+++
author = "kiwiyumi"
date = "2024-02-01"
description = "Dump and Cracking Firefox Passwords"
title = "Dump and Cracking Firefox Passwords"
tags = [
    "firefox",
    "post-exploitation"
]
+++

<!--more-->

Do you trust your browser to store and protect your passwords? Today's article we will understand how an attacker can exploit and obtain sensitive information from your Firefox browser.

The scenario reproduced in this article is based on the following premise: an attacker with access to the victim's machine starts the post-exploitation process and locates the Firefox files, then using the **dumpzilla** tool we decrypt the passwords.

## Standard paths

The firefox browser creates a directory to store information and settings, the paths of which can be different depending on the operating system or the user's choice. Here are the standard paths for each operating system:

- Linux

```bash
/home/$USER/.mozilla/firefox/xxxx.default
```

- MacOS

```bash
/Users/$USER/Library/Application Support/Firefox/Profiles/xxxx.default
```

- Windows

```text
C:\Users\%USERNAME%\AppData\Roaming\Mozilla\Firefox\Profiles\xxxx.default
```

## Dumpzilla installation - Kali Linux

Installing the tool on the Kali Linux system is simple, just use apt to install the program:

```bash
sudo apt install dumpzilla
```

## Dumpzilla installation - Ubuntu

### Installing the dependencies

First you need to install the **lz4** library with pip3, this library is responsible for data compression. Here is the command to install lz4:

```bash
pip3 install logging lz4
```

Other necessary dependencies are python3 (I believe you already have it on your system), the sqlite3 database and the **libnss3** package.  

```bash
sudo apt-get install python3 sqlite3 libnss3*.
```

### Dumpzilla download

Downloading the tool is simple, just clone the repository with git:

```
git clone https://github.com/Busindre/dumpzilla.git
```

After that, just access the directory you created and run the code in python3

```
cd dumpzilla
python3 dumpzilla.py
```

If everything is OK, you should receive a screen showing the options for using the tool:

![It's work!](/assets/images/firefox-dump-passwords/a423071f-ba14-47fd-999d-a0377a50305a.png)

## Firefox versions

The process for encrypting and storing passwords in firefox depends on the summer installed, below is a list showing the values:

- Firefox < 32 (key3.db, signons.sqlite)
- Firefox >=32 (key3.db, logins.json)
- Firefox >=58.0.2 (key4.db, logins.json)
- Firefox >=75.0 (sha1 pbkdf2 sha256 aes256 cbc used by key4.db, logins.json)

Note that the key file stores the encryption key used to encrypt the values in the sqlite/json files. In this way, it is possible to carry out the attack offline, but it is necessary to have the necessary files on hand.

## Data Exfiltration

Once you have access to the victim's machine, it is recommended that you exfiltrate this information to the attacker's machine. There are several techniques for doing this, apply the one that best fits your scenario and operating system.

## Getting users and passwords

By default, firefox automatically creates directories with 8 random characters, being letters and numbers, for example, k2efmn13.study, iyna1qt8.code and Tscks1c3.default for example.

The syntax of the tool for collecting only browser passwords is as follows:

```bash
python3 dumpzilla.py PATH/TO/PROFILES/XXXXXXXX.default/ --Password
```

In the following image you can see an example, note that it was possible to obtain the email access credentials in plain text:

![Dump password!](/assets/images/firefox-dump-passwords/9ae0ab51-3914-4bd5-84b8-3207ea0da77e.png)

However, the tool isn't just for collecting passwords, with dumpzilla we can collect much more, for example:

 - Cookies + DOM Storage (HTML 5).
 - User preferences (Domain permissions, Proxy settings...).
 - Downloads.
 - Web forms (Searches, emails, comments..).
 - Historial.
 - Bookmarks.
 - Cache HTML5 Visualization / Extraction (Offline cache).
 - visited sites "thumbnails" Visualization / Extraction .
 - Addons / Extensions and used paths or urls.
 - Browser saved passwords.
 - SSL Certificates added as a exception.
 - Session data (Webs, reference URLs and text used in forms).
 - Visualize live user surfing, Url used in each tab / window and use of forms. 

To view all this information at once, simply run the tool without specifying an option. In the example above, I've filtered by "--Passwords", but be aware that you can select what you want to collect.

More information about the tool is available on the official github:

[https://github.com/Busindre/dumpzilla](https://github.com/Busindre/dumpzilla)

## Conclusions

As demonstrated in this post, we can see that storing passwords in any browser can be dangerous for you and easy for an attacker. I strongly advise you to use an **offline** database to store your passwords, for example KeePass and its variants.

## References

- [https://support.mozilla.org/en-US/kb/where-are-my-logins-stored](https://support.mozilla.org/en-US/kb/where-are-my-logins-stored)
- [https://null-byte.wonderhowto.com/how-to/hacking-macos-dump-passwords-stored-firefox-browsers-remotely-0185234/](https://null-byte.wonderhowto.com/how-to/hacking-macos-dump-passwords-stored-firefox-browsers-remotely-0185234/)
- [https://kylemistele.medium.com/stealing-saved-browser-passwords-your-new-favorite-post-exploitation-technique-c5e72c86159a](https://kylemistele.medium.com/stealing-saved-browser-passwords-your-new-favorite-post-exploitation-technique-c5e72c86159a)
- [https://github.com/Busindre/dumpzilla](https://github.com/Busindre/dumpzilla)