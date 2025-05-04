# Pelican 🔸

## Enumeration

![](../assets/Pasted%20image%2020250313233355.png)

![](../assets/Pasted%20image%2020250313233506.png)

We see a exhibitor for  zookeeper on 8080

![](../assets/Pasted%20image%2020250313235306.png)

## Initial Access

Searching we have

![](../assets/Pasted%20image%2020250313235326.png)

So adding a nc in the correct field

![](../assets/Pasted%20image%2020250313235406.png)

We have access

![](../assets/Pasted%20image%2020250313235427.png)

## Privilege Escalation

Get the flag

![](../assets/Pasted%20image%2020250313235441.png)

With  sudo -l we can see gcore privileges

![](../assets/Pasted%20image%2020250314002203.png)

![](../assets/Pasted%20image%2020250314002230.png)

with ps aux we search for a process with password and i see password store

![](../assets/Pasted%20image%2020250314002331.png)

So using that pid, and then using strings on the file we have a password

![](../assets/Pasted%20image%2020250314002431.png)

Now we can execute commands as root ( i couldn't switch to the user; next time use a bash connections instead of nc)

![](../assets/Pasted%20image%2020250314002512.png)

![](../assets/Pasted%20image%2020250314002539.png)

## PostExploitation

Get the flag

![](../assets/Pasted%20image%2020250314002600.png)

