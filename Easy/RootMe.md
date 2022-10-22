# [RootMe](https://tryhackme.com/room/rrootme)

> Difficulty: Easy
# Task 1

Deploy your machine!

# Task 2

## -Scan the machine, how many ports are open?
  
We run an NMAP with  
```
nmap -sV targetmachineIP
```

+ `-sV`: Not only checks for open ports but the version of the service running.

![image](https://user-images.githubusercontent.com/115602464/197348592-c75191b5-c5de-422d-9507-7ac67040a0c7.png)

We can see two ports are open. Port 22 for SSH and port 80 for a server running Apache 2.4.29.

## -What version of Apache is running?

The above image shows that Apache 2.4.29 is running.

## -What service is running on port 22?

The above image shows that SSH is running on port 22.

## -Find directories on the web server using the GoBuster tool.

Lets use a tool called dirb to enumerate directories at the targetmachineIP address.

```
dirb http://10.10.165.210/ /usr/share/wordlists/dirb/big.txt 
```

+ `dirb`: We use dirb to scan the targetmachineIP using dirbs big.txt wordlist.

## -What is the hidden directory?

![image](https://user-images.githubusercontent.com/115602464/197349682-691cb38b-5499-419b-ab70-715fad6ac21d.png)

We can see a /panel/ directory!

We can see there are two directories that stand out. (/panel/ and /uploads/)
The reason /panel/ is the one we are interested in is because it brings us to a file upload form.

# Task 3

Find a form to upload and get a reverse shell, and find the flag.

<sub>Hint: Search for "file upload bypass" and "PHP reverse shell".</sub>

## -user.txt

Lets head over to targetmachineIP/panel/

![image](https://user-images.githubusercontent.com/115602464/197354106-7853af06-dc6d-44a7-9468-c6fc3eef0c97.png)

Once here we can see that we are given an upload form. 

The hint is really helpful here in hinting that we need a php reverse shell.

The idea here is that because we are able to access the upload form AND /uploads/, we can upload a reverse shell file and execute it by accessing it in /uploads/. This would give us a reverse shell into the system as a low privilege user.

Lets head to [RevShells](https://www.revshells.com)

Once here, we want to use the IP address of our attacker machine and a port of our choosing, lets use 4444.

We can find our own IP by utilizing `ifconfig`.

![image](https://user-images.githubusercontent.com/115602464/197353522-cecff534-10ea-4e20-aa95-adcc7d03c281.png)

Look for eth0 within the ifconfig output. The inet address here is what we will use for our reverse shell.

We also want to select which reverse shell we want to use. In this case, because I have previous experience with it, I chose `PHP PentestMonkey`.

Our page should look something like this, with the IP adjusted to your machines IP.

![image](https://user-images.githubusercontent.com/115602464/197355378-debafb52-ecad-4a5f-bf2c-0ca3d0233953.png)

Lets copy the code we are given and paste it into a file called reverse.php. Head back to /panel/ and upload your reverse shell!

![image](https://user-images.githubusercontent.com/115602464/197355322-5ac34eb2-356b-4165-8265-9cbb523c60cd.png)

We get an error! Checking /uploads/ we can see that our file was not uploaded.

Doing a quick google translate: `PHP is not allowed!`

Looks like the upload form is validating files before they are uploaded. This means that the admin configured the form to not accept files that could be potentially dangerous, such as PHP. Thankfully, there are many ways to prey on misconfigurations and bypass any validation. [hacktricks.xyz](https://book.hacktricks.xyz/pentesting-web/file-upload)

This is a great resource for file upload methodologies. Lets go with renaming our file to reverse.php2. The file upload form might be blocking .php but not .php2/3/4/etc.

![image](https://user-images.githubusercontent.com/115602464/197355701-fb3bb65e-7328-4668-989b-df24916b48bb.png)

Success! Lets check /uploads/ and make sure our file was actually uploaded.

![image](https://user-images.githubusercontent.com/115602464/197355730-bd320aad-3ab8-4a87-afad-fcb159a63db0.png)

Now that our reverse shell is uploaded, lets setup a listener to intercept the connection from the reverse shell.

+ `dirb`: A listener 
