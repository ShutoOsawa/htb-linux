#linux 

# Enumeration
## Nmap port scan

```
nmap -sC -sV friendzone.htb
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-24 21:59 EST
Nmap scan report for friendzone.htb (10.129.88.74)
Host is up (0.092s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a96824bc971f1e54a58045e74cd9aaa0 (RSA)
|   256 e5440146ee7abb7ce91acb14999e2b8e (ECDSA)
|_  256 004e1a4f33e8a0de86a6e42a5f84612b (ED25519)
53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Friend Zone Escape software
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  ssl/http    Apache httpd 2.4.29
| ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
| Not valid before: 2018-10-05T21:02:30
|_Not valid after:  2018-11-04T21:02:30
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
|_http-title: 404 Not Found
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Hosts: FRIENDZONE, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -40m02s, deviation: 1h09m16s, median: -3s
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-02-25T02:59:46
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: friendzone
|   NetBIOS computer name: FRIENDZONE\x00
|   Domain name: \x00
|   FQDN: friendzone
|_  System time: 2023-02-25T04:59:46+02:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.46 seconds
                                                               
```

port 443 friendzone.red somewhat interesting lets add it to etc/hosts

## Gobuster http
```
gobuster dir -u http://friendzone.htb/ --wordlist /usr/share/wordlis
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://friendzone.htb/
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-li
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.4
[+] Timeout:                 10s
===============================================================
2023/02/24 22:02:14 Starting gobuster in directory enumeration mode
===============================================================
/wordpress            (Status: 301) [Size: 320] [--> http://friendzone.htb/wordpress/]                                                          
Progress: 141570 / 141709 (99.90%)
===============================================================
2023/02/24 22:03:47 Finished
===============================================================
                                    
```


## Gobuster https

```
gobuster dir -u https://friendzone.red/ --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt -k -t 150
```
with -k option we can skip certification 
we would get something like this if we dont skip it.
```
Error: error on running gobuster: unable to connect to https://friendzone.red/: invalid certificate: x509: certificate has expired or is not yet valid: current time 2023-02-25T07:46:50-05:00 is after 2018-11-04T21:02:30Z
```

```
/admin                (Status: 301) [Size: 318] [--> https://friendzone.red/admin/]
/js                   (Status: 301) [Size: 315] [--> https://friendzone.red/js/]
```

## FTP port 21
Anonymous login not allowed


## Http port 80

### friendzone.htb/
![[Pasted image 20230225123517.png]]
friendzoneportal.red???

### /wordpress
![[Pasted image 20230225120516.png]]

## Https port 443
![[Pasted image 20230225165344.png]]

In the source code
![[Pasted image 20230225165434.png]]
### /js/js
![[Pasted image 20230225165633.png]]
```
WElMa3JQdG5vYTE2NzczMTE3NzUxNW9IR0ZlUkpp
```

### /admin
![[Pasted image 20230225215624.png]]


## Samba port 445

### smbmap
```
smbmap -H friendzone.htb   
[+] Guest session       IP: friendzone.htb:445  Name: unknown                                           
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        Files                                                   NO ACCESS       FriendZone Samba Server Files /etc/Files
        general                                                 READ ONLY       FriendZone Samba Server Files
        Development                                             READ, WRITE     FriendZone Samba Server Files
        IPC$                                                    NO ACCESS       IPC Service (FriendZone server (Samba, Ubuntu))
                                                              
```

readonly
general
Read and write
Development

### smbclient
```
smbclient -N -L //friendzone.htb

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        Files           Disk      FriendZone Samba Server Files /etc/Files
        general         Disk      FriendZone Samba Server Files
        Development     Disk      FriendZone Samba Server Files
        IPC$            IPC       IPC Service (FriendZone server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        HTB                  LEGACY
        WORKGROUP            FRIENDZONE
                                              
```

#### Development
```
smbclient -N //friendzone.htb/Development
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Feb 24 22:11:07 2023
  ..                                  D        0  Tue Sep 13 10:56:24 2022

                3545824 blocks of size 1024. 1622184 blocks available
```
nothing

#### general
```
smbclient -N //friendzone.htb/general    
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jan 16 15:10:51 2019
  ..                                  D        0  Tue Sep 13 10:56:24 2022
  creds.txt                           N       57  Tue Oct  9 19:52:42 2018

                3545824 blocks of size 1024. 1622184 blocks available
```

Creds.txt
```
cat creds.txt       
creds for the admin THING:

admin:WORKWORKHhallelujah@#
```

## Check domain
### friendzone.htb
```
dig axfr friendzone.htb @10.129.209.64

; <<>> DiG 9.18.11-2-Debian <<>> axfr friendzone.htb @10.129.209.64
;; global options: +cmd
; Transfer failed.
```

### friendzone.red
```
dig axfr friendzone.red @10.129.88.74

; <<>> DiG 9.18.11-2-Debian <<>> axfr friendzone.red @10.129.88.74
;; global options: +cmd
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
friendzone.red.         604800  IN      AAAA    ::1
friendzone.red.         604800  IN      NS      localhost.
friendzone.red.         604800  IN      A       127.0.0.1
administrator1.friendzone.red. 604800 IN A      127.0.0.1
hr.friendzone.red.      604800  IN      A       127.0.0.1
uploads.friendzone.red. 604800  IN      A       127.0.0.1
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 88 msec
;; SERVER: 10.129.88.74#53(10.129.88.74) (TCP)
;; WHEN: Fri Feb 24 22:36:08 EST 2023
;; XFR size: 8 records (messages 1, bytes 289)
```

`administrator1.friendzone.red, hr.friendzone.red, uploads.friendzone.red`

### administrator1.friendzone.red
```
dig axfr administrator1.friendzone.red @10.129.209.64

; <<>> DiG 9.18.11-2-Debian <<>> axfr administrator1.friendzone.red @10.129.209.64
;; global options: +cmd
; Transfer failed.
```

### hr.friendzone.red
```
dig axfr hr.friendzone.red @10.129.209.64

; <<>> DiG 9.18.11-2-Debian <<>> axfr hr.friendzone.red @10.129.209.64
;; global options: +cmd
; Transfer failed.
```

### uploads.friendzone.red
```
dig axfr uploads.friendzone.red @10.129.209.64

; <<>> DiG 9.18.11-2-Debian <<>> axfr uploads.friendzone.red @10.129.209.64
;; global options: +cmd
; Transfer failed.
```

### friendzoneportal.red

```
dig axfr friendzoneportal.red @10.129.209.64

; <<>> DiG 9.18.11-2-Debian <<>> axfr friendzoneportal.red @10.129.209.64
;; global options: +cmd
friendzoneportal.red.   604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
friendzoneportal.red.   604800  IN      AAAA    ::1
friendzoneportal.red.   604800  IN      NS      localhost.
friendzoneportal.red.   604800  IN      A       127.0.0.1
admin.friendzoneportal.red. 604800 IN   A       127.0.0.1
files.friendzoneportal.red. 604800 IN   A       127.0.0.1
imports.friendzoneportal.red. 604800 IN A       127.0.0.1
vpn.friendzoneportal.red. 604800 IN     A       127.0.0.1
friendzoneportal.red.   604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 188 msec
;; SERVER: 10.129.209.64#53(10.129.209.64) (TCP)
;; WHEN: Sat Feb 25 07:20:44 EST 2023
;; XFR size: 9 records (messages 1, bytes 309)
```

## /etc/hosts

```
10.129.209.64 friendzone.htb friendzone.red friendzoneportal.red administrator1.friendzone.red hr.friendzone.red uploads.friendzone.red admin.friendzoneportal.red files.friendzoneportal.red imports.friendzoneportal.red vpn.friendzoneportal.red
```

## Check other subdomains

### friendzoneportal.red
![[Pasted image 20230225211811.png]]

### administrator1.friendzone.red
![[Pasted image 20230225211731.png]]

#### try to login
admin:WORKWORKHhallelujah@#
![[Pasted image 20230225215810.png]]

### dashboard.php
![[Pasted image 20230225222346.png]]

#### Default image
```
https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=timestamp
```

![[Pasted image 20230225222541.png]]
Final Access timestamp is 1677335106

#### gobuster

```

```

try php as well
```

```
### hr.friendzone.red
![[Pasted image 20230225213527.png]]

### uploads.friendzone.red
![[Pasted image 20230225213648.png]]

### admin.friendzoneportal.red 
![[Pasted image 20230225214100.png]]

#### try to login
admin:WORKWORKHhallelujah@#
![[Pasted image 20230225214341.png]]
### files.friendzoneportal.red 
404
### imports.friendzoneportal.red 
404
### vpn.friendzoneportal.red
404

