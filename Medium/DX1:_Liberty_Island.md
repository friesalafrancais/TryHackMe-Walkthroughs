# [RootMe](https://tryhackme.com/room/rrootme)

> Difficulty: Easy

> Open the necessary task below!

<details>
  <summary>What is the User flag?</summary>

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
  
Head over to [CyberChef](https://gchq.github.io/CyberChef/). Search for the HMAC operation on the left side and drag it into the recipe section.
  
Select MD5 as your hashing function, UTF8 as the key type, use jlebedev as the key, and smashthestate as the input.
  
![image](https://user-images.githubusercontent.com/115602464/198855267-885f1aaa-1560-4b42-97cc-19a5c107348c.png)
  
Your hash will be in the output, lets copy the first 8 characters of the hash.
  
Lets connect to VNC on our target machine with vncviewer.
  
`vncviewer targetmachineIP:5901`
  
We are prompted with a VNC authentication window. Enter our 8 character password from cyberchef!
  
![image](https://user-images.githubusercontent.com/115602464/198855358-4c952219-db4e-457c-83e7-2b6e81897588.png)
  
We have access to the machine with ajacobsons account!
  
![image](https://user-images.githubusercontent.com/115602464/198855415-eaa08d39-f44b-44e1-a81d-39eb403ca41e.png)
  
Open up user.txt to see your user flag! 🚩

</details>


<details>
  <summary>What is the Root flag?</summary>

Deploy your machine!

</details>
