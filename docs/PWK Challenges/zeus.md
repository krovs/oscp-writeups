# Zeus
## Network

**Given:**

- 192.168.167.0/24

**Found:**

- 192.168.167.158 DC
- 192.168.167.159
- 192.168.167.160

## Creds

```
eric.wallows:EricLikesRunning800 (creds for .159)
db_user:Password123! (found in a file on .159)
o.foller:EarlyMorningFootball777 (retrieved using mimikatz from .159)
administrator:97db5c87465431b9091d47a58fe76483 (local admin on .160)
z.thomas:^1+>pdRLwyct]j,CYmyi (found in a document on .160)
```

## Enumeration

Ping sweeping the subnet reveals three machines.

![](assets/Pasted%20image%2020250225234112.png)

## 192.168.167.159

### Enumeration

![](assets/Pasted%20image%2020250225234538.png)

### Initial Access

Using the provided credentials, we log in via WinRM.

![](assets/Pasted%20image%2020250225234448.png)

### Privilege Escalation

We can see that we have all permissions.

Transfer a reverse shell and JuicyPotatoNG, and we can get a system shell.

![](assets/Pasted%20image%2020250226104009.png)

![](assets/Pasted%20image%2020250226104020.png)

### Post Exploitation

Get the proof

![](assets/Pasted%20image%2020250226104110.png)

We see an SQL folder containing `connection.sql` with credentials `db_user:Password123!`.

![](assets/Pasted%20image%2020250226002908.png)

Transfer `adpeas.ps1`, and we identify the DC.

![](assets/Pasted%20image%2020250226095156.png)

Transfer Mimikatz, and we retrieve `o.foller:EarlyMorningFootball777`.

![](assets/Pasted%20image%2020250226104604.png)

## 192.168.167.160 CLIENT02

### Enumeration

![](assets/Pasted%20image%2020250226093305.png)

### Initial Access

Using the credentials for `o.foller`, we use PsExec to access the machine.

![](assets/Pasted%20image%2020250226110902.png)

### Privilege Escalation

We have all privileges, so we use PrintSpoofer.

![](assets/Pasted%20image%2020250226111420.png)

### Post Exploitation

Get the proof

![](assets/Pasted%20image%2020250226111620.png)

Using Mimikatz, we only retrieve the administrator hash, which cannot be cracked with Hashcat.

![](assets/Pasted%20image%2020250226112216.png)

In `z.thomas`'s home folder, we find a Word document containing credentials:

![](assets/Pasted%20image%2020250226120900.png)

`z.thomas:^1+>pdRLwyct]j,CYmyi`

![](assets/Pasted%20image%2020250226121241.png)

## 192.168.167.158 DC

### Enumeration

![](assets/Pasted%20image%2020250226093438.png)

### Initial Access

Use WinRM with `z.thomas`'s credentials found on .160.

![](assets/Pasted%20image%2020250226121322.png)

Get the flag

![](assets/Pasted%20image%2020250226134806.png)

### Privilege Escalation

Transfer SharpHound to the target, download the zip, and import it into BloodHound.

![](assets/Pasted%20image%2020250226130214.png)

We see that `z.thomas` has `GenericAll` over `d.chambers`, who is a backup operator. Perform a targeted Kerberoast attack.

```bash
python targetedKerberoast.py -v -d 'zeus.corp' -u 'z.thomas' -p '^1+>pdRLwyct]j,CYmyi' --dc-ip 192.168.167.158
```

![](assets/Pasted%20image%2020250226130728.png)

Hashcat cannot crack it, so we change it. Transfer `powerview.ps1` to the machine.

![](assets/Pasted%20image%2020250226131459.png)

![](assets/Pasted%20image%2020250226131801.png)

Check with `nxc` and log in using Evil-WinRM.

![](assets/Pasted%20image%2020250226132010.png)

`d.chambers` belongs to the backup operators group.

![](assets/Pasted%20image%2020250226132242.png)

We can retrieve passwords using Robocopy.

First, transfer the script file (I had to duplicate end characters for the script to work).

![](assets/Pasted%20image%2020250226134843.png)

![](assets/Pasted%20image%2020250226134923.png)

Now retrieve the `ntds.dit` file using Robocopy.

![](assets/Pasted%20image%2020250226134948.png)

Retrieve the SYSTEM file as well.

![](assets/Pasted%20image%2020250226135016.png)

Download them to the Kali machine, and using `secretsdump`, we retrieve the domain admin's hash.

![](assets/Pasted%20image%2020250226135043.png)

### Post Exploitation

Using the hash, log in to the machine with Evil-WinRM.

![](assets/Pasted%20image%2020250226134617.png)