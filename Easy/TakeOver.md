# [TakeOver](https://tryhackme.com/room/takeover)

> Difficulty: Easy
>
>"Hello there,
>
>I am the CEO and one of the co-founders of futurevera.thm. In Futurevera, we believe that the future is in space. We do a lot of space research and write blogs about it. We used to help students with space questions, but we are rebuilding our support.
>
>Recently blackhat hackers approached us saying they could takeover and are asking us for a big ransom. Please help us to find what they can takeover.
>
>Our website is located at https://futurevera.thm
>
>Hint: Don't forget to add the MACHINE_IP in /etc/hosts for futurevera.thm ; )"

  
# Scanning
Let's begin by running an nmap scan.

```
nmap -sV MACHINE_IP
```
+ `sV`: Gives us version detection for running services
  
![image](https://user-images.githubusercontent.com/115602464/214450492-de961570-44e1-4962-a410-914b4e4a3121.png)

Looks like 22, 80, 443 are open. SSH on 22, http on 80 and 443.

The description of the room asks that we add the MACHINE_IP to our hosts file. The hosts file is located in `/etc/hosts`
  
```
nano /etc/hosts  
```
  
![image](https://user-images.githubusercontent.com/115602464/214451237-78792940-c7a6-45f3-97dc-d263ad79592d.png)

Visiting the site, we are greeted with the landing page for FutureVera!

Looking around the page, there doesn't seem to be any obvious clues. 

Checking page source is also a dead end.
  
  
## Enumerating Directories

The hint given to us is that enumeration is key. Lets do a dirb scan first. [More info](https://manpages.debian.org/unstable/dirb/dirb.1.en.html)
```
dirb http://futurevera.thm/ /usr/share/wordlists/dirb/big.txt 
```
This brings back /status which we don't seem to have access to.
  
## Enumerating Subdomains
  
Lets run a subdomain scan against http://futurevera.thm using ffuf. [More info](https://linuxcommandlibrary.com/man/ffuf)
  
```
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS?subdomains-top1million-110000.txt -H "Host: FUZZ.futurevera.thm" -u http://10.10.47.183 -fs 0
```
+ `w`: Wordlist used
+ `H`: Host headers
+ `u`: URL being fuzzed
+ `fs 0`: Filters out message size response of 0 bytes 

![image](https://user-images.githubusercontent.com/115602464/214466991-dfaac1a6-4e03-4f75-9613-31d7c7adcbcb.png)

As we can see, the fuzzing returns `portal` and `payroll`.

Lets run the same command with one slight difference, change the url we're fuzzing to https://futurevera.thm. 

```
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS?subdomains-top1million-110000.txt -H "Host: FUZZ.futurevera.thm" -u https://10.10.47.183
```
We can see the size of most of the fuzz results is 4605. These are useless to us, we're interested in results that stand out.

Just like the first ffuf command, lets add `-fs 4605` at the end to filter out results of that size.
  
We have two new results! Blog and Support.

## Update hosts file

If we try to visit these, we won't have success because they are not in our hosts file.
  
Head over to your `/etc/hosts` file and add two new lines similar to the first time.

Your hosts file should have these:

```
MACHINE_IP futurevera.thm
MACHINE_IP portal.futurevera.thm
MACHINE_IP payroll.futurevera.thm
MACHINE_IP blog.futurevera.thm
MACHINE_IP support.futurevera.thm
```
MACHINE_IP is, of course, your target machines IP address, it will be different than mine so i'm using a placeholder.
  
Lets visit every one of these subdomains.
  
Portal and payroll require an internal VPN to access them.
  
Looking around in blog.futurevera.thm, it also seems quite empty.
  
However, when we connect to https://support.futurevera.thm, firefox warns us that the certificate isnt valid for support.futurevera.thm, but IS valid for another site.
 
![image](https://user-images.githubusercontent.com/115602464/214473054-fd796ddb-843b-48a5-aefa-4afb12604eae.png)

Lets add this to our hosts file and give it a visit.
  
When we visit with using https it seems to take us to the main landing page.
  
Lets visit the new page using http://.
  
We get an error, but look closely, we have our flag! 🚩
  
![image](https://user-images.githubusercontent.com/115602464/214474153-0c0dd4c0-315e-4038-bfc5-c765cfdaa9c1.png)
