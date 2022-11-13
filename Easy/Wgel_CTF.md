# [Wgel_CTF](https://tryhackme.com/room/wgelctf)





> Difficulty: Easy
>
>"Have fun with this easy box."


![image](https://user-images.githubusercontent.com/115602464/201534339-432ee364-3c01-4d4e-9f99-7611bf02536e.png)



<details>
  <summary>user_flag.txt</summary>
  
# NMAP

Run an NMAP scan against the targetmachineIP. [Nmap Info](https://linux.die.net/man/1/nmap)
```
nmap -sV targetmachineIP
```
+ `-sV`: Service/Version detection scan
  
![image](https://user-images.githubusercontent.com/115602464/201540101-2392fe1f-cfa1-43f8-b016-6b88dc589298.png)

We see that the following services are running:
  

`ssh` - port 22
  
`apache server` - port 80
  
Keep ssh in mind, lets check out the web server.
  
![image](https://user-images.githubusercontent.com/115602464/201540351-034bf3b6-e504-4cb8-a98d-ed818b3f08bb.png)

Looks like the standard apache page that is loaded after being installed.
  
Lets take a look at the page source. 
  
![image](https://user-images.githubusercontent.com/115602464/201540693-7d4dc133-58d5-4fcd-9150-fbbaaa42d6b8.png)
  
We have a comment that was left behind for a user named Jessie!
  
  
# DIRB
 
Lets run a dirb scan. This will reveal any directories that might be useful. [Dirb Info](https://www.kali.org/tools/dirb/)

Dirb found quite a few directories, but only two seem interesting to me. /sitemap/ and /sitemap/.ssh/. Lets check them out!
  
/sitemap/ seems to be a rabbit hole.
  
![image](https://user-images.githubusercontent.com/115602464/201540748-93d7b9b7-8c2b-44cd-a78c-a1699232874c.png)

/sitemap/.ssh/ contains a file named id_rsa, if we open it up we can see its an RSA private key! 
  
We can use this id_rsa file with ssh to show we have the private key for a specific user. Copy the private key(everything on the page, including the dashes) and save it to your attackbox. Make sure you name it id_rsa.
  
# SSH

Lets try and use this id_rsa file for the user Jessie. No one users are known to us at this time. [SSH Info](https://linux.die.net/man/1/ssh)
  
`ssh targetmachineIP -i id_rsa`

+ `-i`: "Selects a file from which the identity (private key) for RSA or DSA authentication is read."

The id_rsa file is in the present working directory of my terminal so I didnt need to specify the path.
  
![image](https://user-images.githubusercontent.com/115602464/201541931-20396879-726b-46bf-a5e7-2fd10dda8a74.png)

We are given an error telling us our private keys cant be accessible by others. Lets chmod the file to remove any permissions from group/other.
  
`chmod 0600 id_rsa`

+ `0600`: This gives owner r/w, and group/others NO permissions.
  
Do a quick `ls -la` to make certain the permissions are properly set. You should see "-rw-------" permissions for id_rsa.
  
Re-run our ssh command again.
  
![image](https://user-images.githubusercontent.com/115602464/201542134-bce5e849-eac8-47d5-86f1-f1eac1023597.png)
  
We're in as Jessie!
  
If we run `ls *`, we can see the user_flag.txt inside the Documents folder!🚩
  
</details>



<details>
  <summary>root_flag.txt</summary>
  
# GTFObins
  

After looking around a little, there don't seem to be any useful files.
  
Lets check our sudo permissions! `sudo -l`
  
This shows us what we're able to run as root using sudo.
 
![image](https://user-images.githubusercontent.com/115602464/201542491-a39cd09a-0674-44ab-b87f-c9aaa7db5876.png)

Looks like we've been given sudo permissions for the wget bin. [Wget Info](https://linux.die.net/man/1/wget)
  
Head over to GTFObins, and search for wget.

We can see there are a few different methods for wget. The one we will be using is "File read".
  
![image](https://user-images.githubusercontent.com/115602464/201542665-58ac212f-cd41-40a3-8c1c-ba4334d626d2.png)


# WGET
  
Most root flags are in the root home directory. If we follow the naming convention of the user flag we found earlier, we are trying to read root_flag.txt.
  
Remember, we want to run wget with sudo to get the elevated permissions, otherwise we won't be able to access anything in the root directory.
  
We can skip creating a variable and just use the full file path.
  
`sudo wget -i /root/root_flag.txt`
  
+ `-i`: Allows us to specify files we want to use as URL inputs.
  
![image](https://user-images.githubusercontent.com/115602464/201542861-51046a37-555a-449a-8368-ff42011e6395.png)

  
We have the root flag!🚩
  
The output looks funny, because wget is using the contents of root_flag.txt as a web address.
  
These long strings are, of course, not valid URLs.
  

</details>
