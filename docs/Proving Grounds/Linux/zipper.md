# Zipper 🔺

## Enumeration

![](../assets/Pasted%20image%2020250419114322.png)

The web is an app that zips all files you upload

![](../assets/Pasted%20image%2020250419114634.png)

## Initial Access

We can see a LFI clicking on home and using a php filter we can see php code.

![](../assets/Pasted%20image%2020250419135029.png)

![](../assets/Pasted%20image%2020250419135041.png)

The filter is removing the last extension.

We can upload a reverse shell to be zipped and then execute it abusing zip slip without the extension.

![](../assets/Pasted%20image%2020250419135149.png)

![](../assets/Pasted%20image%2020250419135201.png)

![](../assets/Pasted%20image%2020250419135254.png)

## Privilege Escalation

Get the flag

![](../assets/Pasted%20image%2020250419135310.png)

There is a backup script with logs on /opt, read the logs.

![](../assets/Pasted%20image%2020250419135336.png)

![](../assets/Pasted%20image%2020250419135346.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250419135359.png)