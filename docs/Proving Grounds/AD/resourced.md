# Resourced 🔸

## Enumeration

![](../assets/Pasted%20image%2020250416110941.png)

Using rpclient we can enumerate the users and descriptions

![](../assets/Pasted%20image%2020250416135633.png)

So we have `v.ventz:HotelCalifornia194!`

![](../assets/Pasted%20image%2020250416142136.png)

Checking smb shares we have some stuff

![](../assets/Pasted%20image%2020250416142304.png)

![](../assets/Pasted%20image%2020250416185753.png)

![](../assets/Pasted%20image%2020250416190126.png)

We can get system and ntds.dit to get lsa credentials

![](../assets/Pasted%20image%2020250416190158.png)

Put them in a text file and use hashcat

nothing, let's pass the hash to move laterally

## Initial Access

![](../assets/Pasted%20image%2020250416191851.png)

Using evil-winrm 

![](../assets/Pasted%20image%2020250416191910.png)

## Privilege Escalation

Get the flag

![](../assets/Pasted%20image%2020250416192043.png)

Use bloodhound-python to get the zip

![](../assets/Pasted%20image%2020250417203751.png)

![](../assets/Pasted%20image%2020250417225834.png)


This user has genericall to the machine so we can perform a rbcd attack

https://github.com/tothi/rbcd-attack

![](../assets/Pasted%20image%2020250418121053.png)

![](../assets/Pasted%20image%2020250418121105.png)

![](../assets/Pasted%20image%2020250418121115.png)

![](../assets/Pasted%20image%2020250418121130.png)
## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250418121140.png)