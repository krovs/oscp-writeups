# Cockpit 🔸

## Enumeration

![](../assets/Pasted%20image%2020250324170802.png)

9090 port shows the cockpit service page

![](../assets/Pasted%20image%2020250324171014.png)

And 80 shows a web page

![](../assets/Pasted%20image%2020250324171100.png)

Using feroxbuster with -x php we find login.php

![](../assets/Pasted%20image%2020250324192352.png)

## Initial Access

And the web

![](../assets/Pasted%20image%2020250324192406.png)

It seems that there is a possible sqli

![](../assets/Pasted%20image%2020250324192458.png)

putting `admin'-- -` it works

![](../assets/Pasted%20image%2020250324193553.png)

![](../assets/Pasted%20image%2020250324195254.png)

enter and go to terminal

![](../assets/Pasted%20image%2020250324195432.png)

## Privilege Escalation

get the flag

with sudo -l we see that we have permissions to run tar with wildcard

![](../assets/Pasted%20image%2020250324200330.png)

So we create in /tmp a checkpoint and a action to put suid bit to bash

![](../assets/Pasted%20image%2020250324200416.png)

![](../assets/Pasted%20image%2020250324200443.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250324200510.png)

