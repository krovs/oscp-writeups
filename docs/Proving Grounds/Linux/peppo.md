# Peppo 🔺

## Enumeration

![](../assets/Pasted%20image%2020250331191754.png)

Port 8080 shows a redmine service

Trying 'admin:admin' at /login works. The system ask us to change the password, change it to  adminadmin

![](../assets/Pasted%20image%2020250331193344.png)

At port 10000 we have a 

![](../assets/Pasted%20image%2020250331195747.png)

Using feroxbuster we discover some api actions

![](../assets/Pasted%20image%2020250331195817.png)

We have eleanor user from nmap

## Initial Access

Trying ssh with eleanor:eleanor

![](../assets/Pasted%20image%2020250331231054.png)

## Privilege Escalation

We are in a restricted bash and can't execute most  of commands.

![](../assets/Pasted%20image%2020250331232352.png)

Now we can use the commands, get the flag.

![](../assets/Pasted%20image%2020250331232419.png)

`/bin/bash` to upgrade the shell

We have /usr/bin/docker command so https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-docker-socket

![](../assets/Pasted%20image%2020250331235019.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250331235036.png)