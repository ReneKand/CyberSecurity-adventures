Before starting an overview over steps I want to take.
1. NMAP
2. GOBUSTER
If needed, I adapt NMAP for UDP.

--- 
The IP address is **10.129.98.103**
I want the User and root flag


I run nmap to see open ports: **nmap -sC -sV 10.129.98.103**
> Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-01-25 12:15 CST
Nmap scan report for 10.129.98.103
Host is up (0.0087s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    gunicorn

I shortened the output. It mainly reveals three open TCP ports. 

**Port 21** for FTP, to be precise, vsftpd, so a `very` secure version hahaha. Still possible misconfigurations, or this [exploit](https://www.exploit-db.com/exploits/49719)

**Port 22** ssh. Secure shell, no simple exploits. Maybe later, when I have found credentials. No direct vulnerability found.

**Port 80** Works. So, GoBuster's next step. From the scan, I see gunicorn, python backend, and custom logic, therefore also mistakes. 

I run `gobuster dir -u 10.129.98.103 -w /usr/share/wordlists/dirb/common.txt` So common to check for possible endpoints.

>Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.98.103
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/data                 (Status: 302) [Size: 208] [--> http://10.129.98.103/]
/ip                   (Status: 200) [Size: 17452]
/netstat              (Status: 200) [Size: 28506]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================


Really nice opening. I have three openings: 

+ There is /data which moved to ../
+ /ip
+ /netstat

Let's go with the most classical and just parse the IP address. I am automatically logged in as Nathan. I am not on the homepage, but on dashboards. I see Security events, failed login attempts, and port scans visualised in adashboard.
I am at least looking at the website on Home/Dashboard. I will start playing around with it. Home is the dashboard, so that is that.

I can download data from the page. I downloaded the first as 1.pcap file. To open that file, I need Wireshark. I open it in Wireshark it is empty.

I check out IP config and network status.

After finding nothing of interest to my untrained eye, I again open up the first link. I see that in contrast to my first download, there seems to have been traffic. I download and open Wireshark.
Even though some parts seem to have been corrupted or anything, I can read a message. I hover through the Web app again. The ../data/[x] works in a way where x = a serial user id. So I have downloaded 0, 1, and 2. Generally, there is not much difference between the contents except for the sizes.

I follow the Hack the Box page as I am not well-versed with Wireshark. I switched the .pcap files as the one I access is quite small. In it, we have TCP and FTP, something we did not have before.
Remember, FTP is one of the ports open. I use tcp.stream eq x to check for interesting dialogs, as well as using Follow -> upstream to see whole dialogs. 

For example, this dialog:

>220 (vsFTPd 3.0.3)
USER nathan
331 Please specify the password.
PASS Buck3tH4TF0RM3!
230 Login successful.
SYST
215 UNIX Type: L8
PORT 192,168,196,1,212,140
200 PORT command successful. Consider using PASV.
LIST
150 Here comes the directory listing.
226 Directory send OK.
PORT 192,168,196,1,212,141
200 PORT command successful. Consider using PASV.
LIST -al
150 Here comes the directory listing.
226 Directory send OK.
TYPE I
200 Switching to Binary mode.
PORT 192,168,196,1,212,143
200 PORT command successful. Consider using PASV.
RETR notes.txt
550 Failed to open file.
QUIT
221 Goodbye.


Luckily, this was the first I opened, and luckily for me, I am interested in solving it, so I actually looked at the contents. We have an FTP connection, where we have the user `nathan` and the password `Buck3tH4TF0RM3!`. Really nice I can proceed to the FTP port now

I run 

>ftp nathan@10.129.98.103
Connected to 10.129.98.103.
220 (vsFTPd 3.0.3)
331 Please specify the password.
>
>Password: 
230 Login successful.
>
>Remote system type is UNIX.
Using binary mode to transfer files.
>
>ftp> ls
>
>229 Entering Extended Passive Mode (|||52173|)
>
>150 Here comes the directory listing.
>
>-r--------    1 1001     1001           33 Jan 25 18:13 user.txt
>
>226 Directory send OK.
>
>ftp> get user.txt
>
>local: user.txt remote: user.txt
>
>229 Entering Extended Passive Mode (|||65197|)
150 Opening BINARY mode data connection for user.txt (33 bytes).
100% |***********************************|    33      259.89 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (4.41 KiB/s)

From there on, I proceed to open the user.txt file in my directory.

Now to the Root Flag. I look for all access rights I have in the FTP enviroment. Therefore I run **dir -a**- Nice command.

>229 Entering Extended Passive Mode (|||14648|)
>
>150 Here comes the directory listing.
>
>drwxr-xr-x    3 1001     1001         4096 May 27  2021 .
>
>drwxr-xr-x    3 0        0            4096 May 23  2021 ..
>
>lrwxrwxrwx    1 0        0               9 May 15  2021 .bash_history -> /dev/null
>
>-rw-r--r--    1 1001     1001          220 Feb 25  2020 .bash_logout
>
>-rw-r--r--    1 1001     1001         3771 Feb 25  2020 .bashrc
>
>drwx------    2 1001     1001         4096 May 23  2021 .cache
>
>-rw-r--r--    1 1001     1001          807 Feb 25  2020 .profile
>
>lrwxrwxrwx    1 0        0               9 May 27  2021 .viminfo -> /dev/null
>
>-r--------    1 1001     1001           33 Jan 25 18:13 user.txt
>
>226 Directory send OK.

This is the complete output

I am not well-versed, so I looked at the official documentation, and besides the file where I found the user.txt, there are no files that I have any rights to. Upcoming for me is therefore to learn how to read such outputs. I do not know what that means except r for read, w for write and x for execut.
