# Hawat 🔹

## Enumeration

![](../assets/Pasted%20image%2020250325225524.png)

17445 port shows a web page of an issue tracker

![](../assets/Pasted%20image%2020250325225809.png)

We can register users and log in

![](../assets/Pasted%20image%2020250325225944.png)

Port 30455 show us a web with no links

![](../assets/Pasted%20image%2020250325230139.png)

Using feroxbuster we find a nextcloud instance

![](../assets/Pasted%20image%2020250325231552.png)

![](../assets/Pasted%20image%2020250325231604.png)

Trying  admin/admin works

![](../assets/Pasted%20image%2020250325232402.png)

We find the issue tracker project in here

![](../assets/Pasted%20image%2020250325235405.png)

Download and exploring it and we have the mysql password

![](../assets/Pasted%20image%2020250325235439.png)

`issue_user:ManagementInsideOld797`

We can see that priority param is injectable in checkByPriority function

![](../assets/Pasted%20image%2020250327094631.png)

So going to the web

![](../assets/Pasted%20image%2020250327094659.png)

Get is not allowed so, post? Capture the request with caido

Changing GET to POST

![](../assets/Pasted%20image%2020250327101541.png)

And adding priority param, we got to 200

![](../assets/Pasted%20image%2020250327101556.png)

We saw with feroxbuster that there is a phpinfo that shows that the root path is /srv/http so we can inject a webshell.

![](../assets/Pasted%20image%2020250327104552.png)

We can inject a web shell

```shell
' union select '<?php echo system($_REQUEST["bingo"]); ?>' into outfile '/srv/http/cmd.php' -- -
```

![](../assets/Pasted%20image%2020250327121912.png)

Encode with https://www.url-encode-decode.com/

Accessing the path

![](../assets/Pasted%20image%2020250327121932.png)

## Post Exploitation

Get the flag

![](../assets/Pasted%20image%2020250327122007.png)

