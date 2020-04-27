+++
title = "Week 4 Robot Box WriteUp"
date = "2020-02-16"
author = "Donny"
cover = ""
description = "Write up of the Mr Robot box on tryHackMe"
+++

# Mr Robot

# Engagement

As soon as the site was live, I spent an obscene amount of time on the actual content almost forgetting my task was to hack the box. Recalling the pent tester methodology below, the first step requires Information gathering. 

{{<image src="/img/RBArti0.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

# Information Gathering

The most logical simple step was to scan for ports using nmap, luckily it comes pre-installed on linux.

{{<image src="/img/RBArti1.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

Once the scan was finished, I saw that 2 ports were open, one was http on port 80 and the other was SSL on port 443. I also saw that the website was running apache. There was no useful information to me so I tried something else. I knew that Directory traversal could be used to find arbitrary files on a server. To my luck I found the etc/shadow directory was running a wordpress page. 

{{<image src="/img/RBArti2.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

# Vulnerability Assessment

And it just so happens that there is a tool that scans wordpress sites that I can use. Even better, it comes pre-installed on Kali. I also looked at the page source but found nothing. I whip out my terminal, and type in the following command:

    wpscan --url <ip address>

I find some interesting things, mainly that the version running is 4.3.1 released in 2015. I also find that this version is insecure. I find the robots.txt file which was the fist key - WOOOOO! 

{{<image src="/img/RBArti3.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

There is also a dictionary file that after consulting with Max I realised I could use to get the password for the admin. Here is the robots page with the first key and the dictionary file.

{{<image src="/img/RBArti4.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

I had watched the show and knew that Elliot was the name of the main character so I changed my command from:

    wpscan --url <ip Address> -U robotlist.txt -P robotlist.txt  

to 

    wpscan --url <ip Address> --passwords RobotWordList --usernames elliot
    //Can also we written as:
    wpscan --url <ip Address> -P RobotWordList -U elliot

Some things to note:

- Robotlist.txt  and RobotWordList are have the same content, it is the fsocity.dic file I got from the robots.txt page. The reason for this change is because I thought the reason for the abnormally long waiting time was that the duplicates had not been removed properly. Therefore, I used the command below to redo the process of removing the duplicate entries and thus had to make another file.

    cat robotlist.txt | sort | uniq > RobotWordList

- The second thing to note is that the -U = —username and the -P = —passwords are the same parameter.  Just different ways of writing the same thing.

After I tried the second command I had the results in about 10 minutes, as opposed to hours of waiting. A major problem with this box is that for some reasons, windows loses internet connection at certain intervals so I have to constantly reconnect to the VPN and deploy the machine, I cannot wait to fully convert to Linux. Back to the box, this is the result I got: 

{{<image src="/img/RBArti5.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

After putting it into the login form to confirm: 

{{<image src="/img/RBArti6.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

After many long hours of struggle and hardship, I was able to get admin access onto the wordpress site. The next question is - "What do I do next".

Answer - ask Google.

# Exploitation

After just a quick youtube video I knew exactly what to do. This is the process:

For the reverse shell code, you could download the zip file from a source like Pentest Monkey or copy and paste the code from the GitHub into the 404.php template. I didn't bother with downloading a malicious plugin and installing it either. The resource I was using at the time showed me that it was already included in kali.

{{<image src="/img/RBArti7.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

I simply changed my IP address which I gained from using "ifconfig" and also the port number "1234" since it was easy to remember. I could've picked any theme but I chose the 404.php page because it is more stealthy. In other words, the code will cause the site to appear to hang and thus nothing will appear to be suspicious. However, my backdoor runs and I have access. 

{{<image src="/img/RBArti8.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

Using netcat, I started listening on port 1234. I refreshed the page and then had access. I also checked who I was using "whoami" and then tested If had sudo privileges using "sudo -v". I didn't but I wasn't expecting it to be that easy. 

{{<image src="/img/RBArti9.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

Then I went into the "home" directory, then from there I saw there was another user named Robot. I  also saw that with "robot" was another key and an m5d hashed password. I did not have access to the key as daemon, but I used an online MD5 cracker to get the password. Then I used a python script to import a terminal shell so that I could set user as Robot. After I was logged in as robot I was able to get the second flag.

{{<image src="/img/RBArti10.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

The last steps were a bit of a pain but I managed it. I knew the last step was privilege escalation. Now privilege escalation is a huge rabbit hole. So I just watch a YouTube video on common privilege escalation.

{{<image src="/img/RBArti11.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

I was talking to max trying to understand SUID better because there were gaps in my knowledge. The I realised that Nmap was executable by robot but is owned by root. Using gtfobins - [https://gtfobins.github.io/](https://gtfobins.github.io/) the Nmap had an entry to execute shell commands:

{{<image src="/img/RBArti12.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

I had to start nmap in interactive mode and then the "!sh" command and then see that I am root user.

{{<image src="/img/RBArti13.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

# Reporting

This box was a challenge to be sure, and I made many errors. Some of them very stupid, some of them showcased the gaps in my knowledge. Nevertheless I really enjoyed rooting this box, I learnt a lot and I am motivated to try another box soon. 

{{<image src="/img/RBArti14.PNG" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}