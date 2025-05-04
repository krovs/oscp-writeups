# Sorcerer 🔸

## Enumeration

![](../assets/Pasted%20image%2020250330123225.png)

![](../assets/Pasted%20image%2020250330123232.png)

Web server shows a 404 not found, let's try feroxbuster

Port 7742 shows a login form, but is not working, the front always shows invalidlogon.

![](../assets/Pasted%20image%2020250330125803.png)

![](../assets/Pasted%20image%2020250330125819.png)

Ferox finds a zipfiles path

![](../assets/Pasted%20image%2020250330123713.png)

We find a id_rsa key in max's zip

![](../assets/Pasted%20image%2020250330124108.png)

We also find a tomcat password

![](../assets/Pasted%20image%2020250330131824.png)

A wrapper that prevents ssh access, only scp

![](../assets/Pasted%20image%2020250330162325.png)

## Initial Access

The authorized keys use this wraper so we can remove it and scp it to replace it, and connect with ssh normally

![](../assets/Pasted%20image%2020250330162501.png)

![](../assets/Pasted%20image%2020250330162558.png)


![](../assets/Pasted%20image%2020250330162524.png)

## Privilege Escalation

get flag

![](../assets/Pasted%20image%2020250330164953.png)

![](../assets/Pasted%20image%2020250330164725.png)

![](../assets/Pasted%20image%2020250330164750.png)

![](../assets/Pasted%20image%2020250330164800.png)
## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250330164808.png)