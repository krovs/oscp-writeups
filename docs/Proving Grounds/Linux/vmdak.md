# Vmdak 🔸

## Enumeration

![](../assets/Pasted%20image%2020250423233807.png)

Ftp has anonymous access with a jenkins file

![](../assets/Pasted%20image%2020250424020424.png)


Web server shows a prison manag system

![](../assets/Pasted%20image%2020250423234522.png)

## Initial Access

Searching for an exploit we have an sql injection

![](../assets/Pasted%20image%2020250423235208.png)


![](../assets/Pasted%20image%2020250423235136.png)

![](../assets/Pasted%20image%2020250423235220.png)

Inside there is a user with a password

![](../assets/Pasted%20image%2020250423235327.png)

`malcom:RonnyCache001`

And the admin credentials

![](../assets/Pasted%20image%2020250424000033.png)

Now we can upload a php reverse shell using the avatar uploader bypassing the jpg extension like .jpg.php

![](../assets/Pasted%20image%2020250424013942.png)

![](../assets/Pasted%20image%2020250424013850.png)

## Privilege Escalation

We see that there is a user we can pivot to: vmdak

![](../assets/Pasted%20image%2020250424014210.png)

Let's try the password from malcom

![](../assets/Pasted%20image%2020250424014232.png)

Get the flag

![](../assets/Pasted%20image%2020250424014320.png)

We can connect via ssh with vmdak and stabilize the shell with python to have a full interactive shell

![](../assets/Pasted%20image%2020250424014626.png)

Prison web has sql credentials

![](../assets/Pasted%20image%2020250424022625.png)

`sqlCr3ds3xp0seD`

We have jenkins at 8080 and mysql at 3306

In the 3306 db we have malcon data

![](../assets/Pasted%20image%2020250424022914.png)

For jenkins, transfer chisel to the target and make a port forward

![](../assets/Pasted%20image%2020250424171451.png)

![](../assets/Pasted%20image%2020250424171456.png)

Jekins is protected with passwod

![](../assets/Pasted%20image%2020250424171609.png)

Searching jenkins exploits we find a LFI

![](../assets/Pasted%20image%2020250424171429.png)

Run the script with the path for jenkins

![](../assets/Pasted%20image%2020250424171507.png)

Now we can enter

![](../assets/Pasted%20image%2020250424171900.png)

Create a job that puts suid bit to /bin/bash

![](../assets/Pasted%20image%2020250424172121.png)

Build 

![](../assets/Pasted%20image%2020250424172151.png)

Let's check

![](../assets/Pasted%20image%2020250424172303.png)

So

![](../assets/Pasted%20image%2020250424172312.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250424172322.png)