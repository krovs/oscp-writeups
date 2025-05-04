# Poseidon
## Network

**Given:**

- 192.168.167.0/24

**Found:**

- 192.168.167.162 DC
- 192.168.167.163
- 192.168.167.161

## Creds

```
eric.wallows:EricLikesRunning800 (creds for .163)
lisa:905ae9b4d957545fb7b9ea0c4333247b (hash from mimikatz on .163)
chen:freedom (asreproastable user from .163)

lisa:905ae9b4d957545fb7b9ea0c4333247b (from .163)
administrator:3bcdd818f7ec942ac91aa30d8db71927 (from .162)
```

## Enumeration

Ping sweeping the subnet gives three machines.

![](assets/Pasted%20image%2020250226200931.png)

## 192.168.167.163

### Enumeration

![](assets/Pasted%20image%2020250226201135.png)

### Initial Access

Using the provided credentials we enter via winrm

![](assets/Pasted%20image%2020250226202054.png)

### Privilege Escalation

Eric has seimpersonate so we use printspoofer. Create a msfvenom reverse shell and upload it to the machine and start a listener.

![](assets/Pasted%20image%2020250226203819.png)

![](assets/Pasted%20image%2020250226203827.png)

### Post Exploitation

Get the flags

![](assets/Pasted%20image%2020250226204038.png)

Transfer mimikatz and 

![](assets/Pasted%20image%2020250226204324.png)

We have lisa's hash, let's try hashcat, but nothing.

Transfer adpeas and winpeas.

![](assets/Pasted%20image%2020250226204951.png)

Using hashcat

![](assets/Pasted%20image%2020250226205103.png)

so we have `chen:freedom`

Transfer sharphound and get the result back to bloodhound.

![](assets/Pasted%20image%2020250226212250.png)

Lisa has allextendedrights over jackie so we can change the password with pth-net:

![](assets/Pasted%20image%2020250227000002.png)

And now winrm to the machine

## 192.168.167.162 DC02

### Enumeration

![](assets/Pasted%20image%2020250226201916.png)

### Initial Access

We can evil-winrm to the machine after changing jackie's password.

![](assets/Pasted%20image%2020250227000126.png)

### Privilege Escalation

Get the flag

![](assets/Pasted%20image%2020250227000305.png)

Frist thing we notice are jackie's permissions and group, sebackupprivilege and backup operators.
So we can use shadowcopy to get ndist.nit.

Create and transfer a diskshadow script like:

![](assets/Pasted%20image%2020250227001040.png)

![](assets/Pasted%20image%2020250227001102.png)

Now get ntds.dit with robocopy and the system.

![](assets/Pasted%20image%2020250227001138.png)

![](assets/Pasted%20image%2020250227001206.png)

And using secrets-dump:

![](assets/Pasted%20image%2020250227212940.png)

We can reenter with administrator hash using evil-winrm.

![](assets/Pasted%20image%2020250227001622.png)

### Post Exploitation

We get the flag

![](assets/Pasted%20image%2020250227001649.png)

## 192.168.167.161 DC01

### Enumeration

![](assets/Pasted%20image%2020250226201849.png)

### Initial Access

Having the krbtgt hash we can forge a golden ticket to access to the machine. We need the parent SID. We can use mimikatz as:

![](assets/Pasted%20image%2020250227212717.png)

Add sub and domain to host file

![](assets/Pasted%20image%2020250227213058.png)

Create the ticket and export it

![](assets/Pasted%20image%2020250227212753.png)

![](assets/Pasted%20image%2020250227213020.png)

Access DC01

![](assets/Pasted%20image%2020250227213221.png)

### Post Exploitation

Get the flag

![](assets/Pasted%20image%2020250227213315.png)