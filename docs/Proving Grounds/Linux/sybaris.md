# Sybaris 🔸

## Enumeration

![](../assets/Pasted%20image%2020250330170303.png)

Web server has a php blog

![](../assets/Pasted%20image%2020250330170412.png)


![](../assets/Pasted%20image%2020250330170539.png)

The blog is made with htmly and Pablo

![](../assets/Pasted%20image%2020250330170930.png)

ftp shows an exit pub folder

Using redis-cli we can connect, is open.

We can upload a a redis module to execute system  commands, ulpoad it to the  ftp server and load it with redis.

https://book.hacktricks.wiki/en/network-services-pentesting/6379-pentesting-redis.html#load-redis-module
https://github.com/n0b0dyCN/RedisModules-ExecuteCommand#

![](../assets/Pasted%20image%2020250330204047.png)

![](../assets/Pasted%20image%2020250330210000.png)

load module from default public vftpd

![](../assets/Pasted%20image%2020250330210034.png)

execute reverse shell

![](../assets/Pasted%20image%2020250330204226.png)

![](../assets/Pasted%20image%2020250330204313.png)

## Privilege Escalation

Get the flag

![](../assets/Pasted%20image%2020250330221230.png)

Searching passwords in the blog project we find pablo's

![](../assets/Pasted%20image%2020250330221107.png)

Better to connect via SSH

Transfer linpeas

![](../assets/Pasted%20image%2020250330222017.png)

![](../assets/Pasted%20image%2020250330222124.png)

Compile a shared object with a malicious code and put it in /usr/local/lib/dev and wait

![](../assets/Pasted%20image%2020250330233851.png)

![](../assets/Pasted%20image%2020250330233929.png)

## Post Exploitation

get the flag

![](../assets/Pasted%20image%2020250330233942.png)