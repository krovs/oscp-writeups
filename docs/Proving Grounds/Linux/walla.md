# Walla 🔸

## Enumeration

![](../assets/Pasted%20image%2020250327192638.png)

Going to webserver 8091 we are greeted with user and pass login.
Using feroxbuster we find package.json

![](../assets/Pasted%20image%2020250327194322.png)

If we follow github link, we can see defualt user and password in the installation guide

![](../assets/Pasted%20image%2020250327194400.png)

## Initial Access

We can enter with these credentials

![](../assets/Pasted%20image%2020250327194446.png)

going to System we have a console

![](../assets/Pasted%20image%2020250327194937.png)

Get the flag

![](../assets/Pasted%20image%2020250327201231.png)

## Privilege Escalation

we can execute a python script wifi_reset.py

![](../assets/Pasted%20image%2020250327212959.png)

![](../assets/Pasted%20image%2020250327213017.png)

When executing the script, it can't find wificontroller module, so we can make one

![](../assets/Pasted%20image%2020250327213058.png)

Now if we execute it, /bin/bash will have suid bit

![](../assets/Pasted%20image%2020250327213150.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250327213203.png)