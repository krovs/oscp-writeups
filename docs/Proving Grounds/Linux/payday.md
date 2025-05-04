# Payday 🔸

## Enumeration

![](../assets/Pasted%20image%2020250317192645.png)

The web page is

![](../assets/Pasted%20image%2020250317192943.png)

## Initial Access

We can log in with `admin:admin`

Using feroxbuster we discover /admin and using admin admin we are inside.

![](../assets/Pasted%20image%2020250317193602.png)

Go to template editor and upload a php reverse shell with pthlml as seen in

![](../assets/Pasted%20image%2020250317194757.png)

Once upload go to http://[victim]/skins/shell.phtml after setting a reverse shell

![](../assets/Pasted%20image%2020250317194950.png)

## Privilege Escalation

Get local.txt

![](../assets/Pasted%20image%2020250317195318.png)

enumerating users we see patrick and testing patrick with pass patrick

![](../assets/Pasted%20image%2020250317210147.png)

And patrick has all privileges

![](../assets/Pasted%20image%2020250317210236.png)

![](../assets/Pasted%20image%2020250317210249.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250317210324.png)