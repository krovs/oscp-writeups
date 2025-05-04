# Flu 🔸

## Enumeration

![](../assets/Pasted%20image%2020250408132402.png)

Port 8090 shows a confluence page

Searching for exploits we have this https://github.com/jbaines-r7/through_the_wire

## Initial Access

![](../assets/Pasted%20image%2020250408134307.png)

## Privilege Escalation

Get the flag

![](../assets/Pasted%20image%2020250408135013.png)

Notice the script at /opt

![](../assets/Pasted%20image%2020250408140641.png)

Using pspy64 we see that is being executed periodically

![](../assets/Pasted%20image%2020250408140705.png)

The file is ours so we can add a payload

![](../assets/Pasted%20image%2020250408140741.png)

![](../assets/Pasted%20image%2020250408140753.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250408140804.png)