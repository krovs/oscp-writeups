# Nukem 🔸

## Enumeration

![](../assets/Pasted%20image%2020250322134333.png)

On webserver 80 there is a wordpress blog

![](../assets/Pasted%20image%2020250322134419.png)

Another wp on port 13000

![](../assets/Pasted%20image%2020250322134603.png)

We scan wordpress with wp scan and we have

![](../assets/Pasted%20image%2020250323133920.png)

## Initial Access

Going to the link we get the script, we have to edit it and remove in the payload section, the password thing.

![](../assets/Pasted%20image%2020250323134005.png)

![](../assets/Pasted%20image%2020250323134023.png)

Now we have a web shell, using python we make a reverse shell

![](../assets/Pasted%20image%2020250323134100.png)

![](../assets/Pasted%20image%2020250323134237.png)

Get the flag
## Privilege Escalation

Now showing suid programs we have dosbox and searching in gtfobins

![](../assets/Pasted%20image%2020250323134331.png)

We can write in root files

![](../assets/Pasted%20image%2020250323134430.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250323134447.png)
