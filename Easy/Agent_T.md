# [Agent T](https://tryhackme.com/room/agentt)

> Difficulty: Easy
<details>
  <summary>Scanning</summary>
  
# Scanning
We have one flag with a hint that asks us to take a look at the HTTP headers.
We can view the headers by using Curl. Curl is a neat tool that lets us easily communicate with a server from our terminal.
We can upload/download things. In this case we can use the -I option to view the headers of our request.

```
Curl -I http://targetmachineIP
```
+ `-I`: Outputs the headers of our request

![HEADER](https://user-images.githubusercontent.com/105746327/194944244-c78971e7-9238-4361-8842-c0e63fa0b775.jpg)

Most of the information in the headers seems normal except for X-Powered-By.
This looks like something we can dig into further and possibly exploit!

</details>

<details>
  <summary>Exploitation</summary>
  
  
# Exploitation

Heading over to exploit-db.com we can search for ```PHP 8.1.0-dev```.

![EXPLOITDB](https://user-images.githubusercontent.com/105746327/194945329-eef1d32c-7111-4974-a9e5-714ff4af80fb.jpg)

We can see here that a remote code execution exploit exists for this version of PHP.
Lets download it and take a look at the python code! Look for the download arrow at the center of the page.

![EXPLOITCODE](https://user-images.githubusercontent.com/105746327/194945891-361a1fd5-5925-45f1-826a-d48cfafd76cd.jpg)

After looking at the code, we can see here that if we run the script we will be prompted for a URL.
In our case the URL we are exploiting is ```http://targetmachineIP/```.

![EXPLOITED](https://user-images.githubusercontent.com/105746327/194946101-d95c9236-af5b-4594-8b8c-a72e969778ea.jpg)

And we're in!
We can do a little bit of searching around. A basic ```ls /``` shows us whats in the base directory.
We can see a flag.txt. Run ```cat /flag.txt``` to view the flag!
+ `ls /`: Tells us what files exist at the base directory /.
+ `cat /flag.txt`: Outputs the contents of the flag.txt file.

![FLAG](https://user-images.githubusercontent.com/105746327/194949081-ede78965-89cc-443a-a8c9-862f21e10d35.jpg)

</details>
