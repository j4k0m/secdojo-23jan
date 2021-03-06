# secdojo-23jan
SecDojo 23jan CTF writeup.

##
![image](https://user-images.githubusercontent.com/48088579/151042148-acab8117-1cc0-400e-87de-209a5c9f3243.png)


# Knock-It


### Lab Description
The purpose of this lab is to learn how to perform network packet analysis in order to get more intel about the hidden network infrastructure, detect misconfigurations, and most importantly, how you can make use of this intel to get access to systems and escalate your privileges.


## Unacknowledged

IP: 172.16.4.74

### Nmap Scan: `nmap -sC -sV 172.16.4.74`

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
knock 172.16.4.74 21
nmap --script firewall-bypass 172.16.4.74
```

### FTP Enumaration: `nmap -sC -sV -p21 172.16.4.74`

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

### FTP Anonymous Login: `ftp 172.16.4.74`

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

### `login.pcap` Analysis: `strings login.pcap | less`

![image](https://user-images.githubusercontent.com/48088579/151044232-5ddc1ac1-8c8b-44dc-9efb-122d3ef9730d.png)

NOTE: You can use `tshark`.

## User:

![image](https://user-images.githubusercontent.com/48088579/151044341-81c93571-bb77-4578-a013-0611112d38f7.png)

## Privilege Escalation:

```bash
$ sudo -l
Matching Defaults entries for kabayla on ip-172-16-4-74:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User kabayla may run the following commands on ip-172-16-4-74:
    (root) NOPASSWD: /usr/bin/wget
$ 
```

### Using `wget` to download our public key in `.ssh` root folder:

### In our pwnbox:

![image](https://user-images.githubusercontent.com/48088579/151044744-42a0bbdb-027b-412a-abc3-7bb7354a98bb.png)

### In the target machine:

![image](https://user-images.githubusercontent.com/48088579/151044854-1d007ffd-7c70-439a-b932-12d6696e8101.png)

# SSH Connection to root:

![image](https://user-images.githubusercontent.com/48088579/151044910-3b577833-711f-45b1-bb8c-55909f5ff5eb.png)

##
![image](https://user-images.githubusercontent.com/48088579/151046357-ecd91039-7d91-480c-a57c-25a7633ccabf.png)

## Ransomware101


### Lab Description
Ransomware101 is a scenario that represents a typical case of a Windows machine hit by ransomware, a malware that encrypts all personal victim data in exchange for a ransom.
As an investigator, you must first locate and analyze the malware using reverse engineering techniques to be able to understand its infection mechanism.
Once you deduce the encryption key used by the malware, you can decrypt and submit the encrypted flag located at C:\Users\Administrator\Desktop\proof.txt.

## Infected1:

IP: 172.16.4.239

### RDP Connection: `rdesktop -u Administrator -p "bJEhU7R7oNYv5weWf*yhig6SnM.-Yg4s" -r disk:shared=/home/kali/Desktop 172.16.4.239`



![image](https://user-images.githubusercontent.com/48088579/151047933-832d3a8f-d71c-46d6-9807-0858a01075e1.png)

### Scanning Using `Autoruns64`:

Autoruns64:
This utility, which has the most comprehensive knowledge of auto-starting locations of any startup monitor, shows you what programs are configured to run during system bootup or login, and when you start various built-in Windows applications like Internet Explorer, Explorer and media players. These programs and drivers include ones in your startup folder, Run, RunOnce, and other Registry keys. Autoruns reports Explorer shell extensions, toolbars, browser helper objects, Winlogon notifications, auto-start services, and much more. Autoruns goes way beyond other autostart utilities.

Source: https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns

![image](https://user-images.githubusercontent.com/48088579/151049112-c1110321-8de3-41ec-a688-5e8da9462c12.png)

### Malware Files Location: `C:\ProgramData\Microsoft\`

![image](https://user-images.githubusercontent.com/48088579/151049322-da545d5b-3e45-4ef5-9fa0-0471e62facb3.png)

![image](https://user-images.githubusercontent.com/48088579/151049399-21418c60-f5c5-4487-b129-8e59e0394e44.png)



`decryptor101.exe`: a binary to decrypt the encrypted files with a key.


![image](https://user-images.githubusercontent.com/48088579/151050130-fa5b5f04-1aed-4dc5-b2c2-8106e4197c93.png)


`r101.dll`: where all the magic happen.


### Reverse generation key algorithm in `r101.dll`:

The malware will generate a key at the start that will use to encrypt all the files, the malware will exclude the files with `.secdojo,.exe,.dll,.sys,.ps1,.lnk` extension in `C:\\Users\\Administrator\\Desktop` path.

![image](https://user-images.githubusercontent.com/48088579/151051912-0fb6afda-8baf-41f3-bab3-cfa6d6a2b55c.png)


![image](https://user-images.githubusercontent.com/48088579/151051983-ad093d6f-3278-4b66-bb11-be0c17e7ed22.png)


![image](https://user-images.githubusercontent.com/48088579/151052005-ab7e6403-4a21-4052-8099-3a0cf4b9284d.png)

The malware will use the computer name and mac address as key components to generate the key, and the key is the output of md5sum of the key components.

![image](https://user-images.githubusercontent.com/48088579/151053135-054a9534-fd7b-41b7-929a-dc24d7b2e515.png)


![image](https://user-images.githubusercontent.com/48088579/151053540-0f501ce2-d18e-4dd1-8f4c-202cdf24a9e9.png)


![image](https://user-images.githubusercontent.com/48088579/151053630-c83d1671-e646-41ee-8d75-814804f43865.png)

Knowing those components we can try and search with them in the process dump.

### Getting process dump file:

In ProcessExplorer create Full Dump of the process:

![image](https://user-images.githubusercontent.com/48088579/151054070-7661bcbb-024f-40ea-b329-aa669f6c193e.png)

### Getting the key:

Computer Name: `EC2AMAZ-59S0107`

![image](https://user-images.githubusercontent.com/48088579/151054265-a496e3f3-7fae-4ef6-897a-7690573d5425.png)


![image](https://user-images.githubusercontent.com/48088579/151054800-222a4164-ad0d-490f-9b43-1ef91708f8d6.png)

And we can notice a MD5 format string: `08780acf63112fbaf55c96ca5ef0b816` and key components `EC2AMAZ-59S010706-AE-70-11-A2-2C`, and now we can use `08780acf63112fbaf55c96ca5ef0b816`  to decrypt the files.

### Decryption:

![image](https://user-images.githubusercontent.com/48088579/151055381-a95f1618-a11a-4a7e-892d-c7fefa23842c.png)

