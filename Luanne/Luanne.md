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

## Gobuster
### 80
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
### 9001

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
curl http://luanne.htb/weather/forecast?city="'"
<br>Lua error: /usr/local/webapi/weather.lua:49: attempt to call a nil value
```
Lua??

comment out
