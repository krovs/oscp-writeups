# Craft 🔸

## Enumeration

![](../assets/Pasted%20image%2020250405173521.png)

We have a template page at port 80

![](../assets/Pasted%20image%2020250405173442.png)

There is an uploader that only accepts .odt files.

![](../assets/Pasted%20image%2020250406190515.png)

![](../assets/Pasted%20image%2020250405173706.png)

CMS is made with umbraco

If we upload a odt file, the file dissapears in seconds. This is a phishing lab it seems.

## Initial Access

Using this malicious odt generator we can craft a odt with a macro to connect back on open.

https://github.com/0bfxgh0st/MMG-LO

![](../assets/Pasted%20image%2020250406190600.png)

Start a listener and wait

![](../assets/Pasted%20image%2020250406190627.png)

## Privilege Escalation

Now we can put the php reverse shell manually in uploads folder to pivot to apache user

![](../assets/Pasted%20image%2020250406190740.png)

![](../assets/Pasted%20image%2020250406190756.png)

Start a listener, click it and we have shell as apache

![](../assets/Pasted%20image%2020250406190836.png)

![](../assets/Pasted%20image%2020250406190901.png)

We can impersonate, so transfer printspoofer and get a root shell

![](../assets/Pasted%20image%2020250406190926.png)
## Post Exploitation

Get  the flag

![](../assets/Pasted%20image%2020250406190936.png)