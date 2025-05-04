# Nibbles 🔸

## Enumeration

![](../assets/Pasted%20image%2020250319185357.png)

## Initial Access

Searching we find https://github.com/squid22/PostgreSQL_RCE

Clone, create venv and install requirements.

Edit host and port and use port 80 on listener.

![](../assets/Pasted%20image%2020250319202545.png)

![](../assets/Pasted%20image%2020250319202553.png)

## Privilege Escalation

Get local flag

![](../assets/Pasted%20image%2020250319202808.png)

Find has suid privs

![](../assets/Pasted%20image%2020250319203721.png)

Looking in gtfobins

![](../assets/Pasted%20image%2020250319203753.png)

## Postexploitation

Get the flag

![](../assets/Pasted%20image%2020250319203813.png)