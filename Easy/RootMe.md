# [RootMe](https://tryhackme.com/room/rrootme)

> Difficulty: Easy
# Task 1

Deploy your machine!

# Task 2

## -Scan the machine, how many ports are open?
  
We run an NMAP scan with  
```
nmap -sV targetmachineIP
```

+ `-sV`: Not only checks for open ports but the version of the service running.

![image](https://user-images.githubusercontent.com/115602464/197348592-c75191b5-c5de-422d-9507-7ac67040a0c7.png)

We see two open ports. Port 22 for SSH and port 80 for a server running Apache 2.4.29.

## -What version of Apache is running?

Apache 2.4.29 is running.

## -What service is running on port 22?

SSH is running on port 22.

## -Find directories on the web server using the GoBuster tool.

Lets use a tool called dirb to enumerate directories at the targetmachineIP address.

```
dirb http://10.10.165.210/ /usr/share/wordlists/dirb/big.txt 
```

+ `dirb`: We use dirb to scan the targetmachineIP using dirbs big.txt wordlist.

## -What is the hidden directory?

![image](https://user-images.githubusercontent.com/115602464/197349682-691cb38b-5499-419b-ab70-715fad6ac21d.png)

We discovered a /panel/ directory!

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

Looks like the upload form is validating files before they are uploaded. This means that the admin configured the form to not accept files that could be potentially dangerous, such as PHP. Thankfully, there are many ways to prey on misconfigurations and bypass any validation.

[Hacktricks.xyz](https://book.hacktricks.xyz/pentesting-web/file-upload) is a great resource for file upload methodologies. Lets go with renaming our file to reverse.phtml. The file upload form might be blocking .php but not .phtml/.phps/.shtml/etc.

![image](https://user-images.githubusercontent.com/115602464/197355701-fb3bb65e-7328-4668-989b-df24916b48bb.png)

Success! Lets check /uploads/ and make sure our file was actually uploaded.

![image](https://user-images.githubusercontent.com/115602464/197357416-d21c2a62-d3b8-4d8f-9193-8353b7d9001f.png)

Now that our reverse shell is uploaded, lets setup a listener to intercept the connection from the reverse shell.

```nc -lvnp 4444```
+ `nc`: Netcat is a utility for opening/listening to connections using TCP or UDP.
+ `l`: Specifies that netcat should listen for an incoming connection.
+ `v`: Gives a more verbose output.
+ `n`: Avoids any DNS or service lookups on any specified address, hostname or port.
+ `p`: Specifies the source port that netcat should use.
+ `4444`: This is the port we specified for our reverse shell.

With the listener active, lets head to /uploads/ and select our reverse.phtml file.

![image](https://user-images.githubusercontent.com/115602464/197357524-6d71aff9-c33a-4a1a-b230-7fc1e72251df.png)

Our reverse shell was a success!

![image](https://user-images.githubusercontent.com/115602464/197357545-f4bc7dfe-169c-486f-8c28-4063039c499d.png)

Running `whoami` shows us that we are www-data. This means our privileges are low, but we can still find and access the user.txt file.

We can either search around for the file manually or use the find command.

```find / -iname "user.txt" 2>/dev/null```
+ `find`: Find is a utility that searches for files in the linux directory hierarchy.
+ `/`: Specifies the directory we want to start in. This starts our search in the base / directory and recursively searches all files and folders within.
+ `-iname`: Allows us to specify the file name same as -name but is case insensitive.
+ `2>/dev/null`: Redirects any errors to /dev/null. Cleans up the output so that we don't see any useless information.

![image](https://user-images.githubusercontent.com/115602464/197357679-d5ebf36a-7d9c-4479-aef1-a9c2ff04f2d4.png)

`cat` the file to get your user flag. 🚩

# Task 4
Now that we have a shell, let's escalate our privileges to root.

## -Search for files with SUID permission, which file is weird?
<sub>Hint: find / -user root -perm /4000</sub>

We are asked for find files with the SUID permission set. The SUID(Set-user Identification) as well as the group variation(SGID) are permissions that the owner of an executable file can set so that alternate users can run it with privileges of the owner/group. [Learn More](https://linuxhint.com/special-permissions-suid-guid-sticky-bit/)

```find / -user root -perm /4000 2>/dev/null```
+ `-user`: Specifies that we are searching for files with root as owner.
+ `-perm /4000`: Specifies the permissions that we are looking for. We are looking for the SUID which is the 4 in 4000.
+ `2>/dev/null`: Redirects any errors to /dev/null. Cleans up the output so that we don't see any useless information.

![image](https://user-images.githubusercontent.com/115602464/197358826-e05c7b29-35f3-4919-a343-851562504429.png)

Python has the SUID/GUID permissions set. This pops out because having root access to python on the system opens up a lot of possibilities.

## -Find a form to escalate your privileges.
<sub>Hint: Search for gtfobins</sub>

[GTFObins](https://gtfobins.github.io) is a list of binaries that we can use to bypass security restrictions and misconfigured systems.
Typically if we have SUID/GUID set or sudo permissions on a binary we can use GTFObins to exploit it.

![image](https://user-images.githubusercontent.com/115602464/197359044-2e342585-ab82-4f8e-9875-47997e4f6541.png)

Looks like we have to execute python, indicated by ./python, with some options.
Lets `cd` into the /usr/bin directory, this is where the python binary is located.

Run the command under python/SUID section on GTFObins.

```./python -c 'import os; os.execl("/bin/sh","sh","-p")'```


![image](https://user-images.githubusercontent.com/115602464/197359142-8ab969c5-9411-4ff6-882b-eddd3c8289e9.png)

We're in!

## -root.txt

Lets CD into the root directory.

![image](https://user-images.githubusercontent.com/115602464/197359477-4567f062-d290-405a-97ae-23b36418de21.png)

`cat` the root.txt file to get your flag! 🚩
