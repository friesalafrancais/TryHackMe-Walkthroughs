# [RootMe](https://tryhackme.com/room/rrootme)

> Difficulty: Easy

> Open the necessary task below!

<details>
  <summary>What is the User flag?</summary>

Run an nmap scan to see what ports are available to us.
`nmap -sV targetmachineIP`

+ `nmap`: A program that runs various scans against targets to gain information such as which ports are open, services running, OS, etc. [More Info](https://linux.die.net/man/1/nmap)
+ `-sV`: Finds open ports, services running, service versions.

![image](https://user-images.githubusercontent.com/115602464/198853445-09d21943-08ff-4694-8c59-83ebaed126fc.png)

Scan shows us ports 80 and 5901 are open.

80 is for a web server.

5901 is for a service called [VNC](https://www.realvnc.com/en/).



</details>


<details>
  <summary>What is the Root flag?</summary>

Deploy your machine!

</details>
