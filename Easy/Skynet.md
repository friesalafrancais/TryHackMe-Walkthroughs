# [Skynet](https://tryhackme.com/room/skynet)

> Difficulty: Easy

# Scanning
We run an NMAP with  
```
nmap -sV targetmachineIP
```

+ `-sV`: Not only checks for open ports but the version of the service running.

![image](https://user-images.githubusercontent.com/115602464/195729150-4dab67da-e7ae-4a20-9404-8c3a921d1ed6.png)

Two things pop out to me here. There is a website being hosted on port 80 and samba is running on 139 and 445.

+ `Samba`: is a program that allows users to access shared files, printers, and other resources on a company's intranet.

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

Lets get into it using smbclient.

