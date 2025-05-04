# Spx 🔸

## Enumeration

![](../assets/Pasted%20image%2020250421012600.png)

We have a tiny file manager login page

![](../assets/Pasted%20image%2020250421171137.png)

None of the default credentials works. Using feroxbuster we have a phpinfo.

![](../assets/Pasted%20image%2020250421171206.png)

Looking at the phpinfo we see spx

![](../assets/Pasted%20image%2020250421171242.png)

Searching the web we find an issue for a path traversal https://github.com/NoiseByNorthwest/php-spx/issues/251

![](../assets/Pasted%20image%2020250421171340.png)

So using caido we can use the spx key filtered by phpinfo and

![](../assets/Pasted%20image%2020250421172026.png)


Looking at tinyfilemanager repository, we find that the users are hardcoded in the index

![](../assets/Pasted%20image%2020250421172501.png)

So we go to the index and

![](../assets/Pasted%20image%2020250421172516.png)

```shell
$auth_users = array(
    'admin' => '$2y$10$7LaMUa8an8NrvnQsj5xZ3eDdOejgLyXE8IIvsC.hFy1dg7rPb9cqG',
    'user' => '$2y$10$x8PS6i0Sji2Pglyz7SLFruYFpAsz9XAYsdiPyfse6QDkB/QsdShxi'
);
```

Using hashcat

![](../assets/Pasted%20image%2020250421185907.png)

## Initial Access

Now we can access the file manager and upload a reverse shell

![](../assets/Pasted%20image%2020250421190024.png)

![](../assets/Pasted%20image%2020250421190316.png)

## Privilege Escalation

We can pivot to profiler user using lowprofile password from before

![](../assets/Pasted%20image%2020250421194949.png)

Get the flag

![](../assets/Pasted%20image%2020250421194958.png)

we can use make with sudo

![](../assets/Pasted%20image%2020250421195034.png)

Create a Makefile

```shell
install:
	chmod +s /bin/bash
```

![](../assets/Pasted%20image%2020250421202502.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250421202511.png)