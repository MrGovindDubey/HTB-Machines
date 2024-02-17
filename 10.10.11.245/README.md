# Surveillance


<p align="center">
  <img src="https://github.com/MrGovindDubey/HTB-Machines/assets/118271775/6acae25e-f039-4ff6-b160-876222d82942" align="center" alt="keeper-htb" />
</p>



## Introduction :

The Surveillance machine on Hack The Box presented a challenging learning opportunity. The journey involved Nmap scanning, web enumeration, and directory busting to discover an admin login on Craft CMS. Leveraging a Remote Code Execution (RCE) exploit, initial access was gained, leading to the discovery of a user named Matthew. After cracking Matthew's password, SSH access was achieved.

The privilege escalation involved MySQL credentials, "zoneminder" configurations, and a version-specific exploit using Metasploit. Despite challenges, a method to escalate privileges to root was found by manipulating the "user" parameter in zmupdate.pl. The write-up underscores the importance of thorough reconnaissance, exploit analysis, and creative problem-solving in successfully navigating the complexities of the Surveillance machine on HTB.


