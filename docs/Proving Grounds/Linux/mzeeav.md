# Mzeeav 🔸

## Enumeration

![](../assets/Pasted%20image%2020250409163021.png)

Web server shows a simple app that analyzes a file that you can upload.

![](../assets/Pasted%20image%2020250409234009.png)

Uploading a file

![](../assets/Pasted%20image%2020250409234033.png)

## Initial Access

Feroxbuster finds a /backups folder with a zip of the project.

![](../assets/Pasted%20image%2020250409234303.png)

![](../assets/Pasted%20image%2020250409234314.png)

![](../assets/Pasted%20image%2020250409234349.png)

It only checks the first four bytes so we can add it to a reverse shell php.

![](../assets/Pasted%20image%2020250410010837.png)

![](../assets/Pasted%20image%2020250410010855.png)

![](../assets/Pasted%20image%2020250410010905.png)

## Privilege Escalation

![](../assets/Pasted%20image%2020250410011000.png)

We find a fileS in /opt with suid that is the same as find command. So search find in gtfobins.

![](../assets/Pasted%20image%2020250410024853.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250410024920.png)