# Readys 🔸

## Enumeration

![](../assets/Pasted%20image%2020250401193030.png)

Web site at 80 is a wordpress site

![](../assets/Pasted%20image%2020250401193558.png)

Using wpscan we find a local file inclusion in the plugin

![](../assets/Pasted%20image%2020250401195433.png)

![](../assets/Pasted%20image%2020250401195444.png)

So we have alice user

We can see redis config at /etc/redis/redis.conf
![](../assets/Pasted%20image%2020250401203659.png)

So we have `alice` user  and the redis pass `Ready4Redis?`

We can log in in redis

![](../assets/Pasted%20image%2020250401204513.png)

## Initial Access

Searching an rce exploit 

https://github.com/jas502n/Redis-RCE

![](../assets/Pasted%20image%2020250401213340.png)

## Privilege Escalation

Make another reverse shell for using it more stable

![](../assets/Pasted%20image%2020250401214116.png)

![](../assets/Pasted%20image%2020250401214130.png)

Mysql config

![](../assets/Pasted%20image%2020250401214327.png)

![](../assets/Pasted%20image%2020250401214941.png)

`admin:$P$Ba5uoSB5xsqZ5GFIbBnOkXA0ahSJnb0`

Can't crack it

Transfer linpeas.sh

![](../assets/Pasted%20image%2020250401221345.png)

Using pspy64 we see it

![](../assets/Pasted%20image%2020250401222650.png)

![](../assets/Pasted%20image%2020250401223022.png)

We can exploit tar wildcard , but not withs this user, we need alice.

Find writable folder to put a php file and execute it like before with lfi and get a reverse shell.

![](../assets/Pasted%20image%2020250401235624.png)

![](../assets/Pasted%20image%2020250401235632.png)

![](../assets/Pasted%20image%2020250402000244.png)

![](../assets/Pasted%20image%2020250402000303.png)

![](../assets/Pasted%20image%2020250402000336.png)
## Post Exploitation

Get flags

![](../assets/Pasted%20image%2020250402000350.png)