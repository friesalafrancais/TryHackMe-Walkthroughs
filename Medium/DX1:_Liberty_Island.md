# [DX1: Libery Island](https://tryhackme.com/room/dx1libertyislandplde)

> Difficulty: Easy

> Open the necessary task below!

<details>
  <summary>What is the User flag?</summary>
  
# NMAP

Run an nmap scan to see what ports are available.
  
`nmap -sV targetmachineIP`

+ `nmap`: Runs various scans against targets to gain information such as which ports are open, services running, etc. [More Info](https://linux.die.net/man/1/nmap)
+ `-sV`: Finds open ports, services running, service versions.

![image](https://user-images.githubusercontent.com/115602464/198853490-e8daa017-676c-4ef2-ab7a-e209497e218b.png)

Scan shows us ports 80 and 5901 are open.

-80 is for a web server.

-5901 is for a service called [VNC](https://www.realvnc.com/en/). VNC is a service similar to Remote Desktop Protocol, lets keep it in mind.

  
Head over to the site to browse it!
  
![image](https://user-images.githubusercontent.com/115602464/198853696-2aed84ae-ea31-4f2e-955e-e745992363e0.png)
  
The front page and first two links dont really have anything. The third page, however, gives us a cyber watchlist with usernames of apparent cyberterrorists.
  
![image](https://user-images.githubusercontent.com/115602464/198853764-3f4f7396-42a8-4ac8-b8fa-7f8adc1b17fd.png)
  
Lets keep this list in mind as it might be useful later.

  
# Dirb
  

With nothing else standing out, lets run a dirb scan. Dirb is a program that will enumerate directories for us and find directories that were perhaps meant to be hidden.
  
`dirb http://targetmachineIP/`
  
Dirb found /index.html, /robots.txt and /server-status. Lets visit each of these.
  
+ `Index.html` leads us to the main page.
  
+ `Server-status` is a page we dont have access to.
  
+ `Robots.txt` gives us some interesting information. [Robots.txt](https://developers.google.com/search/docs/crawling-indexing/robots/intro) files are part of websites to manage where web crawlers are and aren't allowed to go.

The only page being disallowed by robots.txt is /datacubes. Head there!

![image](https://user-images.githubusercontent.com/115602464/198854120-9fbeafe7-ac00-49ec-a93d-a8bd5f7327ff.png)

  
This is a datapads archive.
  
The page redirected us to /datacubes/0000/. We can safely assume that /0000/ is the number of the current datacube.
  
Instead of going through the datacubes one by one, lets have dirb do the work for us!
  
This will make a clean list going from 0000 to 9999 for us.
  
`seq -w 0000 9999 > wordlist`
  
+ `seq`: creates a sequence of numbers. [More Info](https://linux.die.net/man/1/seq)
+ `-w`: Equalizes the width by padding with leading zeroes. (ex. 0025, instead of 25)
+ `>`: Saves the output into a file for us, in this case the filename is wordlist.
  
Now we can utilize dirb again to scan for any directories that exist using those numbers.

We come back with 5 hits: /0011/, /0068/, /0103/, /0233/, and /0451/.

0011, 0068, 0103, and 0233 don't give us any important information on the pages or the page sources.

/0451/ is more useful though.
  
![image](https://user-images.githubusercontent.com/115602464/198855100-51ad5257-b91a-47a8-b28b-a258e1faacff.png)
  

On this page we are given two important pieces of information.
  
The `login` for VNC is "smashthestate", hmac'ed with a username from the cyberterrorist list we saw earlier at /badactors.html.

The `password` for VNC is the first 8 characters of the login hash. The algorithm used is md5.
  
![image](https://user-images.githubusercontent.com/115602464/198855226-6f7378bf-506b-4659-8a42-00c7ed809a28.png)
  
Specially the hmac key is "my username". The message is written by JL. Looking at the cyberterrorist list we can assume `jlebedev` is the correct username.
  
# VNC

Head over to [CyberChef](https://gchq.github.io/CyberChef/). Search for the HMAC operation on the left side and drag it into the recipe section.
  
Select MD5 as your hashing function, UTF8 as the key type, use jlebedev as the key, and smashthestate as the input.
  
![image](https://user-images.githubusercontent.com/115602464/198855267-885f1aaa-1560-4b42-97cc-19a5c107348c.png)
  
Your hash will be in the output, lets copy the first 8 characters of the hash.
  
Lets connect to VNC on our target machine with vncviewer.
  
`vncviewer targetmachineIP:5901`
  
We are prompted with a VNC authentication window. Enter our 8 character password from cyberchef!
  
![image](https://user-images.githubusercontent.com/115602464/198855358-4c952219-db4e-457c-83e7-2b6e81897588.png)
  
We have access to the machine with ajacobsons account!
  
![image](https://user-images.githubusercontent.com/115602464/198855576-7d02c8cd-47a1-43cb-8b8c-d79f4d4366bb.png)
  
Open up user.txt to see your user flag! 🚩

</details>


<details>
  <summary>What is the Root flag?</summary>
  
# Badactors-list
  

The desktop of our victim machine has a program called badactors-list. Lets run this.
  
The program opens up and we see that its connecting to host UNATCO on port 23023. It gives us the same cyberterrorist list as the website does.
  
![image](https://user-images.githubusercontent.com/115602464/198855730-4e4b6015-3f6f-45be-b858-18f8c8604100.png)

Lets take a look at the [/etc/hosts](https://tldp.org/LDP/solrhe/Securing-Optimizing-Linux-RH-Edition-v1.3/chap9sec95.html) file to see what UNATCO is representing.

The hosts file shows us that UNATCO has the ip 127.0.0.1. This is the localhost, or loopback, address. This is a special address that is used only by the host machine itself. It essentially points to itself. What this means is the victim machine has port 23023 open for this special program. When it says syncing with http://UNATCO:23023 it is in fact connecting to itself on port 23023.
  
Lets see if we can somehow utilize this to escalate our privileges.
  
Lets use python to host a simple HTTP server to get the file over to our attacker machine.

![image](https://user-images.githubusercontent.com/115602464/198855688-e87892a6-d9aa-40ac-b5b5-c7f1223f74d7.png)

We can see that this machine has python3.8 installed. We can use this to our advantage to move the badactors-list program over to our attacker machine!
  
Python3.8 has a module called http.server that we can use. [More Info](https://docs.python.org/3.8/library/http.server.html)
  
`Python3.8 -m http.server` will run the python server from the `current directory` on the default port `8000`.
 
On our attacker machine, lets head to `http://targetmachineIP:8000/`
  
![image](https://user-images.githubusercontent.com/115602464/198855983-133beefd-fcc3-4c26-ab38-3fdf50eaf392.png)

Lets click on badactors-list to download it. It should be downloaded to the /root directory if you are on the attackBOX machine on THM.
  
CD into that directory and run chmod to add execute permission to the file. `chmod +x badactors-list`
  
Run the program with `./badactors-list` and notice nothing happens!
  
This is due to UNATCO not being listed in our `/etc/hosts/` file. Remember that our victim machine had 127.0.0.1 set to UNATCO. This means that if we want the program on OUR attacker machine to connect properly, we will have to set the victims machine as UNATCO in our /etc/hosts file.
  
![image](https://user-images.githubusercontent.com/115602464/198856208-5488ce21-14c2-4527-a831-9c6676b0ed8d.png)

Now our program should work!

# Wireshark
  
Wireshark allows us to analyze traffic entering/exiting our machine. We we use it to understand what the program is doing.
  
Once wireshark is opened, select eth0 as the device we want to monitor. You might see a bunch of traffic but don't worry. We will filter it.
  
With wireshark opened up, monitoring eth0, open up the program and watch the packets come in! Depending on how much you have going on in the background there could be hundreds or thousands of packets. To solve this we will use display filters. `tcp.port == 23023 && http` will show us specifically packets that are using tcp port 23023 AND the HTTP protocol.
  
You should have two packets. Select the one that has your machines IP as the source, and the victim machine as the destination. 
  
![image](https://user-images.githubusercontent.com/115602464/198856456-6e8444c4-5e58-4834-90b4-a76f8c7baf5a.png)

Open up the packet. Inside the info box below, open up `Hypertext Transfer Protocol` and `HTML Form URL Encoded`.
  
![image](https://user-images.githubusercontent.com/115602464/198856509-90f9d550-de61-4240-bcaa-4d83d2e9321a.png)
  
Everything looks normal here besides the Clearance-Code and the Directive form item. It looks like the Clearence-Code is being sent by the program to get authorized access. It then uses the directive key to run a command, in this case it is `cat /var/www/html/badactors.txt`.

# Curl
  
We will craft our own request using [curl](https://linux.die.net/man/1/curl).
  
`curl -H 'Clearance-Code: yourswillgohere' -d 'directive=whoami' targetmachineIP:23023`

+ `curl`: a tool used to transfer data to/from a server.
+ `-H`: used to provide headers. In this case, we are adding the Clearance-Code to the header.
+ `-d`: Specifies the data we want sent. Usable in POST requests. In this case, we are sending the directive "whoami"
  
![image](https://user-images.githubusercontent.com/115602464/198857463-82009970-0d9c-4263-98dc-511254c9d68e.png)

We can see that our curl request works successfully! The whoami command returns `root. Since we are able to execute commands as root, lets peak into the root directory. Change your curl request to include the directive `ls /root`.
  
![image](https://user-images.githubusercontent.com/115602464/198857496-b4b14527-1831-43ca-b9d3-12e03b63475b.png)

We found root.txt! Run the curl command again, except this time change out `ls /root with `cat /root/root.txt`.
  
Congratulations! 🚩

</details>
