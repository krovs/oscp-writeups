# Nickel 🔸

## Enumeration

![](../assets/Pasted%20image%2020250407194542.png)

Devops dashboard in 8089

## Initial Access

![](../assets/Pasted%20image%2020250407195842.png)

These options calls port 3333 with a invalid token response, but if we change it to post, we have a response.

![](../assets/Pasted%20image%2020250408005127.png)

Password is in base64 -> `NowiseSloopTheory139`

![](../assets/Pasted%20image%2020250408005751.png)

Get the flag

![](../assets/Pasted%20image%2020250408010013.png)

## Privilege Escalation

Get pdf from ftp

![](../assets/Pasted%20image%2020250408012201.png)

It's protected, using pdf2john and john

![](../assets/Pasted%20image%2020250408012343.png)

![](../assets/Pasted%20image%2020250408012708.png)

![](../assets/Pasted%20image%2020250408012738.png)

Using netstat we can see that there is a port 80 open on the inside.

Port forward using ssh

![](../assets/Pasted%20image%2020250408115602.png)

![](../assets/Pasted%20image%2020250408120247.png)

Using the pdf commands.

![](../assets/Pasted%20image%2020250408120211.png)

Let's put a reverse shell

![](../assets/Pasted%20image%2020250408123102.png)

![](../assets/Pasted%20image%2020250408123110.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250408123115.png)