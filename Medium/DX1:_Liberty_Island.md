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
  
  
</details>


<details>
  <summary>What is the Root flag?</summary>

Deploy your machine!

</details>
