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

## Gobuster
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

## /etc/hosts

```
10.129.88.74 friendzone.red,friendzoneportal.red,administrator1.friendzone.red,hr.friendzone.red,uploads.friendzone.red
```


![[Pasted image 20230225124900.png]]


## Trouble connecting

- Changed vpn
- Restarted the box again
- removed etc/host
- added only friendzone.red
- visited the `http://<machine ip>`
- then visited the `http://friendzone.red`
- firefox warning showed up and I allowed the website
I dont know why it happened but I got stuck