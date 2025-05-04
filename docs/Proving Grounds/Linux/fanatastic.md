# Fanatastic 🔹

## Enumeration

![](../assets/Pasted%20image%2020250404094543.png)

We have a prometheus + grafana stack

![](../assets/Pasted%20image%2020250404094926.png)

## Initial Access

Searching for exploit we have a path traversal one 

![](../assets/Pasted%20image%2020250404132329.png)

We can read grafana db and get data source credentials from /var/lib/grafana/grafana.db

![](../assets/Pasted%20image%2020250404132428.png)

![](../assets/Pasted%20image%2020250404132440.png)

Searching for an exploit to decrypt it we have  https://github.com/Sic4rio/Grafana-Decryptor-for-CVE-2021-43798

![](../assets/Pasted%20image%2020250404132551.png)

SSH with credentials

![](../assets/Pasted%20image%2020250404132619.png)

## Privilege Escalation

The user belongs to disk group so we can read root files.

We can read root private key and ssh to the host.

![](../assets/Pasted%20image%2020250404132907.png)

![](../assets/Pasted%20image%2020250404132921.png)

## Post Exploitation

Get the flags

![](../assets/Pasted%20image%2020250404132938.png)