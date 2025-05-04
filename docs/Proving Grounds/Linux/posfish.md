# Postfish 🔸

## Enumeration

![](../assets/Pasted%20image%2020250325153256.png)

Add postfish.off to /etc/hosts

Web server is a template 

![](../assets/Pasted%20image%2020250325153439.png)

with people

![](../assets/Pasted%20image%2020250325153459.png)

Make a users file and put the names, put also the departments.

![](../assets/Pasted%20image%2020250325164542.png)

## Initial Access

Using hydra, first use the same list for user and password

![](../assets/Pasted%20image%2020250325164612.png)

We have `sales:sales`

We can install evolution and put the email to view emails

![](../assets/Pasted%20image%2020250325172636.png)

Or with pop3 port, connecting with telnet.

![](../assets/Pasted%20image%2020250325191355.png)

So we need to send an email looking as it to the users. Start a listener on 80 and execute sendEmail.

![](../assets/Pasted%20image%2020250325191519.png)

brian moore send us credentials

`brian.moore:EternaLSunshinE`

We can try ssh

![](../assets/Pasted%20image%2020250325192417.png)

## Privilege Escalation

Get the flag

![](../assets/Pasted%20image%2020250325192547.png)

Execute linpeas and we see

![](../assets/Pasted%20image%2020250325222954.png)

![](../assets/Pasted%20image%2020250325223019.png)

This file executes altermime and adds content to all emails, so if we put a reverse shell and send the email.

![](../assets/Pasted%20image%2020250325223632.png)

Using sudo -l

![](../assets/Pasted%20image%2020250325223726.png)

![](../assets/Pasted%20image%2020250325223749.png)

![](../assets/Pasted%20image%2020250325223816.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250325223834.png)