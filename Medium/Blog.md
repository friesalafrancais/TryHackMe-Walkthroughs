# [Blog](https://tryhackme.com/room/blog)

> Difficulty: Medium
>
>Billy Joel made a blog on his home computer and has started working on it.  It's going to be so awesome!
>
>Enumerate this box and find the 2 flags that are hiding on it!  Billy has some weird things going on his laptop.  Can you maneuver around and get what you need?  Or will you fall down the rabbit hole...
>
>In order to get the blog to work with AWS, you'll need to add blog.thm to your /etc/hosts file.



<details>
  <Summary>Initial access</Summary>
  
  ## Hosts file
  
  The room description tells us we need to add the target machines IP to the /etc/hosts file.
  
  

  
  
    nano /etc/hosts
  

  
  ![image](https://user-images.githubusercontent.com/115602464/227738027-a9e1df7e-33d6-4a4e-b129-dfad0cf9bc25.png)

  ## Nmap
  
  Running an NMAP scan we can see SMB shares
  
    nmap -sV TARGETIP
  
  + `nmap`: Runs various scans against targets to gain information such as which ports are open, services running, etc. [More Info](https://linux.die.net/man/1/nmap)
  + `-sV`: Finds open ports, services running, service versions.
  
  ![image](https://user-images.githubusercontent.com/115602464/227737080-399b2f3a-c4bd-4f76-871b-c0b6d4729363.png)

  I've gone down this path, it leads to nothing.
  
  From our NMAP scan we can see a web page running, lets explore that.
  
  Looks like some sort of blog. If we take a look at the source code we can see that wordpress is running as the CMS.
  
  ![image](https://user-images.githubusercontent.com/115602464/227737323-54272615-f9ca-4769-b67c-b3e225e8c1c3.png)
  
  ## WPScan
  
  Lets enumerate wordpress using WPScan.
  
    wpscan --url blog.thm --enumerate u
  
  + `wpscan`: A security scanner for Wordpress.
  + `--url`: Specifies the IP we are scanning.
  + `--enumerate`: Allows us to enumerate specific parts of Wordpress. We use "u" for users.
  
  We can see a few interesting things here.
  
  ### -There is a wordpress admin portal at /wp-admin/.
  
  ![image](https://user-images.githubusercontent.com/115602464/227738601-e4e02be8-4a6e-41fc-bb5d-280ae27d5c67.png)

  ### -Wordpress is running version 5.0 which is outdated and insecure.
  
  ![image](https://user-images.githubusercontent.com/115602464/227738721-9d7e5e8e-508b-4b21-af7f-51274d29c2b9.png)
  
  ### -We enumerated two users, kwheel and bjoel.
  
  ![image](https://user-images.githubusercontent.com/115602464/227738694-a6464c6f-64ee-4503-b426-591755444107.png)

  
  I did some more digging down rabbit holes but couldn't find anything interesting. Lets try and brute force the logins for the two users we have.
  
  Using WPScan we can brute force the login by providing the users and password wordlist.
  
    wpscan -U users.txt -P /usr/share/wordlists/rockyou.txt --url http://blog.thm
  
  + `wpscan`: A security scanner for Wordpress
  + `-U`: Specifies the user/users
  + `-P`: The password/wordlist
  + `--url`: Specifies the IP we are scanning
  
  ![image](https://user-images.githubusercontent.com/115602464/227742082-6c4cd34a-a2b9-46e7-81d1-4176cf9f3176.png)
  
  We found a password for the user kwheel! `cutiepie1`
  
  ## Metasploit
  
  After logging into the kwheel account and looking around for a bit, I couldn't find anything useful.
  
  Searching for `wordpress 5.0` in metasploit we can see 3 modules.
  
  ![image](https://user-images.githubusercontent.com/115602464/227742649-d6a267e8-0210-408d-9c60-8bd630edbe51.png)
  
  Lets use the first module `use 0`.
  
  Running `options` we can see `username, password, rhosts` are required.
  
  Make sure your `lhost, port` options are also set (should be by default).
  
  Running this module gets us low privilege access!
  
  If we run the command `shell` in meterpreter and then `whoami` we can see we are currently `www-data`.
  
</details>


<details>
  <Summary>Privilege Escalation</Summary>
  
  ## SUID
  
  Looking around we can see we have an SUID permission set for a strange program called `checker`.
  
  
  ![image](https://user-images.githubusercontent.com/115602464/227744419-68d16194-e123-4b26-be93-576878c01418.png)

  Running the checker bin just gives us the output "Not an Admin".
  
  ## LTrace
  
  If we run ltrace on checker it seems its checking an admin environment variable. 
  
  ![image](https://user-images.githubusercontent.com/115602464/227745370-6e5dfcf8-beac-4c57-b2b8-b210f30b86e5.png)

  Lets create an admin environment variable and set it to True. (Though in my experience it doesnt seem to matter what you set it to).
  
    export admin=True
  
  Run checker again. You wont get any visible feedback but if you run whoami you should be root!
  
  ![image](https://user-images.githubusercontent.com/115602464/227745361-aa3d3bc1-fffe-4229-9778-28e033f329fe.png)

  ## Root/user.txt 🚩
  
    find / -name "user.txt" 2>/dev/null
    find / -name "root.txt" 2>/dev/null
  
  + `/`: Searches recursively from the base directory
  + `-name`: The name of the file we're looking for
  + `2>/dev/null`: Outputs errors to null, this removes "Permission denied" from the output
  
</details>
