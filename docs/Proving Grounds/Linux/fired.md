# Fired 🔸

## Enumeration

![](../assets/Pasted%20image%2020250419170300.png)

At 9090 we have an openfire login screen. We search and found the exploit.

## Initial Access

https://github.com/miko550/CVE-2023-32315

Execute the exploit, we have new user, login as the user and upload de jar, then go to server tab, server settings and management tool with 123 pass.

![](../assets/Pasted%20image%2020250419184835.png)

![](../assets/Pasted%20image%2020250419192201.png)

![](../assets/Pasted%20image%2020250419192151.png)

Get the flag

![](../assets/Pasted%20image%2020250419192232.png)

## Privilege Escalation

Find all openfire related folders and search for passwords.

![](../assets/Pasted%20image%2020250419205712.png)

![](../assets/Pasted%20image%2020250419205757.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250419205808.png)