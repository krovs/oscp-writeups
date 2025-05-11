# Mice 🔹

## Enumeration

![](../assets/Pasted%20image%2020250423122020.png)

## Initial Access

We find an exploit for remotemouse https://github.com/p0dalirius/RemoteMouse-3.008-Exploit

I executed this in two parts, first get a reverse shell from httpserver and then execute it.

![](../assets/Pasted%20image%2020250423170505.png)

![](../assets/Pasted%20image%2020250423170512.png)

![](../assets/Pasted%20image%2020250423170524.png)


Get the flag

![](../assets/Pasted%20image%2020250423170554.png)

## Privilege Escalation

Searching in home folder we find a filezilla base 64 password

![](../assets/Pasted%20image%2020250423170726.png)

![](../assets/Pasted%20image%2020250423170802.png)

`ControlFreak11`


This user bleongs to remote users so maybe we can access it using xfreerpd

![](../assets/Pasted%20image%2020250423170934.png)


![](../assets/Pasted%20image%2020250423171742.png)

![](../assets/Pasted%20image%2020250423171809.png)

remotemouse is running with admin privileges so open it from the task manager, go to settings and change 

![](../assets/Pasted%20image%2020250423171844.png)

In the folder dialog put cmd in the search bar

![](../assets/Pasted%20image%2020250423171919.png)

![](../assets/Pasted%20image%2020250423171928.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250423171941.png)