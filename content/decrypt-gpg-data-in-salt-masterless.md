+++
draft = false
date = "2017-07-05T10:49:38+09:00"
title = "Decrypt gpg data in salt masterless"
tags = ["salt"]

+++

<!--more-->

You must copy gpgkeys from it to masterless minion If you encrypt data with gpg in salt master.  

---

If gpgkeys is located here in the salt master(default),  
```
root@salt-master:~# ll /etc/salt/gpgkeys/
total 28
drwx------ 2 root root 4096 Sep  4  2015 ./
drwxr-xr-x 9 root root 4096 Jun 22 23:31 ../
-rw------- 1 root root 1194 Sep  4  2015 pubring.gpg
-rw------- 1 root root  600 Jun 30 03:33 random_seed
-rw------- 1 root root 2495 Sep  4  2015 secring.gpg
-rw------- 1 root root 1280 Sep  4  2015 trustdb.gpg
```

You need to copy it to the same path(default) or some place you defined by config file in the masterless minion.  

And you also have to be careful about the permisson when you download it.  
If your ssh user is not root, `/etc/salt/gpgkeys/` is owned by root and you use sftp to download it, you can't download the directory.  

```
$ sftp salt_master
Connected to salt_master.
sftp> get -r /etc/salt/gpgkeys
Fetching /etc/salt/gpgkeys/ to gpgkeys
Retrieving /etc/salt/gpgkeys
remote readdir("/etc/salt/gpgkeys"): Permission denied
/etc/salt/gpgkeys: Failed to get directory contents
```

To solve this, I copied the directory somewhere and change the owner to the ssh user in the salt master.  
Then I could download it successfully with sftp.  
