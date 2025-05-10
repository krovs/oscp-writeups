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
chen:freedom (AS-REP roastable user from .163)

lisa:905ae9b4d957545fb7b9ea0c4333247b (from .163)
administrator:3bcdd818f7ec942ac91aa30d8db71927 (from .162)
```

## Enumeration

Ping sweeping the subnet reveals three machines.

![](assets/Pasted%20image%2020250226200931.png)

## 192.168.167.163

### Enumeration

![](assets/Pasted%20image%2020250226201135.png)

### Initial Access

Using the provided credentials, we access the machine via WinRM.

![](assets/Pasted%20image%2020250226202054.png)

### Privilege Escalation

Eric has the `SeImpersonatePrivilege`, so we use PrintSpoofer. Create an MSFVenom reverse shell, upload it to the machine, and start a listener.

![](assets/Pasted%20image%2020250226203819.png)

![](assets/Pasted%20image%2020250226203827.png)

### Post Exploitation

Get the flags

![](assets/Pasted%20image%2020250226204038.png)

Transfer Mimikatz and extract credentials.

![](assets/Pasted%20image%2020250226204324.png)

We have Lisa's hash. Let's try Hashcat, but it doesn't yield results.

Transfer ADPEAS and WinPEAS for further enumeration.

![](assets/Pasted%20image%2020250226204951.png)

Using Hashcat:

![](assets/Pasted%20image%2020250226205103.png)

We retrieve `chen:freedom`.

Transfer SharpHound and analyze the results in BloodHound.

![](assets/Pasted%20image%2020250226212250.png)

Lisa has `AllExtendedRights` over Jackie, so we can change the password using `pth-net`:

![](assets/Pasted%20image%2020250227000002.png)

We can now access the machine via WinRM.

## 192.168.167.162 DC02

### Enumeration

![](assets/Pasted%20image%2020250226201916.png)

### Initial Access

We use Evil-WinRM to access the machine after changing Jackie's password.

![](assets/Pasted%20image%2020250227000126.png)

### Privilege Escalation

Get the flag

![](assets/Pasted%20image%2020250227000305.png)

The first thing we notice is Jackie's permissions and group membership, including `SeBackupPrivilege` and `Backup Operators`. We can use ShadowCopy to retrieve `ntds.dit`.

Create and transfer a DiskShadow script like this:

![](assets/Pasted%20image%2020250227001040.png)

![](assets/Pasted%20image%2020250227001102.png)

Now retrieve `ntds.dit` using Robocopy along with the SYSTEM hive.

![](assets/Pasted%20image%2020250227001138.png)

![](assets/Pasted%20image%2020250227001206.png)

Using `secrets-dump`:

![](assets/Pasted%20image%2020250227212940.png)

We can re-enter the machine with the administrator hash using Evil-WinRM.

![](assets/Pasted%20image%2020250227001622.png)

### Post Exploitation

Get the flag

![](assets/Pasted%20image%2020250227001649.png)

## 192.168.167.161 DC01

### Enumeration

![](assets/Pasted%20image%2020250226201849.png)

### Initial Access

With the `krbtgt` hash, we can forge a Golden Ticket to access the machine. We need the parent SID, which can be retrieved using Mimikatz:

![](assets/Pasted%20image%2020250227212717.png)

Add the subdomain and domain to the hosts file.

![](assets/Pasted%20image%2020250227213058.png)

Create the ticket and export it.

![](assets/Pasted%20image%2020250227212753.png)

![](assets/Pasted%20image%2020250227213020.png)

Access DC01.

![](assets/Pasted%20image%2020250227213221.png)

### Post Exploitation

Get the flag

![](assets/Pasted%20image%2020250227213315.png)