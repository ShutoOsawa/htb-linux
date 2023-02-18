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
