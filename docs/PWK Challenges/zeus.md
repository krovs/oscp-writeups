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
db_user:Password123! (in a file in .159)
o.foller:EarlyMorningFootball777 (mimikatzed from .159)
administrator:97db5c87465431b9091d47a58fe76483 (local admin at .160)
z.thomas:^1+>pdRLwyct]j,CYmyi (use doc at .160)
```

## Enumeration

Ping sweeping the subnet we get three machines.

![](assets/Pasted%20image%2020250225234112.png)

## 192.168.167.159

### Enumeration

![](assets/Pasted%20image%2020250225234538.png)

### Initial Access

Using the provided credentials, we log in via winrm.

![](assets/Pasted%20image%2020250225234448.png)

### Privilege Escalation

We can see that we have all permissions

Transfer a reverse shell and juicypotatong and we can get a system shell.

![](assets/Pasted%20image%2020250226104009.png)

![](assets/Pasted%20image%2020250226104020.png)

### Post Exploitation

Get the proof

![](assets/Pasted%20image%2020250226104110.png)

We see a sql folder and inside connection.sql with credentials `db_user:Password123!`

![](assets/Pasted%20image%2020250226002908.png)

Transfer adpeas.ps1 and we have the DC

![](assets/Pasted%20image%2020250226095156.png)

Transfer mimikatz and and we have `o.foller:EarlyMoringFootball777`

![](assets/Pasted%20image%2020250226104604.png)


## 192.168.167.160 CLIENT02

### Enumeration

![](assets/Pasted%20image%2020250226093305.png)

### Initial Access

Using found credentials for o.foller we psexec to the machine

![](assets/Pasted%20image%2020250226110902.png)

### Privilege Escalation

We have all privs so we can use printspoofer

![](assets/Pasted%20image%2020250226111420.png)

### Post Exploitation

Get the proof

![](assets/Pasted%20image%2020250226111620.png)

Making a mimikatz we only get administrator has and can't be hashcat

![](assets/Pasted%20image%2020250226112216.png)

We find in z.thomas home folder a word document with credentials

![](assets/Pasted%20image%2020250226120900.png)

`z.thomas:^1+>pdRLwyct]j,CYmyi`

![](assets/Pasted%20image%2020250226121241.png)

## 192.168.167.158 DC

### Enumeration

![](assets/Pasted%20image%2020250226093438.png)

### Initial Access

Winrm using z.thomas credentials found on .160

![](assets/Pasted%20image%2020250226121322.png)

Get the flag

![](assets/Pasted%20image%2020250226134806.png)

### Privilege Escalation

Transfer sharphound to the target and download the zip and import it into bloodhound.

![](assets/Pasted%20image%2020250226130214.png)

We can see that z.thomas has genericAll over d.chambers, and he is a backup operator, so we'll perform a targeted kerberoast attack.

```bash
python targetedKerberoast.py -v -d 'zeus.corp' -u 'z.thomas' -p '^1+>pdRLwyct]j,CYmyi' --dc-ip 192.168.167.158
```

![](assets/Pasted%20image%2020250226130728.png)

Hashcat can't crack it so we are gonna have to change it.
Transfer powerview.ps1 to the machine.

![](assets/Pasted%20image%2020250226131459.png)

![](assets/Pasted%20image%2020250226131801.png)

Check with nxc and enter using evil-winrm

![](assets/Pasted%20image%2020250226132010.png)

d.chambers belongs to backup operators

![](assets/Pasted%20image%2020250226132242.png)

So we can try to get passwords using robocopy

First transfer the script file (i had to duplicate endchars for the script to work)

![](assets/Pasted%20image%2020250226134843.png)

![](assets/Pasted%20image%2020250226134923.png)

Now get the ntds.dit with robocopy

![](assets/Pasted%20image%2020250226134948.png)

Get the system too

![](assets/Pasted%20image%2020250226135016.png)

Download them to the kali machien and using secretsdump we got the domain admin's hash

![](assets/Pasted%20image%2020250226135043.png)

### Post Exploitation

Using the hash we can evil-winrm to the machine

![](assets/Pasted%20image%2020250226134617.png)