# Shenzi 🔸

## Enumeration

![](../assets/Pasted%20image%2020250411195535.png)

Port 80 shows a xampp page

![](../assets/Pasted%20image%2020250411195212.png)

We can enumerate smb and get some interesting files

![](../assets/Pasted%20image%2020250411223125.png)

![](../assets/Pasted%20image%2020250411223137.png)

## Initial Access

We need the wordpress path, trying shenzi

![](../assets/Pasted%20image%2020250411223201.png)

We can log in with credentials, then edit the 404 theme page and put a reverse shell php, go to a wrong page and

![](../assets/Pasted%20image%2020250411223259.png)

## Privilege Escalation

Get the flag

![](../assets/Pasted%20image%2020250411223319.png)

Transfer winpeas and discover always install elevated missconfig

![](../assets/Pasted%20image%2020250411223425.png)

Make a reverse shell and transfer it to the host

![](../assets/Pasted%20image%2020250411223451.png)

Then execute it 

![](../assets/Pasted%20image%2020250411223516.png)

![](../assets/Pasted%20image%2020250411223524.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250411223539.png)






