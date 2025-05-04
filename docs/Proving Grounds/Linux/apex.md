# Apex 🔸

## Enumeration

![](../assets/Pasted%20image%2020250328134152.png)

Web server shows a page of medial stuff

![](../assets/Pasted%20image%2020250328134243.png)

Let's add apex.offsec to hosts

We have four potential users

![](../assets/Pasted%20image%2020250328134432.png)

And a scheduler app

![](../assets/Pasted%20image%2020250328134501.png)

We find documents in smb share that can be accessed without user/pass

![](../assets/Pasted%20image%2020250328134826.png)

With feroxbuster we find a filemanager path, serving the same files as the smb share in a documents folder

![](../assets/Pasted%20image%2020250328135236.png)

![](../assets/Pasted%20image%2020250328135300.png)

We can upload php files and the app won't show them.

Searching with searchsploit

![](../assets/Pasted%20image%2020250330121414.png)

We can search in github where sqlconf is located, but we need to edit the script and put Documents in the path where the file is  being copied to, this way we can see it in smb share, here we can't see php files.

![](../assets/Pasted%20image%2020250330121051.png)

![](../assets/Pasted%20image%2020250330121103.png)

![](../assets/Pasted%20image%2020250330121115.png)

## Initial Access

We can access the db with the crednetials and get users

![](../assets/Pasted%20image%2020250330121142.png)

![](../assets/Pasted%20image%2020250330121159.png)

![](../assets/Pasted%20image%2020250330121208.png)

Using searchsploit to get a openemr exploit 

![](../assets/Pasted%20image%2020250330121323.png)

![](../assets/Pasted%20image%2020250330121251.png)

![](../assets/Pasted%20image%2020250330121454.png)

## Privilege Escalation

The password is the same as before 

![](../assets/Pasted%20image%2020250330121512.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250330121713.png)